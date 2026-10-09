<div align="center">

# 🔐 RC5 Cipher — AVR ATmega328P Implementation

**A complete implementation of the RC5 symmetric block cipher in three variants:**
**C++, AVR Assembly, and AVR Assembly with LCD Simulation.**

[![Language](https://img.shields.io/badge/Language-C%2B%2B%20%7C%20AVR%20Assembly-blue?style=for-the-badge)]()
[![Platform](https://img.shields.io/badge/Platform-ATmega328P-orange?style=for-the-badge)]()
[![Cipher](https://img.shields.io/badge/Cipher-RC5--16%2F8%2F12-green?style=for-the-badge)]()
[![IDE](https://img.shields.io/badge/IDE-Atmel%20Studio%20%7C%20Visual%20Studio-purple?style=for-the-badge)]()

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [RC5 Algorithm](#-rc5-algorithm)
- [Project Structure](#-project-structure)
- [Versions](#-versions)
  - [Version 1 – C++](#version-1--c)
  - [Version 2 – Assembly](#version-2--assembly)
  - [Version 3 – Assembly with LCD Simulation](#version-3--assembly-with-lcd-simulation)
- [RC5 Parameters](#-rc5-parameters)
- [Key Expansion](#-key-expansion)
- [Encryption & Decryption](#-encryption--decryption)
- [LCD Display](#-lcd-display-simulation-only)
- [Tools & Requirements](#-tools--requirements)
- [How to Build & Run](#-how-to-build--run)
- [Register Map](#-register-map-assembly)
- [Memory Layout](#-memory-layout-assembly)
- [License](#-license)

---

## 🧾 Overview

This project implements the **RC5** symmetric-key block cipher, originally designed by Ronald Rivest in 1994.
RC5 is a fast, simple, and highly parameterizable cipher — well-suited for embedded and constrained environments.

This implementation targets the **AVR ATmega328P** microcontroller (the chip inside the Arduino Uno), running at **16 MHz**.
It is written **entirely from scratch** — no libraries, no HAL abstractions.

All three versions are **functionally equivalent**; they differ only in implementation language and output method.

> **RC5 variant used:** `RC5-16/8/12`
> *(16-bit word size · 8 rounds · 12-byte / 96-bit secret key)*

---

## 🔒 RC5 Algorithm

RC5 is a **symmetric block cipher** with fully parameterizable word size, round count, and key length.

| Property | Value |
|---|---|
| Block size | 2 × w bits → **32 bits** (two 16-bit words A and B) |
| Word size (w) | **16 bits** |
| Number of rounds (r) | **8** |
| Key size (b) | **12 bytes = 96 bits** |
| Subkey table size (t) | 2(r+1) = **18 words** |

### Core Primitive Operations

RC5 is built on exactly **three** operations:

| Operation | Symbol | Description |
|-----------|--------|-------------|
| Addition | `+` | Modulo 2^w (16-bit wrapping) |
| XOR | `⊕` | Bitwise exclusive-or |
| Data-dependent rotation | `<<<` / `>>>` | Rotate left/right by a **variable** amount |

The data-dependent rotation is what gives RC5 its security — the rotation amount depends on the data being processed, not a fixed constant.

### Magic Constants

Derived from the mathematical constants *e* (Euler's number) and *φ* (golden ratio):

```
P16 = 0xB7E1   (Odd((e  − 2) × 2^16))
Q16 = 0x9E37   (Odd((φ − 1) × 2^16))
```

---

## 📁 Project Structure

```
RC5/
│
├── rc5-cpp/                                   # Version 1 – C++ Reference
│   ├── src/
│   │   └── rc5.cpp                            # Full RC5 in C++ with debug output
│   └── project-files/
│       ├── rc5.sln                            # Visual Studio solution
│       ├── rc5.vcxproj                        # VS project file
│       └── rc5.vcxproj.filters
│
├── rc5-assembly/                              # Version 2 – Pure AVR Assembly
│   ├── src/
│   │   └── main.asm                           # RC5 in AVR Assembly (no display)
│   ├── output/
│   │   └── rc5-assembly.hex                   # ✅ Flash directly to ATmega328P / Arduino
│   └── project-files/
│       ├── rc5-assembly.atsln                 # Atmel Studio solution
│       └── rc5-assembly.asmproj
│
├── rc5-assembly-simulation/                   # Version 3 – Assembly + LCD (Proteus)
│   ├── src/
│   │   └── main.asm                           # RC5 + LCD output macros
│   ├── output/
│   │   └── rc5-assembly-simulation.hex        # ✅ Load into Proteus simulation
│   └── project-files/
│       ├── rc5-assembly-simulation.atsln      # Atmel Studio solution
│       └── rc5-assembly-simulation.asmproj
│
├── rc5-simulation.pdsprj                      # ✅ Proteus ISIS schematic & simulation
├── .gitignore
└── README.md
```

---

## 📦 Versions

### Version 1 — C++

A clean, heavily-commented **C++ reference implementation** — ideal for understanding the algorithm step by step.

**Source:** [`rc5-cpp/src/rc5.cpp`](rc5-cpp/src/rc5.cpp)

**Features:**
- Full key expansion, encryption, and decryption
- Verbose `cout` debug output for **every** intermediate value (L array, S table, A/B at each round)
- Automatic verification: confirms `decrypt(encrypt(plaintext)) == plaintext`
- Uses `uint16_t` throughout to precisely match the 16-bit word size
- A commented-out clean version is included alongside the verbose one

**Example run:**
```
Key:        01 23 45 67 89 AB CD EF FE DC BA 98  (96-bit)
Plaintext:  A = 0x1234,  B = 0x5678

[key_expansion output...]
[encryption output...]

Encrypted:  A = 0xXXXX,  B = 0xXXXX
Decrypted:  A = 0x1234,  B = 0x5678
Result:     Decryption successful - matches original plaintext!
```

---

### Version 2 — Assembly

A **bare-metal AVR Assembly** implementation. No LCD, no UART — results sit in SRAM and are inspected using the Atmel Studio simulator/debugger.

**Source:** [`rc5-assembly/src/main.asm`](rc5-assembly/src/main.asm)
**HEX file:** [`rc5-assembly/output/rc5-assembly.hex`](rc5-assembly/output/rc5-assembly.hex)

**Features:**
- 100% pure AVR Assembly — zero C runtime overhead
- Fully commented register usage throughout every routine
- Implements all three RC5 phases: `key_expansion`, `encrypt`, `decrypt`
- Uses ATmega328P DSEG (data segment) for the S table, L array, and plaintext words
- Results inspectable via Atmel Studio's Memory View or debugger watch

**Plaintext:**
```
A = 0x1234,  B = 0x5678
```

---

### Version 3 — Assembly with LCD Simulation

An **enhanced Assembly version** designed for **Proteus ISIS simulation** with a connected 16×2 **HD44780 LCD** in 4-bit mode. After each RC5 phase, the result bytes are written to the LCD display.

**Source:** [`rc5-assembly-simulation/src/main.asm`](rc5-assembly-simulation/src/main.asm)
**HEX file:** [`rc5-assembly-simulation/output/rc5-assembly-simulation.hex`](rc5-assembly-simulation/output/rc5-assembly-simulation.hex)
**Proteus schematic:** [`rc5-simulation.pdsprj`](rc5-simulation.pdsprj)

**Additional features over Version 2:**

| Feature | Description |
|---------|-------------|
| `LCD` macro | Initializes the HD44780 LCD (4-bit mode) and writes the 4 result bytes |
| `CLEAN` macro | Clears all 32 registers (R0–R31) between phases to prevent register pollution |
| `UPDATE_A_B` macro | Re-reads A and B words from SRAM into display registers (R4–R7) after each phase |
| Delay routines | `delay_short`, `delay_us`, `delay_ms`, `delay_seconds` — calibrated for 16 MHz |

**Plaintext used:**
```
A = 0x5941  ("YA")
B = 0x5337  ("S7")
```

**LCD Wiring:**

| LCD Signal | ATmega328P Pin | Description |
|-----------|----------------|-------------|
| D4 – D7 | PORTD bits 4–7 | 4-bit data bus |
| RS | PB1 | Register Select (0=command, 1=data) |
| EN | PB0 | Enable pulse |
| RW | GND | Always write mode |

> The LCD shows **4 bytes** of result after each of the 3 phases (key expansion → encrypt → decrypt), with a **3-second hold** between each byte.

---

## ⚙️ RC5 Parameters

| Parameter | Symbol | Value | Notes |
|-----------|--------|-------|-------|
| Word size | w | **16 bits** | Half-word on AVR (two 8-bit registers per word) |
| Rounds | r | **8** | 8 full encrypt/decrypt rounds |
| Key length | b | **12 bytes** | 96-bit secret key |
| Key words | c | **6** | b / (w/8) = 12 / 2 |
| Subkey table size | t | **18** | 2(r+1) = 2×9 |
| Key mix iterations | n | **54** | 3 × max(t, c) = 3 × 18 |
| Magic constant P | P16 | **0xB7E1** | Based on Euler's number e |
| Magic constant Q | Q16 | **0x9E37** | Based on golden ratio φ |

**Secret Key (96-bit / 12 bytes):**
```
0x01  0x23  0x45  0x67  0x89  0xAB  0xCD  0xEF  0xFE  0xDC  0xBA  0x98
```

---

## 🔑 Key Expansion

The key schedule transforms the 12-byte user key into an **18-word expanded subkey table** `S[0..17]`.

**Step 1 — Copy key bytes into word array L:**
```
for i = 0 to b-1:
    L[i/2] = (L[i/2] << 8) | K[i]
```

**Step 2 — Initialize S with the magic constants:**
```
S[0] = P16
for i = 1 to t-1:
    S[i] = S[i-1] + Q16       (mod 2^16)
```

**Step 3 — Mix L into S over n = 54 iterations:**
```
A = B = i = j = 0
for k = 0 to n-1:
    A = S[i] = (S[i] + A + B) <<< 3
    B = L[j] = (L[j] + A + B) <<< (A + B)
    i = (i + 1) mod t
    j = (j + 1) mod c
```

---

## 🔐 Encryption & Decryption

### Encryption Algorithm
```
A = A + S[0]
B = B + S[1]
for i = 1 to r:
    A = ((A XOR B) <<< B) + S[2i]
    B = ((B XOR A) <<< A) + S[2i+1]
```

### Decryption Algorithm
```
for i = r downto 1:
    B = ((B - S[2i+1]) >>> A) XOR A
    A = ((A - S[2i])   >>> B) XOR B
B = B - S[1]
A = A - S[0]
```

> `<<<` = rotate left, `>>>` = rotate right.
> All arithmetic is modulo 2^16. All rotation amounts are modulo 16 (word size).

---

## 🖥️ LCD Display (Simulation Only)

The simulation version writes results to a **16×2 HD44780 LCD** after each phase.

**Display sequence** (runs 3 times — after key expansion, after encrypt, after decrypt):

| Step | Register | Content |
|------|----------|---------|
| 1 | R5 | A high byte |
| 2 | R4 | A low byte |
| 3 | R7 | B high byte |
| 4 | R6 | B low byte |
| 5 | — | Hold 3 seconds |

**Timing at 16 MHz:**

| Routine | Approximate Duration |
|---------|---------------------|
| `delay_short` | ~2 CPU cycles (2× NOP) |
| `delay_us` | ~90 × delay_short |
| `delay_ms` | ~40 × delay_us |
| `delay_seconds` | ~255 × 255 × 20 iterations ≈ 250 ms/call |

---

## 🛠️ Tools & Requirements

### Version 1 — C++

| Tool | Requirement |
|------|------------|
| Visual Studio | 2019 or 2022 |
| C++ Standard | C++11 or later |
| Platform | Windows x64 |

### Version 2 & 3 — Assembly

| Tool | Requirement |
|------|------------|
| Atmel Studio / Microchip Studio | 7.0+ |
| Assembler | AVRASM2 (built into Atmel Studio) |
| Target MCU | ATmega328P |
| Clock Speed | 16 MHz |

### Version 3 — Simulation (additional)

| Tool | Purpose |
|------|---------|
| Proteus ISIS | Circuit simulation environment |
| HD44780 LCD module | 16×2 character display in schematic |

---

## 🚀 How to Build & Run

### Version 1 — C++ in Visual Studio

1. Open `rc5-cpp/project-files/rc5.sln` in Visual Studio
2. Set build configuration to **Debug** / **x64**
3. Press **F5** to build and run
4. The console prints all intermediate values and the final encrypt/decrypt result

---

### Version 2 — Assembly in Atmel Studio (no display)

1. Open `rc5-assembly/project-files/rc5-assembly.atsln` in Atmel Studio
2. Press **F7** to build
3. Go to **Debug → Start Without Debugging** to run in the simulator
4. Use **Debug → Windows → Memory** to inspect SRAM:

| Address | Content |
|---------|---------|
| `0x0100` | S table — 36 bytes (18 × 2-byte words) |
| `0x0124` | L array — 12 bytes (6 × 2-byte words) |
| `0x0130` | Word A — 2 bytes (low, high) |
| `0x0132` | Word B — 2 bytes (low, high) |

> **To flash to hardware:** Program `rc5-assembly/output/rc5-assembly.hex` directly onto an ATmega328P or Arduino Uno using Atmel Studio, AVRDUDE, or the Arduino IDE bootloader.

---

### Version 3 — Assembly + LCD in Proteus

1. Open `rc5-assembly-simulation/project-files/rc5-assembly-simulation.atsln` in Atmel Studio
2. Press **F7** to build, or use the pre-built HEX: `rc5-assembly-simulation/output/rc5-assembly-simulation.hex`
3. Open `rc5-simulation.pdsprj` in **Proteus ISIS**
4. In the ATmega328P component properties, set the **Program File** to the HEX above
5. Press **Play** to start the simulation
6. Observe the LCD — it displays the 4 result bytes after each of the 3 phases

---

## 📌 Register Map (Assembly)

### Shared Routines (key_expansion, encrypt, decrypt)

| Register | Role |
|----------|------|
| R14 | SREG scratch — temporary carry flag holder |
| R15 | Iteration counter for key mix loop (n = 54) |
| R16–R17 | Word A (low byte, high byte) |
| R18–R19 | Word B (low byte, high byte) |
| R20–R21 | Temporary — S[i] or B accumulator |
| R22–R23 | S[i] working copy (low, high) |
| R24–R25 | L[j] working copy (low, high) |
| R26 (XL) | Rotation bit counter |
| R28–R29 (YL/YH) | Pointer to L array in SRAM |
| R30–R31 (ZL/ZH) | General data pointer (S, A, B, flash) |

### Simulation-Only Registers (Version 3)

| Register | Role |
|----------|------|
| R2–R3 | Saved ZL/ZH inside `UPDATE_A_B` macro |
| R4–R5 | Mirror of word A (low, high) for LCD |
| R6–R7 | Mirror of word B (low, high) for LCD |
| R25 | LCD command / data byte |
| R28–R29 | LCD timing loop counters |

---

## 🗺️ Memory Layout (Assembly)

```
ATmega328P SRAM — Data Segment (DSEG):

Address   Contents
───────── ──────────────────────────────────────────────
0x0100    S[0]  low byte  ┐
0x0101    S[0]  high byte │ Expanded subkey table
  ...                     │ 18 words × 2 bytes = 36 bytes
0x0122    S[17] low byte  │
0x0123    S[17] high byte ┘

0x0124    L[0]  low byte  ┐
0x0125    L[0]  high byte │ Secret key word array
  ...                     │ 6 words × 2 bytes = 12 bytes
0x012E    L[5]  low byte  │
0x012F    L[5]  high byte ┘

0x0130    A     low byte  ┐ Plaintext / ciphertext word A
0x0131    A     high byte ┘

0x0132    B     low byte  ┐ Plaintext / ciphertext word B
0x0133    B     high byte ┘
```

---

## 📄 License

This project is open-source and free to use for **educational purposes**.
Attribution is appreciated if you build on or share this work.

---

<div align="center">

*Built for embedded systems education*

**ATmega328P · RC5-16/8/12 · AVR Assembly · Proteus ISIS**

</div>
