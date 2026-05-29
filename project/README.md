# 🤖 Robot Otto MKS — Projet Robotique

**École d'ingénieurs UniLaSalle Amiens**  
**Projet de robotique embarquée — Compétition de marche bipède**

---

## Sommaire

1. [Présentation du projet](#1-présentation-du-projet)
2. [Matériel utilisé](#2-matériel-utilisé)
3. [Architecture du système](#3-architecture-du-système)
4. [Description du code](#4-description-du-code)
5. [Problèmes rencontrés et solutions](#5-problèmes-rencontrés-et-solutions)
6. [Résultats de la compétition](#6-résultats-de-la-compétition)
7. [Guide d'utilisation](#7-guide-dutilisation)
8. [Pistes d'amélioration](#8-pistes-damélioration)

---

## 1. Présentation du projet

Ce projet consiste à programmer et calibrer un robot bipède **Otto MKS** à base de microcontrôleur **Seeed XIAO ESP32-C3**, dans le cadre d'une compétition de marche en ligne droite sur table.

L'objectif de la compétition était de faire parcourir au robot une distance définie entre un départ et une arrivée, le plus droit et le plus rapidement possible, en le contrôlant via une interface **Bluetooth BLE** depuis un smartphone.

Le robot dispose de :
- 4 servomoteurs pilotant les hanches et les pieds
- Un capteur ultrasonique pour la détection d'obstacle
- Un buzzer pour les signaux sonores
- Une connexion Bluetooth BLE via l'application **RemoteXY**

---

## 2. Matériel utilisé

| Composant | Référence | Rôle |
|---|---|---|
| Microcontrôleur | Seeed XIAO ESP32-C3 | Cerveau du robot, gestion BLE |
| Servomoteurs (×4) | SG90 | Hanches gauche/droite, pieds gauche/droit |
| Capteur ultrason | HC-SR04 | Détection d'obstacle |
| Buzzer | Buzzer passif 5V | Signal sonore au démarrage |
| Batterie | LiPo 3.7V / 500mAh | Alimentation embarquée |
| Structure | Kit Otto MKS | Châssis imprimé en 3D |

### Câblage des servomoteurs

| Broche ESP32 | Servomoteur |
|---|---|
| D7 | Jambe gauche (hanche gauche) |
| D8 | Jambe droite (hanche droite) |
| D9 | Pied gauche |
| D10 | Pied droit |
| GPIO 2 | TRIG capteur ultrason |
| GPIO 1 | ECHO capteur ultrason |
| GPIO 0 | Buzzer |

---

## 3. Architecture du système

```
┌─────────────────────────────────────────┐
│           Smartphone (RemoteXY)         │
│  Joystick / Boutons / Sélecteur mode    │
└────────────────────┬────────────────────┘
                     │ Bluetooth BLE
┌────────────────────▼────────────────────┐
│         XIAO ESP32-C3                   │
│  - Réception commandes BLE              │
│  - Calcul trajectoires sinusoïdales     │
│  - Gestion capteur ultrason             │
│  - Génération signaux PWM servos        │
└──┬──────┬──────┬──────┬─────────────────┘
   │      │      │      │
  JG     JD     PG     PD       Buzzer / Ultrason
(D7)   (D8)   (D9)  (D10)
```

---

## 4. Description du code

Le projet est contenu dans un seul fichier `Robot_Otto_MKS.ino`.

### 4.1 Principe de locomotion

La marche bipède repose sur une **locomotion sinusoïdale** : chaque servo suit une trajectoire en sinus, avec un décalage de phase de 90° entre les hanches et les pieds.

```
Hanche droite  :  90 + A × sin(θ)
Hanche gauche  :  90 - A × sin(θ)        ← opposition de phase
Pied droit     :  90 + A × sin(θ + 90°)  ← décalage 90°
Pied gauche    :  90 - A × sin(θ + 90°)  ← décalage 90°
```

Ce principe garantit qu'un pied se lève exactement quand la hanche correspondante avance, reproduisant un cycle de marche naturel.

### 4.2 Paramètres de vitesse

```cpp
#define PAS_ANGLE    5   // incrément angulaire par itération (°)
#define VITESSE_PAS  8   // délai entre chaque pas (ms)
// Durée d'un cycle = (360 / 5) × 8 = 576 ms
```

### 4.3 Modes de fonctionnement

Le sélecteur de l'interface RemoteXY permet de basculer entre deux modes :

| Mode | Sélecteur | Comportement |
|---|---|---|
| Télécommande | Position 0 | Joystick contrôle direction, boutons pour saluer et mélodie |
| Évitement auto | Position 1 | Le robot avance seul et contourne les obstacles |

### 4.4 Correction de trajectoire

Pour compenser une dérive mécanique, une variable `biasPied` agit asymétriquement sur l'amplitude des pieds :

```cpp
int biasPied = +5;  // positif = correction dérive gauche
                    // négatif = correction dérive droite

int pd = 90 + offsetPD + (amplitudePied + biasPied) * sin(rad + PI/2);
int pg = 90 + offsetPG - (amplitudePied - biasPied) * sin(rad + PI/2);
```

### 4.5 Détection d'obstacle

Le capteur HC-SR04 est interrogé une fois par cycle de marche (et non à chaque pas) pour éviter de bloquer la boucle de contrôle :

```cpp
if (direction == 1 && lireDistance() < DISTANCE_OBSTACLE_CM) {
    positionRepos();
    return;
}
```

---

## 5. Problèmes rencontrés et solutions

### 5.1 Erreur de compilation — fonction non déclarée

**Problème** : `'playMelody' was not declared in this scope`

**Cause** : En C++ sur ESP32, les fonctions doivent être déclarées avant leur premier appel. Les fonctions définies après `setup()` et `loop()` ne sont pas reconnues sans prototype.

**Solution** : Ajout des prototypes en tête de fichier :
```cpp
void positionRepos();
void marche(int direction);
void tourner(int direction);
void saluer();
void evitementAuto();
void playMelody();
float lireDistance();
```

---

### 5.2 Robot trop lent

**Problème** : Le robot avançait avec un cycle de ~430 ms, trop lent pour la compétition.

**Cause** : `VITESSE_PAS = 6 ms` avec `PAS_ANGLE = 5°` donnait 72 itérations × 6 ms = 432 ms par cycle. De plus, la lecture du capteur ultrason à chaque pas ajoutait ~30 ms × 72 = 2 secondes de blocage par cycle.

**Solution** :
- `PAS_ANGLE` passé de 5° à 10° (moins d'itérations)
- `VITESSE_PAS` réduit de 6 ms à 3 ms
- Lecture ultrason déplacée en dehors de la boucle (une fois par cycle)
- Résultat : cycle de ~108 ms, soit **4× plus rapide**

---

### 5.3 Robot ne levait pas les pieds

**Problème** : Le robot glissait au sol sans lever les pieds.

**Cause** : Le déphasage entre hanches et pieds était asymétrique (`sin(rad - PI/2)` d'un côté, `sin(rad + PI/2)` de l'autre), créant un déséquilibre dans la levée.

**Solution** : Harmonisation du déphasage en `sin(rad + PI/2)` symétrique et augmentation de l'amplitude de levée de 25° à 35°.

---

### 5.4 Dérive vers la gauche

**Problème** : Le robot dévie progressivement vers la gauche après 3-4 pas.

**Cause identifiée** : La dérive apparaissant après plusieurs pas (et non dès le premier) indique une **accumulation d'asymétrie** dans le cycle de marche, et non un problème de calibration mécanique statique. Le robot étant bien droit au repos et les cornes de servos montées à 90°, la cause est un déséquilibre dans le temps d'appui entre le pied droit et le pied gauche.

**Solution** : Introduction d'un paramètre `biasPied` qui augmente l'amplitude du pied droit et réduit celle du pied gauche, allongeant la phase d'appui côté droit pour compenser la dérive gauche.

```cpp
int biasPied = +5;
```

---

### 5.5 Incohérence entre les deux fichiers initiaux

**Problème** : Les fichiers `Robot_Otto_MKS.ino` et `move.h` originaux utilisaient des noms de servos différents (`jambeDroite` vs `hancheDroite`) et des numéros de broches contradictoires.

**Solution** : Fusion en un seul fichier `.ino` avec une convention de nommage unique et cohérente.

---

## 6. Résultats de la compétition

| Critère | Résultat |
|---|---|
| Parcours complété | ✅ Oui |
| Marche en ligne droite | ⚠️ Dérive résiduelle corrigée par `biasPied` |
| Détection d'obstacle | ✅ Fonctionnelle |
| Connexion BLE stable | ✅ Oui |

---

## 7. Guide d'utilisation

### 7.1 Prérequis logiciels

- [Arduino IDE](https://www.arduino.cc/en/software) version 2.x
- Carte ESP32 : package `esp32` version 3.3.8 via le gestionnaire de cartes
- Bibliothèques à installer via le gestionnaire de bibliothèques :
  - `RemoteXY` v4.1.10
  - `ESP32Servo` v3.2.0
  - `BLE` (incluse dans le package ESP32)
- Application **RemoteXY** sur smartphone (iOS ou Android)

### 7.2 Configuration Arduino IDE

1. Ouvrir Arduino IDE
2. Aller dans `Fichier > Préférences` et ajouter l'URL de gestionnaire de cartes ESP32 :
   ```
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   ```
3. Aller dans `Outils > Type de carte > Gestionnaire de cartes`, chercher `esp32` et installer
4. Sélectionner la carte : `Outils > Type de carte > ESP32 Arduino > XIAO_ESP32C3`
5. Sélectionner le port COM correspondant au robot

### 7.3 Téléversement

1. Brancher le robot en USB
2. Ouvrir `Robot_Otto_MKS.ino` dans Arduino IDE
3. Cliquer sur **Téléverser** (flèche →)
4. Attendre le message `Leaving... Hard resetting via RTS pin...`

### 7.4 Connexion Bluetooth

1. Ouvrir l'application **RemoteXY** sur le smartphone
2. Appuyer sur `+` puis `Bluetooth`
3. Sélectionner `OTTOMAN` dans la liste
4. Entrer le mot de passe : `ines`
5. L'interface de contrôle apparaît automatiquement

### 7.5 Interface de contrôle

| Élément | Fonction |
|---|---|
| Joystick ↑ | Avancer |
| Joystick ↓ | Reculer |
| Joystick ← | Tourner à gauche |
| Joystick → | Tourner à droite |
| Bouton 1 | Saluer (lever le pied gauche) |
| Bouton 2 | Jouer la mélodie |
| Sélecteur position 0 | Mode télécommande |
| Sélecteur position 1 | Mode évitement automatique |

### 7.6 Calibration

Si le robot dérive, modifier la valeur `biasPied` dans le code :

```cpp
int biasPied = +5;  // Dérive à gauche → augmenter (+3, +5, +7, +9)
                    // Dérive à droite → diminuer (-3, -5, -7, -9)
```

Retéléverser après chaque modification et tester sur une surface plane.

---

## 8. Pistes d'amélioration

- **Gyroscope MPU-6050** : ajouter un capteur IMU pour détecter et corriger la dérive en temps réel de façon automatique, sans calibration manuelle
- **Vitesse variable** : exposer `VITESSE_PAS` dans l'interface RemoteXY pour ajuster la vitesse depuis le smartphone sans recompiler
- **Marche arrière symétrique** : affiner le déphasage pour la marche arrière qui reste moins stable que la marche avant
- **Batterie avec indicateur** : ajouter une lecture de tension batterie pour prévenir les coupures en compétition
- **Mode autonome amélioré** : intégrer une logique de navigation plus élaborée (tourner à gauche si obstacle, puis réévaluer) plutôt qu'un simple demi-tour fixe

---

*Projet réalisé dans le cadre du cours de robotique embarquée — UniLaSalle Amiens*
