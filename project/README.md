# Robot Otto MKS — Projet de Robotique Embarquée

Projet réalisé dans le cadre du module de robotique embarquée à UniLaSalle Amiens.

---

# Sommaire

1. Présentation du projet  
2. Matériel utilisé  
3. Architecture du système  
4. Fonctionnement du programme  
5. Difficultés rencontrées  
6. Résultats obtenus  
7. Utilisation du robot  
8. Améliorations possibles  

---

# 1. Présentation du projet

Le projet consiste à programmer un robot bipède Otto MKS à l’aide d’une carte Seeed XIAO ESP32-C3.

L’objectif était de développer un robot capable de marcher en ligne droite lors d’une compétition de vitesse et de stabilité. Le robot est contrôlé à distance grâce au Bluetooth BLE avec l’application RemoteXY.

Le système comprend :
- 4 servomoteurs ;
- un capteur ultrason ;
- un buzzer ;
- une communication Bluetooth BLE.

---

# 2. Matériel utilisé

| Composant | Référence | Fonction |
|---|---|---|
| Carte microcontrôleur | Seeed XIAO ESP32-C3 | Gestion du robot |
| Servomoteurs | SG90 | Mouvement des jambes |
| Capteur ultrason | HC-SR04 | Détection d’obstacles |
| Buzzer | 5V passif | Signal sonore |
| Batterie | LiPo 3.7V | Alimentation |
| Châssis | Otto MKS | Structure du robot |

---

# 3. Architecture du système

```text
Smartphone
     |
Bluetooth BLE
     |
ESP32-C3
 |   |   |   |
Servos  Ultrason  Buzzer
```

Le smartphone envoie les commandes via Bluetooth.  
La carte ESP32 traite les informations et contrôle les différents composants du robot.

---

# 4. Fonctionnement du programme

Le programme principal est contenu dans le fichier :

```cpp
Robot_Otto_MKS.ino
```

## Principe de locomotion

Le déplacement du robot repose sur un mouvement sinusoïdal des servomoteurs.

```cpp
Hanche droite = 90 + A * sin(angle)
Hanche gauche = 90 - A * sin(angle)

Pied droit  = 90 + A * sin(angle + PI/2)
Pied gauche = 90 - A * sin(angle + PI/2)
```

Ce système permet d’obtenir une marche plus fluide et plus stable.

---

## Paramètres de déplacement

```cpp
#define PAS_ANGLE 5
#define VITESSE_PAS 8
```

- `PAS_ANGLE` : précision du mouvement ;
- `VITESSE_PAS` : vitesse de déplacement.

---

## Modes disponibles

| Mode | Description |
|---|---|
| Manuel | Contrôle avec joystick |
| Automatique | Détection d’obstacle |

En mode manuel, l’utilisateur peut :
- avancer ;
- reculer ;
- tourner ;
- lancer une animation ;
- jouer une mélodie.

---

## Correction de trajectoire

Une variable a été ajoutée pour corriger la dérive du robot :

```cpp
int biasPied = +5;
```

Cette correction permet d’équilibrer les mouvements des pieds.

---

# 5. Difficultés rencontrées

## Erreur de compilation

Erreur obtenue :

```cpp
'playMelody' was not declared in this scope
```

Cause :
fonction appelée avant sa déclaration.

Solution :
ajout des prototypes de fonctions en début de programme.

---

## Robot trop lent

Le robot avançait difficilement au début.

Causes :
- trop d’itérations ;
- lecture excessive du capteur ultrason.

Solutions :
- augmentation du pas angulaire ;
- réduction du délai ;
- optimisation de la lecture du capteur.

---

## Dérive pendant la marche

Le robot déviait progressivement vers la gauche.

Solution :
mise en place d’une correction logicielle avec `biasPied`.

---

# 6. Résultats obtenus

| Critère | Résultat |
|---|---|
| Marche fonctionnelle | Oui |
| Contrôle Bluetooth | Oui |
| Détection d’obstacle | Oui |
| Stabilité | Correcte |
| Vitesse améliorée | Oui |

Le robot a réussi à effectuer le parcours demandé pendant la compétition.

---

# 7. Utilisation du robot

## Logiciels nécessaires

- Arduino IDE
- Bibliothèque ESP32
- Bibliothèque RemoteXY
- Bibliothèque ESP32Servo

---

## Téléversement

1. Brancher le robot en USB  
2. Ouvrir le fichier `.ino`  
3. Sélectionner la carte ESP32-C3  
4. Choisir le bon port COM  
5. Cliquer sur "Téléverser"

---

## Connexion Bluetooth

1. Ouvrir l’application RemoteXY  
2. Rechercher le robot  
3. Entrer le mot de passe  
4. Utiliser l’interface de contrôle

---

# 8. Améliorations possibles

Des améliorations peuvent être ajoutées :
- intégration d’un gyroscope ;
- meilleure correction de trajectoire ;
- réglage de vitesse depuis le smartphone ;
- amélioration du mode autonome ;
- affichage du niveau de batterie.

---

# Conclusion

Ce projet a permis de mettre en pratique plusieurs notions de robotique embarquée :
- programmation ESP32 ;
- contrôle de servomoteurs ;
- communication Bluetooth ;
- traitement des capteurs.

Le robot fonctionne correctement et répond aux objectifs demandés pour la compétition.




