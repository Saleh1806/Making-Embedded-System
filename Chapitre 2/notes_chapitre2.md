# Bloc-note d’ingénieur – Chapitre 2
## Creating a System Architecture

### Idées principales

- But : obtenir une vue claire du système pour identifier dépendances, points critiques et zones de flexibilité.
- Approche : partir d’un design « OK » et l’améliorer avant l’implémentation.
- Importance du matériel : un hardware instable rend le logiciel peu fiable.

## Outils de conception

1. Block Diagram
   - Représente les composants physiques et leurs interfaces.
   - Chaque périphérique = objet, chaque bus de communication = objet.
   - Séparer communication (SPI, I²C) des périphériques (Flash, LCD).

2. Hierarchy of Control
   - Diagramme en forme d’organigramme.
   - Montre quelles fonctions dépendent des autres.
   - Met en évidence les ressources partagées (ex. Flash utilisé pour images ET texte).

3. Layered View
   - Représente les modules par couches.
   - Taille des boîtes = complexité.
   - Aide à regrouper ou séparer modules pour simplifier.

## Principes de conception

- Encapsulation : interfaces claires entre modules, indépendantes des implémentations internes.
- Minimisation des dépendances : réduire les interconnexions pour faciliter maintenance et tests.
- Prévoir le changement : concevoir des modules flexibles car matériel et besoins évoluent.
- Delegation of Tasks : découper en sous-modules attribuables à différents ingénieurs.

## Interfaces de drivers

- Inspirées du modèle Unix/POSIX :
  - open() → initialisation
  - close() → libération
  - read() → lecture
  - write() → écriture
  - ioctl() → contrôle spécifique
- Avantage : cohérence, réutilisabilité, portabilité.
- Chaque driver agit comme un Adapter Pattern : masque les détails matériels derrière une interface standard.

## Exemple pratique : Logging Interface

- Objectif : fournir un mécanisme robuste de journalisation même en environnement contraint.
- Options :
  - Sortie texte via UART ou port série.
  - Buffer interne en mémoire Flash.
  - LED clignotante (cas extrême).
- Importance : rendre le système déboguable et maintenable.


### 1. Pourquoi créer plusieurs diagrammes (block, hiérarchie, couches) ?

- Chaque vue révèle des dépendances ou des goulots d’étranglement différents.
- Permet d’anticiper les conflits de ressources et de simplifier l’architecture.

### 2. Que doit contenir une bonne interface de driver ?

- Fonctions standard (open, close, read, write, ioctl).
- Encapsulation des détails matériels.
- Cohérence pour faciliter la maintenance et la portabilité.

### 3. Comment gérer les ressources partagées (ex. Flash utilisée par plusieurs modules) ?

- Synchronisation via sémaphores ou flags.
- Planification pour éviter collisions.
- Découpage en sous-drivers spécialisés si nécessaire.

### 4. Pourquoi l’Adapter Pattern est-il utile en systèmes embarqués ?

- Permet de changer le matériel sans modifier le code haut-niveau.
- Favorise la réutilisation et la portabilité.

## Conseils pratiques pour révision

- Illustre avec un exemple concret (ex. architecture d’un projet où tu as séparé SPI et Flash).
- Mets en avant ta capacité à anticiper les changements et à concevoir des interfaces robustes.
- Prépare une explication simple du modèle de driver POSIX et de son adaptation en embarqué.
