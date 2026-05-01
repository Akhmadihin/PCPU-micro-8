# PCPU-micro 8

<div align="center">

![Version](https://img.shields.io/badge/version-1.0-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Bits](https://img.shields.io/badge/bit-8-red)
![Status](https://img.shields.io/badge/status-stable-brightgreen)

**An 8-bit CPU made from discrete components, following the PCPU philosophy**

[Architecture](#-architecture) •
[Specifications](#-specifications) •
[Building](#-building) •
[Programming](#-programming) •
[Applications](#-applications)

</div>

---

## 📜 Description

**PCPU-micro 8** is an 8-bit processor built following the **PCPU** philosophy: minimalistic CPUs made from discrete components. It is a direct evolution of the 1-bit `PCPU-NANO1 T4` and the 4-bit `PCPU-MICRO-4`, scaled up to 8 bits.

> **Acknowledgement:** This project would not exist without the pioneering work of **ppc-8051**, the creator of the original `PCPU-NANO1` architecture. While the original repository has been removed, his core idea of a 1-bit CPU using only 5 transistors is the foundation upon which this entire family of processors is built. `PCPU-micro 8` is an independent, optimized implementation, compatible with the original's instruction set.

This CPU is intended for:
- **Education:** Learning CPU architecture hands-on with real hardware.
- **Embedded systems:** Controlling peripherals with minimal latency.
- **Retrocomputing:** Creating a DIY computer with 1970s/80s aesthetics.
- **Experimentation:** Testing new architectural concepts.

---

## 🖥️ Architecture

### Bit-Slice Structure

`PCPU-micro 8` is built as an **8-bit slice**: eight independent, identical 1-bit `PCPU-NANO1 T4` cores running in parallel.

```
┌─────────────────────────────────────────────────────────────────┐
│                     PCPU-micro 8                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   INS ──┬──►┌─────────┐                                         │
│         │   │ Core 0  │──► OUT0                                 │
│   DATA0 ┼──►│ (bit 0) │                                         │
│   IN0   └──►└─────────┘                                         │
│                                                                 │
│   DATA1 ──►┌─────────┐                                          │
│   IN1   ──►│ Core 1  │──► OUT1                                  │
│            │ (bit 1) │                                          │
│            └─────────┘                                          │
│              ...                                                │
│   DATA7 ──►┌─────────┐                                          │
│   IN7   ──►│ Core 7  │──► OUT7                                  │
│            │ (bit 7) │                                          │
│            └─────────┘                                          │
│                                                                 │
│   JMP0 ──┐                                                      │
│   JMP1 ──┼──► Resistor ──► JMP_total ──► LOAD (to PC)           │
│   ...    │                                                      │
│   JMP7 ──┘                                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Instruction Set

| INS | Mnemonic | Description | JMP | OUT |
|-----|----------|-------------|-----|-----|
| 0 | **NOP** | No Operation | 0 | NAND(DATA, IN) |
| 1 | **NADJZ** | NAND and Jump if Zero | DATA AND IN | NAND(DATA, IN) |

While minimal, this set is functionally complete (NAND is a universal logic gate).

### Scalability

| Processor | Cores | Transistors | OUT Bus |
|-----------|-------|-------------|---------|
| PCPU-NANO1 T4 | 1 | 4 | 1-bit |
| PCPU-MICRO-4 | 4 | 16 | 4-bit |
| **PCPU-micro 8** | **8** | **32** | **8-bit** |
| PCPU-MINI-13 | 13 | 52 | 13-bit |
| PCPU-MINI-16 | 16 | 64 | 16-bit |

---

## 📊 Specifications

| Parameter | Value |
|-----------|-------|
| **Data Width** | 8-bit |
| **Architecture** | Bit-slice (8 × 1-bit cores) |
| **Number of Cores** | 8 (PCPU-NANO1 T4 × 8) |
| **Transistors** | 32 + JMP logic |
| **Instructions** | 2 (NOP, NADJZ) |
| **Inputs** | INS, DATA[0..7], IN[0..7] |
| **Outputs** | OUT[0..7], JMP_total |
| **Clock Frequency** | 1-50 kHz (breadboard), up to 10 MHz (PCB) |
| **Power Supply** | 5V (4.5-5.5V) |
| **Current Draw** | ~40-100 mA |

---

## 🔧 Building

### Core Components (8-bit CPU only)

| Component | Quantity | Note |
|-----------|----------|------|
| **PCPU-NANO1 T4** (core) | 8 | 4 transistors each |
| **Resistor 1 kΩ** | 64 | 7 per core and JMP_total |

### Full System Components

To create a standalone computer, you will also need:

| Component | Model | Purpose |
|-----------|-------|---------|
| BIOS EEPROM | 28C64 / 28C256 | Bootloader and monitor program |
| Data SRAM | 62256 (32K×8) | Data and address memory |
| Program Counter | 2×74HC163 (cascaded) | 8-bit PC |
| Clock source | NE555 or crystal | 1-50 kHz CLK |
| Bus buffers | 2×74HC245 | CPU isolation |

---

## 💻 Programming

### Emulated Operations

Because NAND is a universal gate, any logical operation can be emulated via short sequences of `NADJZ`:

| Operation | Implementation using NADJZ |
|-----------|----------------------------|
| **AND A, B** | NAND(NAND(A, B), NAND(A, B)) |
| **OR A, B** | NAND(NAND(A, A), NAND(B, B)) |
| **XOR A, B** | AND(NAND(A, B), OR(A, B)) |
| **NOT A** | NAND(A, A) |

### Assembly (Pseudo‑instructions)

A monitor program (running on an ATtiny or within the BIOS) can translate convenient pseudo‑instructions into sequences of `NADJZ`:

```assembly
; Blinking an LED connected to OUT0
START:
    MOV A, IN0      ; read bit from input 0
    NOT A           ; invert it
    OUT A, OUT0     ; write result to LED
    JMP START       ; repeat forever
```

### Machine Code

Each instruction is stored in memory as a single 8-bit word: 1 bit for `INS` followed by 7 bits of `DATA` (though only the LSB of `DATA` is used in the basic instruction set).

---

## 🎮 Applications

### 1. Driving an 8×8 LED Matrix

The 8‑bit data bus makes this CPU a natural fit for an 8×8 LED matrix:
- **OUT[0..7]** → rows or columns of the matrix.
- 8 × 8 = 64 pixels → enough for simple sprites and animations.

### 2. Simple Games

| Game | Requirements | Feasibility |
|------|--------------|--------------|
| **Snake on 8×8** | 64 pixels, simple logic | ✅ Easy |
| **Pong on 8×8** | Two paddles, a ball | ✅ Easy |
| **Simplified Space Invaders** | Low‑res sprites | ⚠️ Possible |
| **Tetris (8×10)** | 80 pixels (external display) | ✅ Easy |

### 3. Peripherals

| Device | How to interface |
|--------|------------------|
| **Video card** | ATtiny receives OUT[0..7] and drives a UART TFT display |
| **Sound** | Second ATtiny + buzzer for 4‑bit chiptune |
| **Keyboard** | PS/2 module → UART → ATtiny → IN[0..7] |

---

## 📈 Scaling

`PCPU-micro 8` is part of a whole family of scalable PCPU processors:

```
1‑bit    → PCPU‑NANO1 T4          (4 transistors,  1 core)
8‑bit    → PCPU‑micro 8           (32 transistors, 8 cores)
13‑bit   → PCPU‑MINI‑13           (52 transistors, 13 cores)
16‑bit   → PCPU‑MINI‑16           (64 transistors, 16 cores)
32‑bit   → PCPU‑MINI‑32           (128 transistors, 32 cores)
```

By adding more 1‑bit cores, you can create a processor of any word length.

---

## 🔄 Comparison with Similar Projects

| Parameter | PCPU-micro 8 | MC14500B | Ben Eater SAP‑1 |
|-----------|--------------|----------|-----------------|
| **Data width** | 8‑bit | 1‑bit | 8‑bit |
| **Technology** | Discrete transistors | Single chip | TTL logic |
| **Transistor count** | 32 | ≈1500 (internal) | ≈500-1000 |
| **Instructions** | 2 | 16 | ≈10 |
| **Max. frequency** | up to 10 MHz | up to 1 MHz | up to 1 MHz |
| **Build difficulty** | Mid | Low | Medium |

---

## 📚 Resources

### Related Projects

- **PCPU‑NANO1** – original 1‑bit architecture by **ppc‑8051** (archived).
- **PCPU‑NANO1 T4** – optimised 4‑transistor implementation.
- **PCPU‑micro 8** – 8‑bit version (this project).

### Tools

- **Falstad Circuit Simulator** – for logic emulation.

---

## 📋 License

MIT License – free to use, modify, and distribute.

```
MIT License

Copyright (c) 2026 Akhmadihin

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction...
```

---

<div align="center">

**Built from 32 transistors. 🔧**  
**Inspired by minimalism. 🎯**  
**Made for learning. 📚**  
**Thanks to ppc‑8051 for the original idea. 🙏**

</div>
