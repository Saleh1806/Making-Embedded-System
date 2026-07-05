# Bloc-note d'ingenieur - Chapitre 3
## Getting the Code Working

### Idee centrale

Ce chapitre explique le passage entre une architecture sur papier et un systeme qui fonctionne reellement sur une carte. En embarque, faire marcher le code ne signifie pas seulement compiler : il faut verifier que le materiel est alimente, que le processeur demarre, que les horloges sont correctes, que les broches sont bien configurees et que les peripheriques repondent comme prevu.

Un ingenieur systemes embarques doit donc penser logiciel et materiel en meme temps. Le code n'est qu'une partie du systeme.

## Board bring-up

Le board bring-up est la premiere mise en route progressive d'une carte.

### 1. Avant d'executer du code

- Lire le schema electronique pour comprendre les alimentations, les signaux de reset, les horloges, les bus et les peripheriques connectes.
- Verifier les tensions avec un multimetre avant de connecter un debugger ou de flasher le microcontroleur.
- Controler que les masses sont communes et que les polarites sont correctes.
- Identifier les broches critiques : reset, boot mode, clock, alimentation, UART de debug, LED, JTAG/SWD.
- Lire les parties utiles des datasheets : pas tout le document, mais les registres, timings, modes de boot et contraintes electriques necessaires au demarrage.

Point important : ne jamais supposer que la carte fonctionne. Chaque hypothese doit etre verifiee.

### 2. Premier code

Le premier programme doit etre minimal. Son objectif n'est pas d'implementer une fonctionnalite produit, mais de prouver que le processeur execute bien des instructions.

Exemples de premiers tests :

- Faire clignoter une LED.
- Envoyer un caractere sur UART.
- Lire un registre d'identification d'un peripherique SPI ou I2C.
- Basculer une broche GPIO et observer le signal a l'oscilloscope.

La LED clignotante est utile parce qu'elle valide plusieurs choses a la fois : alimentation, clock, execution du code, configuration GPIO et boucle principale.

## Outils indispensables

### Multimetre

- Verifie les tensions d'alimentation.
- Detecte les courts-circuits simples.
- Controle la continuite des pistes ou connexions.

### Oscilloscope

- Observe les signaux rapides.
- Verifie les horloges, fronts, niveaux logiques et timings.
- Permet de voir si une broche change vraiment d'etat.

### Analyseur logique

- Capture et decode les communications numeriques.
- Tres utile pour SPI, I2C, UART et bus paralleles.
- Aide a distinguer un bug logiciel d'un mauvais cablage ou d'une mauvaise configuration de protocole.

### Debugger JTAG/SWD

- Permet de flasher le programme.
- Sert a executer le code pas a pas.
- Permet de lire registres CPU, memoire, pile et registres peripheriques.

Attention : le debugger peut parfois modifier le comportement temporel du systeme. Un bug qui disparait en mode debug reste un vrai bug.

## Methode de developpement

### Avancer par petites preuves

L'approche recommandee est incrementale :

1. Verifier l'alimentation.
2. Verifier le reset et la clock.
3. Flasher un programme minimal.
4. Tester une sortie simple, comme une LED ou une broche GPIO.
5. Ajouter UART pour obtenir une trace de debug.
6. Tester les peripheriques un par un.
7. Integrer progressivement les modules.

Chaque etape doit produire une preuve observable : tension mesuree, signal visible, message UART, valeur de registre, retour de fonction ou resultat de test.

### Ne pas tout integrer trop tot

Un systeme complet qui ne demarre pas est difficile a diagnostiquer. Il vaut mieux construire des tests simples pour isoler chaque sous-systeme :

- GPIO
- UART
- SPI
- I2C
- timers
- interruptions
- memoire Flash
- capteurs
- actionneurs

Le but est de reduire l'espace de recherche quand une panne apparait.

## Tests embarques

Les tests en embarque ne ressemblent pas toujours aux tests logiciels classiques, car ils doivent parfois interagir avec du vrai materiel.

### Types de tests utiles

- Test de demarrage : verifier que le firmware atteint bien la boucle principale.
- Test memoire : ecrire, lire et comparer des motifs connus.
- Test peripherique : lire un registre d'identification ou effectuer une transaction simple.
- Test GPIO : forcer un etat haut/bas et mesurer le resultat.
- Test de communication : envoyer une trame connue et verifier la reponse.
- Test de regression : s'assurer qu'une correction ne casse pas une fonction deja validee.

### Exemple : test simple de memoire Flash

Objectif :

- Ecrire une valeur connue dans une zone autorisee.
- Relire la valeur.
- Comparer le resultat.
- Retourner un code d'erreur clair en cas d'echec.

Points a surveiller :

- Ne jamais ecrire dans une zone contenant le firmware.
- Respecter l'alignement impose par la Flash.
- Effacer le secteur avant d'ecrire si la technologie l'exige.
- Tenir compte de l'usure de la Flash.
- Proteger ce test pour qu'il ne soit pas lance accidentellement en production.

## Command Pattern

Le chapitre met en avant l'interet d'une interface de commandes pour tester le systeme.

Principe :

- Une commande texte ou binaire arrive par UART, USB, BLE ou autre interface.
- Le firmware associe cette commande a une fonction.
- La fonction execute une action precise et retourne un resultat.

Exemples :

- `led on`
- `led blink 5`
- `flash test`
- `i2c scan`
- `sensor read`
- `version`
- `reboot`

Interet pour l'ingenieur embarque :

- Tester rapidement une fonctionnalite sans recompiler.
- Isoler un driver.
- Automatiser des tests.
- Donner un outil de diagnostic a l'equipe hardware, validation ou production.

## Gestion des erreurs

Une bonne gestion d'erreurs evite les comportements aleatoires.

### Bonnes pratiques

- Centraliser les codes d'erreur.
- Utiliser des noms explicites : `ERR_TIMEOUT`, `ERR_INVALID_PARAM`, `ERR_CRC`, `ERR_NOT_READY`.
- Retourner les erreurs au lieu de les masquer.
- Journaliser les erreurs critiques quand une interface de log existe.
- Prevoir une reaction claire : retry, reset du peripherique, passage en mode degrade, arret securise.

### Watchdog et recuperation

Le watchdog sert a redemarrer le systeme s'il reste bloque. Il ne remplace pas une bonne conception, mais il protege contre certains blocages.

Il faut comprendre :

- Quand le watchdog est active.
- Quelle partie du code le rafraichit.
- Quels blocages il peut detecter.
- Quels blocages il ne peut pas detecter.
- Comment diagnostiquer la cause apres redemarrage.

## Ce qu'il faut vraiment comprendre

### 1. Le materiel peut mentir au logiciel

Un driver peut etre correct mais echouer a cause d'une alimentation instable, d'une broche mal routee, d'une mauvaise resistance de pull-up ou d'une horloge absente.

### 2. Une mesure vaut mieux qu'une supposition

En embarque, il faut mesurer :

- tensions
- signaux
- timings
- consommation
- etats des registres
- traces de communication

### 3. Le code minimal est un outil de diagnostic

Un petit programme qui teste une seule chose est souvent plus utile qu'une application complete. Il permet de savoir exactement ce qui marche et ce qui ne marche pas.

### 4. Les datasheets sont des documents de travail

Il ne suffit pas de lire la description generale. Il faut apprendre a chercher :

- sequence d'initialisation
- contraintes de timing
- registres importants
- valeurs par defaut
- modes basse consommation
- conditions d'erreur

### 5. Les erreurs doivent etre concues

Un systeme robuste ne se contente pas de fonctionner quand tout va bien. Il doit avoir un comportement defini quand un peripherique ne repond pas, quand une ecriture echoue ou quand une communication expire.

## A retenir

Le chapitre 3 montre que le role d'un ingenieur systemes embarques est hybride : programmer, lire le schema, mesurer les signaux, comprendre le processeur, diagnostiquer les peripheriques et construire une methode de test fiable.

La bonne attitude est simple : avancer progressivement, verifier chaque hypothese et transformer chaque test en preuve observable.

