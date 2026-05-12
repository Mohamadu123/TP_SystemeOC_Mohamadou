# Tuto_Making_Qsys_Comp

Projet réalisé dans le cadre du tutoriel **"Making Qsys Components"** d'Altera (Quartus II 13.0).  
L'objectif est de créer un composant Qsys personnalisé (un registre 16 bits) et de l'intégrer dans un système embarqué Nios II sur une carte **DE1**.

---

## Point de départ

Ce projet est basé sur l'exemple fourni par Altera University Program :

```
C:\altera\13.0sp1\University_Program\NiosII_Computer_Systems\DE1\DE1_Basic_Computer\
```

Le projet de base `DE1_Basic_Computer` fournit un système Nios II complet pour la carte DE1, incluant le processeur, les mémoires SRAM/SDRAM, l'UART, les LEDs, les switches et les boutons.

### Modifications apportées au projet de base

- **Désactivation** du composant HEX3_HEX0 natif du système Qsys (ports mis en commentaire dans `DE1_Basic_Computer.vhd`)
- **Création** d'un composant Qsys custom `reg16_avalon_interface` et ajout dans `nios_system.qsys`
- **Ajout** du signal conduit `to_hex_readdata` exporté depuis le système Qsys vers le top-level
- **Ajout** de 4 instances de `hex7seg` dans le top-level pour décoder et afficher la valeur du registre sur HEX0 à HEX3
- **Correction** de `hex7seg.vhd` : direction du port `display` changée de `(0 TO 6)` à `(6 DOWNTO 0)` pour correspondre aux pins de la DE1, et mise à jour de la table de vérité en conséquence

---

## Architecture du projet

```
Tuto_Making_Qsys_Comp/
│
├── ip_modules/                        # Composants VHDL custom
│   ├── reg16.vhd                      # Registre 16 bits
│   ├── reg16_avalon_interface.vhd     # Interface Avalon MM du registre
│   └── hex7seg.vhd                    # Décodeur 7 segments (corrigé)
│
├── nios_system/                       # Système généré par Qsys
├── software/                          # Code logiciel Nios II
│
├── DE1_Basic_Computer.vhd             # Top-level VHDL (modifié)
├── DE1_Basic_Computer.qpf             # Fichier projet Quartus
├── DE1_Basic_Computer.qsf             # Assignments et pin assignments
├── nios_system.qsys                   # Description du système Qsys (modifié)
├── reg16_avalon_interface_hw.tcl      # Fichier TCL du composant Qsys
├── sdram_pll.vhd                      # PLL pour la SDRAM (inchangé)
└── .gitignore
```

---

## Composants créés

### `reg16.vhd`
Registre 16 bits synchrone avec :
- Reset actif bas (`resetn`)
- Écriture octet par octet contrôlée par `byteenable[1:0]`

### `reg16_avalon_interface.vhd`
Interface Avalon Memory-Mapped (esclave) qui entoure `reg16` :

| Signal | Direction | Description |
|---|---|---|
| `clock` | Entrée | Horloge système |
| `resetn` | Entrée | Reset actif bas |
| `writedata[15:0]` | Entrée | Données à écrire |
| `readdata[15:0]` | Sortie | Données lues |
| `write` | Entrée | Signal d'écriture |
| `read` | Entrée | Signal de lecture |
| `chipselect` | Entrée | Sélection du composant |
| `byteenable[1:0]` | Entrée | Sélection des octets |
| `Q_export[15:0]` | Sortie | Conduit vers l'extérieur |

### `hex7seg.vhd`
Décodeur hexadécimal vers afficheur 7 segments (actif bas).  
Instancié 4 fois dans le top-level pour afficher les 4 nibbles du registre.

> **Note :** Par rapport à la version du tutoriel Altera (`display : OUT STD_LOGIC_VECTOR(0 TO 6)`),
> la direction a été corrigée en `(6 DOWNTO 0)` pour correspondre aux pins `HEX[6:0]` de la DE1,
> et la table de vérité a été mise à jour en conséquence.

---

## Système Qsys (`nios_system.qsys`)

Le système embarqué contient (base DE1_Basic_Computer + ajout) :
- **Nios II/e** — processeur soft-core
- **On-Chip Memory** — mémoire programme/données
- **SDRAM Controller** — mémoire externe
- **SRAM Controller** — mémoire externe
- **UART** — port série
- **reg16_avalon_interface** *(ajouté)* — registre 16 bits custom
  - Adresse de base : `0x00000000`
  - Conduit exporté sous le nom `to_hex`

---

## Top-level (`DE1_Basic_Computer.vhd`)

Modification clé par rapport au fichier original : le signal `to_hex_readdata` récupéré depuis Qsys est décodé par 4 instances de `hex7seg` et envoyé vers les afficheurs.

```
to_HEX[3:0]   → hex7seg → HEX0
to_HEX[7:4]   → hex7seg → HEX1
to_HEX[11:8]  → hex7seg → HEX2
to_HEX[15:12] → hex7seg → HEX3
```

---

## Comment tester

1. Ouvrir le projet dans **Quartus II 13.0**
2. Compiler (`Processing > Start Compilation`)
3. Programmer la carte DE1 avec le fichier `.sof` généré
4. Ouvrir l'**Altera Monitor Program**, créer un projet avec `nios_system.qsys`
5. Dans l'onglet **Memory**, aller à l'adresse `0x00000000`
6. Modifier la valeur du registre → observer les afficheurs 7 segments

---

## Matériel requis

- Carte **Altera DE1**
- **Quartus II 13.0sp1** (ou version compatible)
- **Altera Monitor Program**

---

## Références

- Tutoriel Altera University Program — *Making Qsys Components for Quartus II 13.0* (Mai 2013)
- Projet de base : `C:\altera\13.0sp1\University_Program\NiosII_Computer_Systems\DE1\DE1_Basic_Computer\`
