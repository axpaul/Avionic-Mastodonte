# Mastodonte – Séquenceur 

**Mastodonte** est un ordinateur de bord ou plutot un séquenceur conçu pour piloter de manière autonome les événements critiques d’un vol de fusée, en conformité avec le cahier des charges du **C'Space**.

Il assure :
- la détection du décollage,
- l’activation des moteurs de séparation,
- le déploiement des systèmes de récupération,
- ainsi que l’enregistrement des données critique du en vol.

<p align="center">
  <img src="Image/Mastodonte-N6.png" alt="Carte Mastodonte" width="600"/>
</p>

---

## Architecture & Fonctionnalités

### Alimentation et protections
- Protection contre l’inversion de polarité (MOSFET P **SQD50P04-13L**)
- Protection contre surtensions (**TVS SMAJ14A**)
- Limitation de courant via fusible réarmable
- Régulation de tension (LM340AT : 7.4 V batterie → 5 V)
- Sélection d’alimentation (USB-C ou batterie)
- LED d’état de l’alimentation

### Microcontrôleur principal
- **RP2040** double cœur
- Interface USB-C
- 12 MB de flash externe (W25Q128)
- Prise en charge :
  - GPIO
  - I²C
  - SPI
  - UART
  - PWM
- LED RGB et bouton utilisateur embarqués

### Interfaces et connectiques
- Connecteurs au format **B2B-XH**
- Ports pour moteurs, charges pyrotechniques, capteurs et communications
- Tensions disponibles : **5 V** et **3.3 V**

---

## Interfaces et E/S

### Commande moteur / Charge pyrotechnique
- 3 drivers **DRV8872DDA** (ponts en H)
  - Jusqu’à **3.6 A** sous **6.5–45 V**
  - Contrôle **PWM**
  - LED de direction intégrées pour tests sans charge
  - Détection d’erreurs via la broche **nFAULT**

### Signaux d’entrée isolés
- **Optocoupleurs ACPL-214** pour isolation galvanique
- Buffers logiques **74HC14** pour signaux numériques
- Isolation UART et conversion de niveau : **ADuM1281**
- Pull-ups 4.7 kΩ intégrés sur les lignes I²C

### Buzzer de notification
- Pilotage via transistor **BSS138**
- Protection par diode **1N4148**
- Activation par GPIO (IN_BUZZ)

---

## Schéma fonctionnel

<p align="center">
  <img src="Image/Mastodonte_synoptique.png" alt="Synoptique Mastodonte" width="750"/>
</p>

---

## Caractéristiques mécaniques

- Dimensions : **100 × 40 mm**
- Épaisseur max : **18.13 mm**
- Trous de fixation : 4 × Ø3.2 mm
- Facilement intégrable dans une fusée ou un banc de test

<p align="center">
  <img src="Image/Mastodonte-N8.jpg" alt="Dimensions mécaniques" width="500"/>
</p>

---

## Domaines d'application

- Fusées expérimentales (C'Space, PERSEUS…)
- Charges utiles autonomes
- Plateformes de test multi-étapes
- Projets éducatifs ou amateurs avec séquence critique

---

## Ressources utiles

- [📘 Datasheet DRV8872](https://www.ti.com/lit/ds/symlink/drv8872.pdf)
- [📘 RP2040 Datasheet](https://www.raspberrypi.com/documentation/microcontrollers/rp2040.html)
- [📘 ACPL-214 Datasheet](https://www.broadcom.com/products/optocouplers/industrial-plastic/acpl-214)

---

## Contribution

Les contributions sont les bienvenues.  
Signalez un bug, proposez une amélioration ou ouvrez une *pull request* directement depuis le dépôt GitHub.

---

© 2024 – Projet Mastodonte  
Licence à préciser
