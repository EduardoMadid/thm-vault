---
title: Impact Printers - CompTIA A+ 220-1201 - 3.8
tags:
  - comptiaA+
  - core1
---


# Impact Printers

## Overview

Impact printers physically strike the paper to create a mark. The most common type is the **dot-matrix printer**.

- Print head holds a small **matrix of pins** (commonly 9 or 24 pins)
- Pins press an **inked ribbon** against the paper
- Characters and graphics are built from a grid of dots
- The only printer type that can produce **carbon / multipart copies** in one pass

## Pros and Cons

|Pros|Cons|
|---|---|
|Multipart (carbon copy) printing in a single pass|Very noisy|
|Low cost per page|Poor graphics quality|
|Cheap, durable, tolerant of harsh environments|Slow compared to inkjet / laser|
|Works with continuous-feed forms|Niche use cases, hard to source supplies|

**Where you still find them:** point-of-sale receipts, auto repair shops, shipping and warehouse forms, medical/pharmacy labels, banks, industrial floors.

## Print Head

- Moves back and forth **horizontally** across the page
- Pins fire against the ribbon, ribbon hits the paper
- There is only **one matrix** of pins, so the head must traverse the page to print a full line
- Gets **hot** during use — let it cool before servicing
- Wears out over time; a stuck or broken pin leaves a missing dot row

## Printer Ribbon

- Fabric ribbon soaked with ink
- One long loop — a **continuous / never-ending circle**
- Replaced as a **single unit** (cartridge), which makes swaps quick and clean
- **Proprietary size** — specific to the printer model, no universal ribbon
- Faded or light print = ribbon is worn out, replace it (do not "re-ink")

## Tractor Feed

- **Continuous paper feed** — one long strip with perforations between pages
- Sprockets grab the **holes along both edges** of the paper and pull it through
- Uses pin/sprocket traction instead of **friction feed** rollers
- Result: precise alignment, no slipping, unattended long print jobs
- The perforated strips on the sides are torn off after printing

text

```text
   Tractor-feed paper
 ┌───────────────────────┐
 │ o                   o │
 │ o                   o │
 │ o                   o │  <- sprocket holes on both edges
 ├ - - - - - - - - - - - ┤  <- perforation between pages
 │ o                   o │
 │ o                   o │
 └───────────────────────┘
```

## Multipart Paper (Carbonless Copy)

- Produces **multiple copies of a single form in one pass**
- The impact on the top sheet is transferred to the sheets underneath
- How it works:
    - **Micro-encapsulated ink (dye)** coated on the _back_ of the first sheet
    - **Clay-based reactive surface** on the _front_ of the sheet below
    - The pin strike bursts the capsules, the dye reacts with the clay, the mark appears
- Requires a real **impact**, so inkjet and laser printers cannot do this
- Those inks, dyes and clays can cause **mild skin irritation** for some people
- Today it is usually cheaper and easier to just print another copy — multipart paper and dot-matrix printers are getting hard to find

## Maintenance and Troubleshooting

| Symptom                                  | Likely cause / fix                                     |
| ---------------------------------------- | ------------------------------------------------------ |
| Faded or light print                     | Worn ribbon — replace the ribbon cartridge             |
| White horizontal line through characters | Broken or stuck print head pin — replace print head    |
| Smudged / smeared output                 | Ribbon misaligned or print head too close to platen    |
| Paper jams, skewed pages                 | Tractor feed misaligned, paper not seated on sprockets |
| Nothing prints, head moves               | Ribbon installed wrong or completely dry               |

**Consumables to know for the exam:** ribbon, tractor-feed paper, print head.

## Quick Recall

- Impact printer = strikes the paper. Dot-matrix = pins + ribbon.
- Only technology that does **carbon copies / multipart forms**.
- Ribbon is a **continuous fabric loop**, proprietary, replaced as one unit.
- Tractor feed = **sprocket holes**, continuous paper, not friction.
- Multipart = micro-encapsulated ink + clay reaction, no carbon paper needed.