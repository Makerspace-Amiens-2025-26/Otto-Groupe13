# Robot Otto MKS — Projet de Robotique Embarquée

Projet réalisé dans le cadre du module de robotique embarquée à UniLaSalle Amiens.

---

# Sommaire

1. Présentation du projet et Vidéo
2. Objectifs du projet  
3. Matériel utilisé  
4. Architecture du système  
5. Fonctionnement du programme  
6. Difficultés rencontrées et solutions apportées  
7. Résultats obtenus  
8. Guide d’utilisation  
9. Améliorations possibles   
11. Conclusion  

---

# 1. Présentation du projet et Vidéo

Dans le cadre du cours de robotique embarquée, nous avons travaillé sur un robot bipède Otto MKS basé sur une carte Seeed XIAO ESP32-C3.

Le but du projet était de programmer un robot capable de marcher de manière stable et autonome tout en étant contrôlable à distance via Bluetooth BLE depuis un smartphone.

Le projet devait répondre à plusieurs contraintes :
- assurer une marche fluide ;
- limiter les déséquilibres ;
- contrôler les servomoteurs avec précision ;
- intégrer un mode automatique ;
- détecter les obstacles ;
- réussir un parcours pendant une compétition.

Le robot dispose de plusieurs éléments :
- 4 servomoteurs pour les mouvements ;
- un capteur ultrason HC-SR04 ;
- un buzzer ;
- une batterie LiPo ;
- une connexion Bluetooth BLE.

L’ensemble du projet a été programmé sous Arduino IDE en langage C++.

## Vidéo de présentation

sha256:aff76ea2632ca0c3f083fb53d9d84fd44d07dcdee51c93e69783e3b88ca61567

---

# 2. Objectifs du projet

Les objectifs principaux étaient :
- comprendre le fonctionnement d’un robot bipède ;
- utiliser un microcontrôleur ESP32 ;
- programmer des servomoteurs ;
- mettre en place une communication Bluetooth ;
- développer une logique de déplacement ;
- corriger les défauts mécaniques du robot ;
- améliorer la stabilité et la vitesse du déplacement.

Une partie importante du projet consistait aussi à effectuer plusieurs tests afin de calibrer correctement les mouvements.

---

# 3. Matériel utilisé

| Composant | Référence | Fonction |
|---|---|---|
| Carte microcontrôleur | Seeed XIAO ESP32-C3 | Gestion du programme |
| Servomoteurs | SG90 | Mouvement des jambes |
| Capteur ultrason | HC-SR04 | Détection d’obstacles |
| Buzzer | Buzzer passif 5V | Signal sonore |
| Batterie | LiPo 3.7V | Alimentation |
| Structure | Otto MKS | Châssis du robot |

---

## Connexions utilisées

| Broche ESP32 | Élément connecté |
|---|---|
| D7 | Hanche gauche |
| D8 | Hanche droite |
| D9 | Pied gauche |
| D10 | Pied droit |
| GPIO 2 | TRIG ultrason |
| GPIO 1 | ECHO ultrason |
| GPIO 0 | Buzzer |

---

# 4. Architecture du système

Le système fonctionne grâce à une communication Bluetooth entre le smartphone et le robot.

```text
Smartphone
     |
Bluetooth BLE
     |
ESP32-C3
 |   |   |   |
Servos  Ultrason  Buzzer
```

Le smartphone envoie les commandes au robot via l’application RemoteXY.

La carte ESP32 :
- reçoit les commandes ;
- calcule les mouvements ;
- contrôle les servomoteurs ;
- lit le capteur ultrason ;
- gère les différents modes.

---

# 5. Fonctionnement du programme

Le programme principal est regroupé dans un fichier unique :

```cpp
Robot_Otto_MKS.ino
```

---

## Principe de locomotion

Le déplacement du robot repose sur des fonctions sinusoïdales.

Chaque servo suit une trajectoire périodique afin de reproduire un mouvement de marche.

```cpp
Hanche droite = 90 + A * sin(angle)
Hanche gauche = 90 - A * sin(angle)

Pied droit  = 90 + A * sin(angle + PI/2)
Pied gauche = 90 - A * sin(angle + PI/2)
```

Le décalage de phase permet d’obtenir une alternance entre les jambes.

Quand une jambe avance, l’autre reste en appui pour maintenir l’équilibre.

---

## Paramètres de déplacement

```cpp
#define PAS_ANGLE 5
#define VITESSE_PAS 8
```

Ces paramètres permettent de régler :
- la fluidité des mouvements ;
- la vitesse du robot ;
- le nombre d’itérations.

---

## Modes de fonctionnement

Le robot possède deux modes.

### Mode manuel

Le robot est contrôlé depuis le smartphone :
- avancer ;
- reculer ;
- tourner ;
- jouer une mélodie ;
- effectuer un mouvement de salutation.

### Mode automatique

Le robot avance seul et s’arrête lorsqu’un obstacle est détecté grâce au capteur ultrason.

---

## Détection d’obstacle

Le capteur HC-SR04 mesure la distance devant le robot.

```cpp
if (distance < DISTANCE_OBSTACLE_CM) {
    positionRepos();
}
```

Le robot passe alors en position de repos afin d’éviter une collision.

---

## Correction de trajectoire

Pendant les essais, le robot avait tendance à dévier vers la gauche.

Une variable de correction a été ajoutée :

```cpp
int biasPied = +5;
```

Cette variable modifie légèrement l’amplitude d’un pied afin de compenser l’asymétrie mécanique.

---

# 6. Difficultés rencontrées et solutions apportées

## 6.1 Erreur de compilation

Lors des premiers essais, une erreur de compilation apparaissait :

```cpp
'playMelody' was not declared in this scope
```

### Cause

La fonction était appelée avant sa déclaration.

### Solution

Ajout des prototypes de fonctions au début du programme :

```cpp
void playMelody();
void marche(int direction);
void tourner(int direction);
```

---

## 6.2 Robot trop lent

Le robot avançait très lentement au début du projet.

### Causes
- trop d’itérations dans les boucles ;
- délai trop élevé entre les mouvements ;
- lecture du capteur ultrason trop fréquente.

### Solution

Les paramètres ont été modifiés :

```cpp
#define PAS_ANGLE 10
#define VITESSE_PAS 3
```

Le capteur ultrason a également été lu une seule fois par cycle.

### Résultat

Le robot est devenu beaucoup plus rapide.

---

## 6.3 Robot qui glissait

Le robot glissait sur la table sans réellement marcher.

### Cause

Les pieds ne se levaient pas suffisamment pendant le mouvement.

### Solution

Modification du déphasage entre les jambes et les pieds ainsi qu’augmentation de l’amplitude.

Cela a permis d’obtenir une marche plus naturelle.

---

## 6.4 Dérive pendant la marche

Après plusieurs pas, le robot déviait progressivement vers la gauche.

### Analyse

Le problème ne venait pas du montage mécanique mais d’un déséquilibre dans le cycle de marche.

### Solution

Ajout de la variable :

```cpp
int biasPied = +5;
```

Cette correction améliore la trajectoire du robot.

---

## 6.5 Instabilité des servomoteurs

Certains servos vibraient fortement.

### Causes possibles
- alimentation insuffisante ;
- bruit électrique ;
- angles trop importants.

### Solution
- limitation des amplitudes ;
- meilleure fixation des servos ;
- alimentation plus stable.

---

## 6.6 Problèmes Bluetooth

Au début, la connexion BLE se déconnectait régulièrement.

### Solution
- réduction du nombre de données envoyées ;
- simplification de l’interface RemoteXY ;
- redémarrage propre de la connexion.

---

# 7. Résultats obtenus

| Critère | Résultat |
|---|---|
| Marche fonctionnelle | Oui |
| Contrôle Bluetooth | Fonctionnel |
| Détection d’obstacle | Fonctionnelle |
| Vitesse améliorée | Oui |
| Stabilité correcte | Oui |
| Parcours réalisé | Oui |

Le robot a réussi à terminer le parcours demandé pendant la compétition.

Même si une légère dérive restait présente, les performances obtenues étaient satisfaisantes.

---

# 8. Guide d’utilisation

## Logiciels nécessaires

- Arduino IDE
- Bibliothèque ESP32
- Bibliothèque RemoteXY
- Bibliothèque ESP32Servo

---

## Téléversement du programme

1. Brancher le robot en USB  
2. Ouvrir le fichier `.ino`  
3. Sélectionner la carte `XIAO ESP32C3`  
4. Choisir le bon port COM  
5. Cliquer sur "Téléverser"

---

## Connexion Bluetooth

1. Ouvrir l’application RemoteXY  
2. Rechercher le robot  
3. Entrer le mot de passe  
4. Utiliser les commandes à l’écran

---

## Commandes disponibles

| Commande | Fonction |
|---|---|
| Joystick haut | Avancer |
| Joystick bas | Reculer |
| Joystick gauche | Tourner à gauche |
| Joystick droite | Tourner à droite |
| Bouton 1 | Saluer |
| Bouton 2 | Jouer une mélodie |

---

# 9. Améliorations possibles

Plusieurs améliorations pourraient être ajoutées au projet :

- ajout d’un gyroscope MPU6050 ;
- correction automatique de trajectoire ;
- réglage de vitesse via smartphone ;
- meilleure autonomie de batterie ;
- ajout d’un écran OLED ;
- amélioration de l’évitement d’obstacles ;
- optimisation de la stabilité.

---

# 10. Conclusion

Ce projet nous a permis de découvrir plusieurs notions importantes en robotique embarquée.

Nous avons appris à :
- programmer un ESP32 ;
- utiliser des servomoteurs ;
- gérer une communication Bluetooth ;
- contrôler des mouvements mécaniques ;
- corriger des problèmes de stabilité.

Malgré plusieurs difficultés techniques rencontrées pendant le développement, le robot a réussi à fonctionner correctement et à répondre aux objectifs fixés pour la compétition.

Ce projet a également permis de mieux comprendre les contraintes réelles liées à la robotique mobile et à la programmation embarquée.





