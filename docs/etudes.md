# Étude et choix techniques

## Introduction

Dans le cadre de ce projet, nous avons réalisé la programmation d'un robot Otto à l'aide d'une carte ESP32 et de quatre servomoteurs. L'objectif principal était de permettre au robot d'effectuer différents déplacements tels que la marche avant, les rotations vers la gauche et la droite ainsi que l'arrêt dans une position stable. Ce projet nous a permis d'appliquer des notions de programmation embarquée, de contrôle de moteurs et d'organisation du code.

## Analyse des besoins

Avant de commencer le développement, nous avons étudié les fonctionnalités que le robot devait réaliser. Le robot devait être capable de :

- Se tenir debout de manière stable.
- Effectuer des mouvements fluides.
- Marcher vers l'avant.
- Tourner à gauche et à droite.
- Revenir à une position neutre après chaque mouvement.
- Être facilement modifiable pour ajouter de nouvelles fonctionnalités.

Ces besoins nous ont conduits à choisir une architecture logicielle simple et modulaire afin de faciliter la compréhension du programme et sa maintenance.

## Choix du matériel

### Carte ESP32

La carte ESP32 a été choisie comme unité principale de contrôle. Cette carte présente plusieurs avantages :

- Une puissance de calcul supérieure à celle d'un Arduino Uno.
- Une gestion efficace des signaux PWM nécessaires au pilotage des servomoteurs.
- Une faible consommation énergétique.
- La présence du Wi-Fi et du Bluetooth permettant d'envisager des évolutions futures du projet.

L'ESP32 constitue donc une solution adaptée pour le contrôle d'un robot bipède comme Otto.

### Servomoteurs

Le robot utilise quatre servomoteurs :

- Un servomoteur pour la hanche gauche.
- Un servomoteur pour la hanche droite.
- Un servomoteur pour le pied gauche.
- Un servomoteur pour le pied droit.

Ces moteurs permettent de reproduire les mouvements nécessaires à la marche et aux rotations.

## Choix du langage de programmation

Le programme a été développé en langage C++ à l'aide de l'environnement Arduino IDE. Ce choix a été motivé par plusieurs raisons :

- Compatibilité directe avec l'ESP32.
- Documentation abondante.
- Grande communauté d'utilisateurs.
- Facilité de développement et de débogage.

## Utilisation de la bibliothèque ESP32Servo

Pour piloter les servomoteurs, nous avons utilisé la bibliothèque ESP32Servo. Cette bibliothèque simplifie la génération des signaux PWM nécessaires au fonctionnement des moteurs et permet un contrôle précis des angles.

## Architecture du programme

Le programme est organisé en plusieurs fonctions afin de faciliter sa compréhension.

### Fonction d'initialisation

La fonction `setup()` initialise la communication série, configure les timers de l'ESP32, attache les servomoteurs à leurs broches respectives et place le robot dans sa position de départ.

### Gestion des positions

Un tableau nommé `posActuelle` est utilisé pour mémoriser en permanence la position de chaque servomoteur.

### Mouvement simultané des moteurs

La fonction `bougerSimultanement()` constitue l'élément central du programme. Elle permet de déplacer les quatre servomoteurs de manière progressive et synchronisée.

Cette méthode permet :

- D'obtenir des mouvements plus fluides.
- De limiter les secousses.
- D'améliorer la stabilité du robot.
- De réduire les contraintes mécaniques sur les servomoteurs.

### Marche avant

La fonction `marcherAvant()` réalise un cycle complet de marche grâce à plusieurs déplacements successifs des servomoteurs.

### Rotation à gauche et à droite

Les fonctions `tournerGauche()` et `tournerDroite()` permettent au robot de changer de direction en modifiant les positions des hanches et des pieds.

### Arrêt du robot

La fonction `stopperMouvement()` replace l'ensemble des servomoteurs à 90 degrés afin de remettre le robot dans une position neutre.

## Gestion de la fluidité des mouvements

Pour rendre les déplacements plus naturels, le programme utilise un système de mouvements progressifs basé sur un nombre d'étapes et un délai entre chaque étape.

Cette technique permet d'améliorer la qualité des déplacements et de réduire les mouvements brusques.

## Difficultés rencontrées

Plusieurs difficultés ont été rencontrées durant le développement :

- Synchronisation des servomoteurs.
- Maintien de l'équilibre du robot.
- Réglage des angles de déplacement.
- Fluidité des mouvements.

Ces difficultés ont été résolues grâce à plusieurs phases de tests et d'ajustements.

## Perspectives d'amélioration

Des améliorations futures peuvent être envisagées :

- Contrôle Bluetooth.
- Application mobile.
- Détection d'obstacles.
- Nouvelles animations.
- Déplacements autonomes.

## Conclusion

Ce projet a permis de développer un robot Otto capable de réaliser différents mouvements grâce à l'utilisation d'une carte ESP32 et de quatre servomoteurs. Les choix techniques effectués ont permis d'obtenir un programme structuré, évolutif et relativement simple à maintenir.
