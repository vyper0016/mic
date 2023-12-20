# Mic-1

## Registers

![mic1](img/2023-12-20-15-59-37.png)

Registers sind im wesentlichen 32-Bit/4-byte Speicher (Ausnahme: `MBR` nur 1 Byte/8 Bit breit)

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
- `OPC`: Operation Code
- `H`: Help Register
  
### Descriptionen

#### `MDR`: Memory Data Register

#### `MAR`: Memory Address Register

#### `PC`: Program Counter

#### `MBR`: Memory Buffer Register

#### `SP`: Stack Pointer

Es steht immer die Adresse des obersten Elements auf dem Stack im Register `SP`.

#### `LV`: Local Variable

#### `CPP`: Constant Pool Pointer

#### `TOS`: Top of Stack

Es steht immer der Wert des obersten Elements auf dem Stack im Register `TOS`.

#### `OPC`: Operation Code

#### `H`: Help Register
