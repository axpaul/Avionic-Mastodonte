# Mastodonte – Séquenceur

**Mastodonte** est un ordinateur de bord, ou plus précisément un **séquenceur**, destiné à orchestrer de manière autonome les événements critiques d’un vol de fusée expérimentale. Il est conçu pour répondre aux exigences du **cahier des charges du C'Space**, et s’intègre à tout projet nécessitant un séquencement fiable, robuste et temps réel.

---

> *© 2025 Paul Miailhe, all rights reserved.*  
> *Material licensed under [→ CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0)*

---

<p align="center">
  <img src="Image/Mastodonte-N6.png" alt="Carte Mastodonte" width="800"/>
</p>

---

## Fonctionnement

Mastodonte supervise et déclenche en toute autonomie les étapes clés du vol, sans intervention extérieure :

### Détection du décollage
La détection initiale peut se faire par deux méthodes :
- Un **contact jack** (fil coupe-circuit) : un câble fin relie la fusée au sol, et se sectionne au moment du lancement. La coupure génère une interruption matérielle, qui marque le T-0 du vol.
- Un **accéléro-contact** (contacteur à masselotte) peut également déclencher le séquenceur dès détection d’une accélération.

Dans les deux cas, la carte démarre son chronomètre interne pour initier la séquence.

### Fenêtrage temporel autour de l’apogée
Après le décollage, Mastodonte ouvre une **fenêtre temporelle** durant laquelle la détection de l’apogée est autorisée :
- Un signal extérieur, par exemple issu d’un autre système avionique ou d’un capteur, peut être reçu via un **optocoupleur** pour indiquer l’apogée.
- Ce signal n’est pris en compte **que pendant la fenêtre**, afin d’écarter toute détection parasite en dehors de cette phase critique.
- Si aucune détection n’est reçue, un **déclenchement de sécurité** est effectué après un **temps maximal prédéfini** post-apogée, garantissant ainsi l’ouverture du système de récupération.

### Déploiement du système de récupération
En réponse à une détection d’apogée (ou par sécurité en fin de fenêtre), Mastodonte déclenche le système de récupération :
- Par **charge pyrotechnique** (détonateur, coupe-corde, etc.),
- Ou par **moteur DC** (vis de poussée, libération mécanique, etc.).

### Enregistrement des événements critiques
Mastodonte assure un enregistrement précis des points clés du vol :
- Heure de **décollage** (T-0),
- **Début et fin** de la fenêtre d’apogée,
- **Réception (ou absence)** du signal d’apogée,
- **Déclenchement** du système de récupération,
- **Tension batterie** mesurée durant le vol,
- **Événements d’erreur moteur** détectés.

Ce journal d’exécution agit comme une **boîte noire embarquée**, permettant d’analyser a posteriori le bon déroulement de la mission, et d’identifier toute anomalie de comportement.

---

> Mastodonte vise à offrir un **fonctionnement déterministe**, **fiable** et **autonome**, même en l’absence de télémétrie ou de supervision au sol, conformément à l’esprit du **cahier des charges C'Space**.

---

## Architecture & Fonctionnalités

### Alimentation et protections
- Protection contre l’inversion de polarité (MOSFET P **SQD50P04-13L**)
- Protection contre surtensions (**TVS SMAJ14A**)
- Limitation de courant via fusible réarmable
- Régulation de tension (LM340AT : 7.4 V batterie → 5 V)
- Sélection de la source d’alimentation (USB-C ou batterie)
- LED d’indication d’alimentation

### Microcontrôleur RP2040-YD

Mastodonte s’appuie sur la carte **RP2040-YD**, une carte commerciale compacte embarquant un microcontrôleur **RP2040** de Raspberry Pi, accompagnée de ses périphériques essentiels :

- Double cœur **ARM Cortex-M0+** cadencé à 133 MHz
- **128 Mbits (16 MB)** de mémoire flash externe **W25Q128**
- Oscillateur intégré à **12 MHz**
- Interface native **USB-C**
- Format physique compatible **Raspberry Pi Pico**
- Brochage latéral avec **40 broches GPIO** (20 de chaque côté)
- LED RGB **WS2812** intégrée (GPIO23)
- Boutons embarqués :
  - **BOOT**
  - **RESET**
  - **USER KEY** (GPIO24)
- Interface **SWD** pour débogage

La carte **RP2040-YD** est conçue pour venir se **connecter verticalement** sur le PCB principal appelé **BR-Motor** via deux rangées de connecteurs femelles. Une fois assemblée, l'ensemble forme la carte **Mastodonte**, regroupant la logique de contrôle (RP2040-YD) et l’électronique de puissance et d’interface (BR-Motor).

<p align="center">
  <img src="Image/BR-Motor-N11.png" alt="RP2040-YD plug sur BR-Motor" width="900"/>
</p>

### Interfaces et connectiques
- Connecteurs au format **JST B2B-XH**
- Connexions dédiées pour moteurs, charges pyrotechniques, capteurs et interfaces de communication
- Tensions disponibles : **5 V** et **3.3 V**

---

## Interfaces et E/S

### Commande moteur / Charge pyrotechnique
- 3 drivers **DRV8872DDA** (ponts en H)
  - Jusqu’à **3.6 A** sous **6.5–45 V**
  - Contrôle **PWM**
  - LED de direction pour visualisation sans charge
  - Détection d’erreurs via broche **nFAULT**

### Signaux d’entrée isolés
- **Optocoupleurs ACPL-214** pour isolation galvanique
- Buffers logiques **74HC14** (Schmitt trigger) pour traitement numérique
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

## Signaux sortants – Carte Mastodonte (RP2040-YD)

| Signal         | Type de signal     | Connecteur        | GPIO RP2040 | Description |
|----------------|--------------------|-------------------|-------------|-------------|
| `IN1_M1`       | PWM / Logique      | J6 (Moteur 1)     | GP14        | Commande H-Bridge moteur/charge 1 |
| `IN2_M1`       | PWM / Logique      | J6 (Moteur 1)     | GP15        | Commande H-Bridge moteur/charge 1 |
| `IN1_M2`       | PWM / Logique      | J7 (Moteur 2)     | GP16        | Commande H-Bridge moteur/charge 2 |
| `IN2_M2`       | PWM / Logique      | J7 (Moteur 2)     | GP17        | Commande H-Bridge moteur/charge 2 |
| `IN1_M3`       | PWM / Logique      | J8 (Moteur 3)     | GP18        | Commande H-Bridge moteur/charge 3 |
| `IN2_M3`       | PWM / Logique      | J8 (Moteur 3)     | GP19         | Commande H-Bridge moteur/charge 3 |
| `IN_BUZZ`      | Logique             | J5 (Buzzer)       | GP2         | Commande du buzzer (MOSFET BSS138) |
| `OUT_N1`       | GPIO                | J10               | GP26        | Sortie numérique non isolée |
| `OUT_N2`       | GPIO                | J10               | GP27        | Sortie numérique non isolée |
| `OUT_N3`       | GPIO                | J10               | GP10        | Sortie numérique non isolée |
| `OUT_N4`       | GPIO                | J11               | GP11        | Sortie numérique non isolée |
| `OUT_N5`       | GPIO                | J11               | GP12        | Sortie numérique non isolée |
| `OUT_N6`       | GPIO                | J11               | GP13        | Sortie numérique non isolée |
| `OUT_ISO_N3`   | GPIO isolée         | J13 (Opto)        | GP3         | Sortie isolée via optocoupleur |
| `OUT_ISO_N4`   | GPIO isolée         | J13 (Opto)        | GP4         | Sortie isolée via optocoupleur |
| `SDA`          | I²C (données)       | J12 (I²C)         | GP8         | Bus I²C données (avec pull-up) |
| `SCL`          | I²C (horloge)       | J12 (I²C)         | GP9         | Bus I²C horloge (avec pull-up) |
| `IN_ISO_N1`    | UART (émission)     | J9 (UART)         | GP0         | Émission isolée série vers modules externes |
| `OUT_ISO_N2`   | UART (réception)    | J9 (UART)         | GP1         | Réception isolée série depuis modules externes |
| `VCC_BAT`      | Alimentation        | J2 / J3 (OUT PWR) | —           | Tension batterie (après filtrage) exposée |
| `+5V`          | Alimentation régulée| J2 / J3 / I²C     | —           | Tension 5 V régulée disponible |
| `+3.3V`        | Alimentation logique| Tous connecteurs  | —           | Tension logique (niveau RP2040) |

> Tous les signaux sont sur le standart CMOS 3.3 V côté logique.

## RP2040-YD PINOUT

<p align="center">
  <img src="Image/rp2040-yd_pinout_zl.jpg" alt="RP2040-YD board" width="600"/>
</p>

---

## Caractéristiques mécaniques

- Dimensions : **100 × 40 mm**
- Épaisseur maximale : **18.13 mm**
- Trous de fixation : 4 × Ø3.2 mm
- Masse sans support : **37 g**
- Fichier **STEP 3D** disponible ici : [`Mechanical/BR-Motor.step`](Mechanical/BR-Motor.step)

<p align="center">
  <img src="Image/Mastodonte-N8.jpg" alt="Dimensions mécaniques" width="500"/>
</p>

---

## Ressources utiles

Composants utilisés sur la carte Mastodonte, avec leurs fiches techniques officielles :

| Composant | Rôle dans le système | Fiche technique |
|----------|----------------------|-----------------|
| **DRV8872** | Driver de moteur DC (pont en H jusqu’à 3.6 A) | [→ Datasheet TI](https://www.ti.com/lit/ds/symlink/drv8872.pdf) |
| **RP2040** | Microcontrôleur principal double cœur | [→ Datasheet Raspberry Pi](https://www.raspberrypi.com/documentation/microcontrollers/rp2040.html) |
| **ACPL-214** | Optocoupleur pour isolation galvanique des signaux d’entrée | [→ Datasheet Broadcom](https://www.broadcom.com/products/optocouplers/industrial-plastic/acpl-214) |
| **TLP292(TPL,E** | Optocoupleur pour isolation galvanique des signaux d’entrée | [→ Datasheet Toshiba](https://toshiba.semicon-storage.com/info/TLP292_datasheet_en_20191129.pdf?did=14425&prodName=TLP292) |
| **74HC14** | Buffer logique à trigger de Schmitt pour traitement des signaux numériques | [→ Datasheet TI](https://www.ti.com/lit/ds/symlink/sn74hc14.pdf?ts=1743611017766) |
| **ADuM1281** | Isolateur numérique double canal pour signaux UART | [→ Datasheet Analog Devices](https://www.analog.com/media/en/technical-documentation/data-sheets/ADuM1280_1281_1285_1286.pdf) |
| **LM340AT** | Régulateur de tension linéaire 5V | [→ Datasheet TI](https://www.ti.com/lit/ds/symlink/lm340.pdf) |
| **1812L260/16MR** | Fusible réarmable pour limitation de courant | [→ Datasheet Littelfuse](https://www.littelfuse.com/assetdocs/resettable-ptcs-1812l-datasheet?assetguid=ca5c80cb-504e-4a8a-8e74-0107520a1717) |
| **SQD50P04-13L** | MOSFET P pour protection contre l’inversion de polarité | [→ Datasheet Vishay](https://www.vishay.com/docs/65157/sqd50p04-13l.pdf) |
| **BSS138** | Transistor N-MOS pour commutation de charges faibles (buzzer, etc.) | [→ Datasheet OnSemi](https://www.onsemi.com/pdf/datasheet/bss138-d.pdf) |

> Sur la production 02/03/2025, les optocoupleurs utilisés sont les TLP292(TPL,E).

---

## Licence CC-BY-NC-SA

<p align="center">
  <img src="Image/Cc-by-nc-sa_icon.svg.png" alt="CC-BY-NC-SA" width="500"/>
</p>

