# Bloc-note d'ingenieur - Chapitre 4
## Outputs, Inputs, and Timers

### Idee centrale

Ce chapitre explique comment un logiciel embarque interagit avec le monde physique : il commande des sorties, observe des entrees et s'appuie sur des timers pour maitriser le temps. Ces mecanismes paraissent simples, mais ils imposent de comprendre les registres, les contraintes electriques, les interruptions et les incertitudes du monde reel.

La regle de conception essentielle est de separer la logique applicative du code dependant du microcontroleur et de la carte.

## Sorties : commander le materiel

### GPIO de sortie

Une sortie GPIO permet de placer une broche a un niveau logique bas ou haut. Elle peut servir a commander une LED, un transistor, un signal de selection de circuit, ou a fournir une indication de diagnostic.

Pour utiliser une sortie, il faut en general :

1. Activer l'horloge du port GPIO.
2. Configurer le multiplexage de la broche pour choisir la fonction GPIO.
3. Configurer la direction en sortie.
4. Choisir l'etat initial sans provoquer de front involontaire.
5. Ecrire l'etat haut ou bas dans le registre de sortie.

### La LED clignotante

Faire clignoter une LED est le "Hello World" de l'embarque. Ce test simple valide souvent :

- l'alimentation de la carte ;
- le demarrage et l'execution du firmware ;
- la configuration de l'horloge ;
- l'acces aux registres du GPIO ;
- le routage de la broche et le montage de la LED.

Une LED peut etre active-haut ou active-bas. Il faut donc verifier le schema : ecrire un `1` ne signifie pas toujours que la LED s'allume.

### Eviter les acces fragiles aux registres

Un registre GPIO peut contenir plusieurs broches. Une ecriture de type lecture-modification-ecriture risque de modifier un bit entre la lecture et l'ecriture, notamment si une interruption intervient.

Quand le microcontroleur le propose, utiliser les registres atomiques de mise a 1, mise a 0 ou basculement (`SET`, `CLEAR`, `TOGGLE`) plutot qu'une modification manuelle du registre complet.

### Couche d'abstraction materielle

Le code applicatif ne devrait pas connaitre les adresses de registres ni les numeros de broches.

Exemple d'interface :

```c
void status_led_set(bool on);
void status_led_toggle(void);
```

L'application appelle `status_led_set(true)` ; le module specifique a la carte sait quelle broche commander et si la LED est active-haut ou active-bas. Cette facade limite les changements necessaires lors d'un changement de carte ou de microcontroleur.

## Entrees : observer des evenements reels

### Lire un bouton

Une entree GPIO retourne un niveau logique. Pour un bouton, il faut definir un etat stable au repos avec une resistance de pull-up ou de pull-down, interne ou externe. Sans cette polarisation, l'entree flotte et peut produire des valeurs aleatoires.

Une lecture periodique convient si l'evenement n'est pas critique et si la periode d'echantillonnage est adaptee. Le logiciel doit traduire le niveau electrique en intention : pression, relachement, appui long ou double-appui.

### Interruptions GPIO

Une interruption permet au processeur de reagir a un front montant, descendant, ou aux deux, sans surveiller continuellement la broche.

Une routine d'interruption (ISR) doit etre courte et previsible :

- acquitter la source d'interruption si necessaire ;
- enregistrer un evenement, un horodatage ou positionner un drapeau ;
- reporter le travail long a la boucle principale, une tache ou une machine a etats.

Il faut eviter dans une ISR les delais, les boucles longues, les allocations, les acces bloquants et les traitements complexes. Une ISR trop longue retarde les autres interruptions et degrade la reactivite globale.

### Anti-rebond (debounce)

Un bouton mecanique ne change pas proprement d'etat : ses contacts rebondissent durant quelques millisecondes et peuvent produire plusieurs fronts pour un seul appui.

Strategies courantes :

- Apres un premier front, ignorer les changements pendant une courte fenetre temporelle.
- Echantillonner periodiquement et valider l'etat seulement apres plusieurs lectures identiques.
- Utiliser un filtrage materiel si le contexte l'exige.

Le delai d'anti-rebond doit etre choisi a partir du composant et des contraintes d'usage ; une valeur trop courte laisse passer des faux evenements, une valeur trop longue rend l'interface moins reactive.

## Timers : maitriser le temps

### Pourquoi utiliser un timer materiel

Les timers comptent des impulsions d'horloge et produisent des evenements a des instants previsibles. Ils servent notamment a :

- creer un tick periodique ;
- mesurer une duree ou une frequence ;
- declencher une interruption a echeance ;
- generer du PWM ;
- horodater un evenement externe.

Un delai en boucle active depend du compilateur, de la frequence CPU et des interruptions. Il bloque aussi le processeur. Il est donc inadapté aux delais longs ou a un systeme qui doit faire autre chose pendant l'attente.

### Calcul de periode

Avec une horloge timer de frequence `f_timer`, un prescaler `P` et une valeur de rechargement `N`, une approximation courante est :

```text
periode = N * P / f_timer
frequence = f_timer / (N * P)
```

La formule exacte depend du microcontroleur : certains timers comptent de `0` a `N`, ce qui introduit un `N + 1`. Il faut toujours verifier le mode de comptage dans la datasheet et tester le signal a l'oscilloscope si le timing est critique.

Exemple : avec une horloge de 48 MHz, un prescaler de 4800 et 1000 pas de comptage, on obtient une periode de 100 ms.

### Attentes longues et non bloquantes

Un timer peut fournir une base de temps courte, puis le logiciel compare l'heure courante avec une echeance. La boucle principale reste ainsi libre de traiter les communications, entrees et autres taches.

Principe :

```c
if ((uint32_t)(now - deadline) < 0x80000000u) {
    // echeance atteinte
}
```

Cette forme de soustraction non signee permet de gerer correctement le retour a zero d'un compteur, a condition que les delais restent inferieurs a la moitie de sa plage de comptage.

## PWM : moduler une sortie

Le PWM (Pulse Width Modulation) produit un signal periodique dont le rapport cyclique varie. Si la periode est `T` et le temps a l'etat haut `t_on` :

```text
rapport cyclique = t_on / T * 100 %
```

Usages frequents :

- variation de luminosite d'une LED ;
- commande de vitesse d'un moteur ;
- commande de servo-moteur ;
- generation d'une consigne moyenne pour certains actionneurs.

Le choix de frequence est important : trop basse, elle peut etre visible ou audible ; trop elevee, elle peut reduire la resolution disponible ou augmenter les pertes de commutation. La charge commandee et son driver doivent etre pris en compte : un GPIO ne pilote pas directement un moteur.

## Testabilite et architecture

### Injection de dependances

La logique metier peut dependre d'interfaces simples plutot que des registres reels :

```c
typedef struct {
    bool (*button_is_pressed)(void);
    void (*led_set)(bool on);
    uint32_t (*time_ms)(void);
} board_io_t;
```

En production, ces fonctions appellent les drivers materiels. En test, elles peuvent etre remplacees par des doubles simulant un bouton, une LED et le temps. On peut ainsi tester une machine a etats, un anti-rebond ou une temporisation sans carte physique.

### Ce qu'il faut isoler

- Les definitions de broches et leurs polarites.
- La configuration des registres GPIO et timers.
- Les ISR et les details propres au microcontroleur.
- La logique de comportement : pressions, clignotements, delais et etats de l'application.

## Bonnes pratiques

- Lire le schema avant de supposer le comportement d'une entree ou sortie.
- Configurer un etat de sortie sur avant ou pendant le changement de direction pour eviter les glitches.
- Utiliser les interruptions avec parcimonie et garder les ISR tres courtes.
- Anticiper le rebond, le bruit et les entrees flottantes.
- Preferer les timers materiels et les echeances non bloquantes aux boucles d'attente.
- Verifier les calculs de timer avec la frequence d'horloge reellement configuree, pas seulement celle attendue.
- Mesurer les timings et les signaux critiques avec les outils adaptes.
- Proteger les donnees partagees entre ISR et code principal (`volatile`, sections critiques ou mecanismes adaptes).

## A retenir

Les GPIO donnent au firmware un moyen de voir et d'agir sur le monde physique ; les timers lui donnent une notion de temps fiable. Un systeme embarque robuste ne se limite pas a ecrire dans un registre : il traite les imperfections electriques et mecaniques, limite le travail dans les interruptions, evite les attentes bloquantes et isole le materiel derriere des interfaces testables.
