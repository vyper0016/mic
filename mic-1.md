# Mic-1

- [Mic-1](#mic-1)
  - [Registers](#registers)
    - [Namen](#namen)
    - [Descriptionen](#descriptionen)
      - [`MDR`: Memory Data Register](#mdr-memory-data-register)
      - [`MAR`: Memory Address Register](#mar-memory-address-register)
      - [`PC`: Program Counter](#pc-program-counter)
      - [`MBR`: Memory Buffer Register](#mbr-memory-buffer-register)
      - [`MBRU` *(pseudo)*: Memory Buffer Register Unsigned](#mbru-pseudo-memory-buffer-register-unsigned)
      - [`SP`: Stack Pointer](#sp-stack-pointer)
      - [`LV`: Local Variable](#lv-local-variable)
      - [`CPP`: Constant Pool Pointer](#cpp-constant-pool-pointer)
      - [`TOS`: Top of Stack](#tos-top-of-stack)
      - [`OPC`: Old Program Counter](#opc-old-program-counter)
      - [`H`: Help Register](#h-help-register)
  - [ALU](#alu)
  - [Hauptspeicher](#hauptspeicher)
  - [Zyklus](#zyklus)
    - [Speichern](#speichern)
    - [Laden](#laden)
    - [Bytecode Zugriff](#bytecode-zugriff)
  - [Mikroinstruktionen](#mikroinstruktionen)
    - [Mikroinstruktionen Format](#mikroinstruktionen-format)
      - [Addr (9 Bit)](#addr-9-bit)
      - [JAM (3 Bit)](#jam-3-bit)
      - [ALU (8 Bit)](#alu-8-bit)
      - [C-Bus (9 Bit)](#c-bus-9-bit)
      - [Mem (3 Bit)](#mem-3-bit)
      - [B (4 Bit)](#b-4-bit)
  - [Kontrollpfad](#kontrollpfad)
  - [Ablauf eines Zyklus](#ablauf-eines-zyklus)
    - [Δw](#δw)
    - [Δx](#δx)
    - [Δy](#δy)
    - [Δz](#δz)
    - [Steigende Flanke](#steigende-flanke)

## Registers

![mic1](img/2023-12-20-15-59-37.png)

Registers sind im wesentlichen 32-Bit/4-byte Speicher (Ausnahme: `MBR` nur 1 Byte/8 Bit breit).

### Namen

Registers mit Specherzugriff:

- `MDR`: Memory Data Register
- `MAR`: Memory Address Register
- `PC`: Program Counter
- `MBR`: Memory Buffer Register

----

- `SP`: Stack Pointer
- `LV`: Local Variable
- `CPP`: Constant Pool Pointer
- `TOS`: Top of Stack
- `OPC`: Old Program Counter
- `H`: Help Register
  
### Descriptionen

#### `MDR`: Memory Data Register

Beinhaltet das Wert des Speicherwortes, das gelesen oder geschrieben werden soll.

#### `MAR`: Memory Address Register

Beinhaltet die Adresse des Speicherwortes, das gelesen oder geschrieben werden soll.

#### `PC`: Program Counter

Beinhaltet die Adresse des nächsten Befehls (in der Method Area). Wird nach jedem Befehl inkrementiert.

#### `MBR`: Memory Buffer Register

Beinhaltet das Wert des Speicherwortes, die in Adresse `PC` steht. Also was als nächstes ausgeführt werden soll. (Vorzeichenbehaftet)

#### `MBRU` *(pseudo)*: Memory Buffer Register Unsigned

Vorzeichenlose Variante von `MBR`. Wird für Vorzeichenlose Operanden verwendet (z.B. `varnum`).

#### `SP`: Stack Pointer

Beinhaltet die Adresse des obersten Elements auf dem Stack.

#### `LV`: Local Variable

Beinhaltet die Adresse der unteren Rand des aktuellen Stackframes (`OBJREF`).

#### `CPP`: Constant Pool Pointer

Adresse des ersten Elements im Constant Pool. Es ändert sich nicht während der Laufzeit.

#### `TOS`: Top of Stack

Wert des obersten Wort auf dem Stack.

#### `OPC`: Old Program Counter

Wird benutzt um verschiedene Werte zwischenzuspeichern. (z.B. `PC` bei `GOTO` / `TOS` bei `if_icmpeq`)

#### `H`: Help Register

Das verwenden wir, wenn wir zwei Operanten brauchen. Was in H liegt, liegt auch an A an (ALU Eingang).

## ALU

![ALU](img/2023-12-20-16-26-27.png)

Wir haben in der `ALU` 32 von diese Einheiten in Verkettung.

- `N`: Negative Flag<br>(Das Ergebnis der letzten Operation eine 1 im MSB hat)
- `Z`: Zero Flag<br>(Das Ergebnis der letzten Operation eine 0 ist)

## Hauptspeicher

![Hauptspeicher](img/2023-12-20-16-35-38.jpg)

Wir haben im Hauptspeicher drei verschiedene Speicherbereiche:

- `Constant Pool`
- `Stack Frame`
- `Method Area`

Wir interagieren (rd, write, fetch) mit dem Hauptspeicher über die `MAR`, `MDR` (für rd und wr) und `PC` (für fetch) [Registers](#namen).

## Zyklus

genauer [weiter unten](#ablauf-eines-zyklus)

### Speichern

- Lade in `MAR` die Addresse des Wortes, das wir speichern wollen. (1. Zyklus)
- Lade in `MDR` das Wert des Wortes, das wir speichern wollen. (2. Zyklus)
- Speicher signalisieren (Ende des 2. Zyklus)
- Daten sind am Ende des 3. Zyklus im Speicher. `MDR` und `MAR` dürfen während des 3. Zyklus wieder verwendet werden.

### Laden

- Lade in `MAR` die Addresse des Wortes, das wir laden wollen. Signalisiere, dass geladen werden soll (1. Zyklus)
- Ergebnis ist am Ende des 2. Zyklus in `MDR`. Ursprüngliche Inhalt des `MDR` darf im 2. Zyklus noch verwendet werden aber nicht vom C-Bus geschrieben werden (sonst Kollision).
- Ab dem 3. Zyklus darf der neuen Inhalt des `MDR` verwendet werden.

### Bytecode Zugriff

Wir nutzen hier die `PC` und `MBR` [Registers](#namen), um den Bytecode byteweise zu lesen, mit einem `Fetch` Signal.

## Mikroinstruktionen

![Mikroinstruktion](img/2023-12-20-18-38-22.png)

Anhand 36-Bit Mikroinstruktionen wird die [ALU](#alu) und die [Registers](#registers) gesteuert.

### Mikroinstruktionen Format

#### Addr (9 Bit)

Beinhaltet `NEXT_ADDRESS` (Adresse der nächsten Mikroinstruktion), je nachdem was in [`JAM`](#jam-3-bit) steht.

#### JAM (3 Bit)

- `JMPC`: Jump on [``PC``](#pc-program-counter )<br>
Bytecode byte wird in `MPC` geladen. z.B. `0x60` für `IADD`.
- `JAMN`: Jump if Negative<br>
  ``MPC[8] = NEXT_ADDRESS[8] OR N``
- `JAMZ`: Jump if Zero<br>
`MPC[8] = NEXT_ADDRESS[8] OR Z`

Es gilt insgesamt: `MPC[8] = (JAMZ AND Z) OR (JAMN AND N) OR NEXT_ADDRESS[8]`

#### ALU (8 Bit)

- `SLL8`: Shift Left Logical 8
- `SRA1`: Shift Right Arithmetic 1
- `F0` - `INC`: [ALU](#alu) Input

#### C-Bus (9 Bit)

Welche Register vom C-Bus geschrieben werden sollen, je nachdem welche Bits auf 1 gesetzt sind.

#### Mem (3 Bit)



#### B (4 Bit)

Was auf den B-Bus gelegt werden soll. Dafür ist folgende Kodierung vorgesehen:

![Bkodierung](img/2023-12-20-19-23-12.png)

## Kontrollpfad

![Kontrollpfad](img/2023-12-20-19-03-28.png)

## Ablauf eines Zyklus

![Zyklus](img/2023-12-20-16-51-23.png)

### &Delta;w

Wir nehmen die 9-Bit Addresse die in `MPC` steht und laden die Mikroinstruktion aus `control store`, die in dieser Adresse steht, in `MIR`.

### &Delta;x

- Wert von `B` in `MIR` wird dekodiert und auf den B-Bus gelegt.
- Es wird das Wert vom jeweiligen Register am [``ALU``](#alu) Eingang ``B`` gelegt.
- Wert von `H` wird am [``ALU``](#alu) Eingang ``A`` gelegt.

### &Delta;y

- `ALU` rechnet gemäß [Mikroinstruktion](#alu-8-bit) und leitet das Ergebnis an den Shifter weiter.
- `Shifter` modifiziert das Ergebnis.
  
### &Delta;z

- Ergebnis vom Shifter auf den ``C-Bus`` stabilisiert sich.

### Steigende Flanke

- Register werden vom ``C-Bus`` neugeladen.
