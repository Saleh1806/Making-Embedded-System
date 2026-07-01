Chapitre 1 – Introduction aux systèmes embarqués

Résumé du chapitre 1

Le premier chapitre pose les bases des systèmes embarqués :

- Définition : un système embarqué est un ordinateur spécialisé conçu pour accomplir une tâche précise (voiture, appareil médical, électroménager, etc.).
- Contraintes : mémoire limitée, processeur peu puissant, consommation énergétique réduite, fiabilité critique et disponibilité souvent requise.
- Langages : C et C++ dominent le développement embarqué, avec des restrictions pratiques à respecter (exceptions, héritage multiple, allocation dynamique, etc.).
- Outils : cross-compilateurs, débogueurs embarqués, interfaces JTAG/SWD et environnements de développement adaptés.
- Défis : trouver le bon équilibre entre performances et consommation d’énergie, gérer les incertitudes matérielles, assurer la maintenance et l’évolutivité.
- Bonnes pratiques : organiser le code modularisé, documenter clairement, tester tôt et fréquemment, et éviter l’optimisation prématurée.

Notes

1. Différence entre microcontrôleur et microprocesseur

- Microcontrôleur : intègre CPU, mémoire et périphériques sur une même puce. Idéal pour les systèmes embarqués compacts et économes.
- Microprocesseur : nécessite des composants externes (RAM, périphériques, contrôleurs). Plus flexible, mais souvent plus coûteux et gourmand en énergie.

2. Gestion de la mémoire limitée

- Utiliser des structures de données compactes.
- Préférer les variables locales aux variables globales.
- Exploiter la mémoire non volatile (EEPROM, Flash) pour les données persistantes.
- Éviter les allocations dynamiques excessives et la fragmentation mémoire.

3. C vs C++ en systèmes embarqués

- C : simple, efficace, très proche du matériel. Souvent utilisé pour les couches basses et le code temps réel.
- C++ : apporte modularité, abstraction et réutilisabilité. Mais certaines fonctionnalités comme les exceptions, le RTTI et l’allocation dynamique peuvent être coûteuses.
- Bon compromis : utiliser C++ avec discipline, en privilégiant des classes légères, une hiérarchie simple et en évitant les fonctionnalités coûteuses.

4. Interruptions

- Les interruptions permettent de réagir rapidement aux événements matériels.
- Il est essentiel de gérer les priorités et de minimiser la latence des routines d’interruption.
- Risques : interblocages, corruption des sections critiques et problèmes avec les ressources partagées.

5. Temps réel

- Hard real-time : les échéances sont strictes et doivent toujours être respectées (ex. pacemaker, contrôleur de freinage).
- Soft real-time : les retards sont tolérables, mais la qualité de service peut se dégrader (ex. streaming audio).
- Solutions : planification déterministe, gestion des priorités et utilisation de watchdog timers pour détecter les blocages.

6. Optimisation

- Ne pas optimiser trop tôt.
- Prioriser la lisibilité et la maintenabilité du code.
- Optimiser uniquement les parties critiques identifiées par profilage.

7. Débogage sans console classique

- Utiliser LED, UART, GPIO ou journaux matériels pour tracer l’exécution.
- Exploiter les simulateurs, émulateurs et débogueurs matériels.
- Ajouter des assertions et des tests unitaires pour valider le comportement dès que possible.
