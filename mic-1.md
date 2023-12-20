# Mic-1

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

#### `MAR`: Memory Address Register

#### `PC`: Program Counter

#### `MBR`: Memory Buffer Register

#### `SP`: Stack Pointer

Adresse des obersten Elements auf dem Stack.

#### `LV`: Local Variable

Unterste Rand des aktuellen Stackframes (`OBJREF`).

#### `CPP`: Constant Pool Pointer

Adresse des ersten Elements im Constant Pool. Es ändert sich nicht während der Laufzeit.

#### `TOS`: Top of Stack

Wert des obersten Wort auf dem Stack.

#### `OPC`: Old Program Counter

#### `H`: Help Register

Das verwenden wir, wenn wir zwei Operanten brauchen. Was in H liegt, liegt auch an A an (ALU Eingang).

## ALU

![ALU](img/2023-12-20-16-26-27.png)

Wir haben in der `ALU` 32 von diese Einheiten in Verkettung.

## Hauptspeicher

![Hauptspeicher](img/2023-12-20-16-35-38.jpg)

Wir haben im Hauptspeicher drei verschiedene Speicherbereiche:

- `Constant Pool`
- `Stack Frame`
- `Method Area`

Wir interagieren mit dem Hauptspeicher über die `MAR` und `MDR` [Registers](###Namen).
