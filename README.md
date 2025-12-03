# Circle-Packing-Tool
A command-line utility for computing and visualizing optimal circle packings in rectangular regions using square, hexagonal, and hybrid arrangements.

# Circle Packing Tool – Documentation

**Version:** 0.8.0  

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Installation](#installation)
- [Usage](#usage)
  - [Mode 1: Maximize Circles in Fixed Rectangle](#mode-1-maximize-circles-in-fixed-rectangle)
  - [Mode 2: Minimize Length for Fixed Width and N Circles](#mode-2-minimize-length-for-fixed-width-and-n-circles)
- [Input Format](#input-format)
- [Supported Units](#supported-units)
- [Packing Strategies](#packing-strategies)
- [Output and Visualization](#output-and-visualization)
- [Saving Results](#saving-results)
- [Dependencies](#dependencies)
- [Development Notes](#development-notes)
- [License](#license)

---

## Overview

The **Circle Packing Tool** helps engineers, designers, and educators determine how to optimally arrange equal-sized circles within rectangular spaces. It supports multiple packing algorithms and provides interactive visualization and comparison of results.

---

## Features

- **Two operational modes**:
  - **Mode 1**: Compute the maximum number of circles that fit in a fixed rectangle.
  - **Mode 2**: Compute the minimum required length to fit a given number of circles in a strip of fixed width.
- **Three packing algorithms**:
  - **Square packing**: Grid-aligned layout.
  - **Hexagonal packing**: Dense staggered rows (standard optimal for infinite planes).
  - **Hybrid packing**: Hexagonal rows with a top row shifted vertically by full diameter (not just vertical offset) to reduce required length—introduced in v0.8.0.
- **Unit-aware input**: Accepts measurements in `mm`, `cm`, `m`, `in`, or `ft`. Internally converts everything to centimeters.
- **Interactive visualization**: Compare layouts side-by-side with utilization percentages.
- **Export to PNG**: Save results for reports or documentation.
- **Robust error handling**: Validates inputs and gracefully handles infeasible configurations.

---

## Installation

Ensure you have Python 3.7+ installed. Then install dependencies:

```bash
pip install matplotlib
```

No additional packages are required.

Run the tool:

```bash
python circle_packing_tool.py
```

> **Note:** The script assumes `matplotlib` is available for plotting. Headless environments (e.g., servers without display) may require a backend like `Agg` if saving plots without display.

---

## Usage

Upon launch, you'll see:

```
=== Circle Packing Tool V0.8.0 ===

Select Mode:
1) Max circles in fixed rectangle
2) Calculate length for N circles
q) Quit
```

### Mode 1: Maximize Circles in Fixed Rectangle

**Use case**: You have a fixed sheet (e.g., metal, fabric) and want to know how many circles of a given size you can cut from it.

**Prompts**:
- `Rectangle length`
- `Rectangle width`
- `Circle diameter`

**Output**:
- Counts for square and hexagonal packings.
- Side-by-side plot showing both arrangements.
- Option to adjust any dimension interactively (`l`, `w`, `d`) or quit (`q`).

> **Example**:  
> Input: `Length = 100 cm`, `Width = 50 cm`, `Diameter = 10 cm`  
> Output: Square = 50 circles, Hex = 56 circles.

---

### Mode 2: Minimize Length for Fixed Width and N Circles

**Use case**: You need to pack *N* circles into a conveyor belt, roll of material, or strip of fixed width, and want the shortest possible length.

**Prompts**:
- `Fixed width`
- `Circle diameter`
- `Number of circles (N)`

**Output**:
- Required length for square and hex/hybrid layouts.
- Row/column breakdown.
- Interactive menu to:
  - View individual layouts.
  - Compare layouts side-by-side.
  - Save PNG files (e.g., `square_N20_80x50.png`).
  - Adjust `width`, `diameter`, or `N`.

> **Note**: The "Hex" result in this mode may use **hybrid packing** (introduced in v0.8.0) if it yields a shorter length.

---

## Input Format

All measurements follow the format:

```
<number> <unit>
```

- **Examples**: `12.5 cm`, `30in`, `2 m`
- **Unit is optional**: defaults to **centimeters** (`cm`).
- **Case-insensitive**: `CM`, `Cm`, `cm` are all accepted.
- **Whitespace**: Flexible (e.g., `10cm`, `10 cm`, ` 10   cm `).
- **Exit**: Type `q` at any input prompt to return to the main menu.

---

## Supported Units

| Unit | Symbol | Conversion to cm |
|------|--------|------------------|
| Millimeter | `mm` | 0.1 cm |
| Centimeter | `cm` | 1 cm |
| Meter | `m` | 100 cm |
| Inch | `in` | 2.54 cm |
| Foot | `ft` | 30.48 cm |

> **Tip**: Use consistent units for cleaner output display (e.g., all in `cm` or all in `in`).

---

## Packing Strategies

### 1. **Square Packing**
- Circles arranged in a regular grid.
- Vertical and horizontal spacing = diameter (`d`).
- Simpler but less dense (~78.5% max packing density).

### 2. **Hexagonal Packing**
- Rows staggered by `d/2` horizontally.
- Vertical spacing = `d × √3 / 2 ≈ 0.866d`.
- Theoretical max density = **π / (2√3) ≈ 90.7%**.
- Tries both orientations (portrait/landscape) and picks the better one.

### 3. **Hybrid Packing** (v0.8.0+)
- Uses hexagonal spacing for all but the **top row**.
- The top row is **shifted upward by a full diameter `d`** (not just `dy`) so it aligns vertically with the second-to-last row.
- Allows the use of **shorter total length** (`cols × d` instead of `cols × d + r`) while maintaining feasibility.
- Often reduces material waste in finite strips.

> **Why hybrid?** In constrained height scenarios, standard hex packing may waste space at the top. Hybrid packing exploits this space more efficiently.

---

## Output and Visualization

All plots include:
- **Rectangle boundary** (black outline).
- **Circles** (sky blue with steel blue edge).
- **Title** showing:
  - Packing type
  - Circle count
  - Area utilization (%) = (total circle area / rectangle area) × 100
  - Dimensions in selected unit
  - Circle diameter
- **Grid lines** for reference.
- **Equal aspect ratio** to preserve geometry.

In **comparison modes**, both plots share the same axis scale for fair visual assessment.

---

## Saving Results

In **Mode 2**, you can save layouts as PNG files:
- Filenames follow the pattern:  
  `square_N{N}_{length}x{width}.png`  
  `hex_N{N}_{length}x{width}.png`
- Resolution: 150 DPI.
- Includes all labels, title, and grid.

> **Note**: Only feasible layouts can be saved.

---

## Dependencies

- **Python ≥ 3.7**
- **matplotlib ≥ 3.0** (for `patches`, `pyplot`)
- Standard library: `math`, `re`

No external configuration or setup required.

---

## Development Notes

- **Extensibility**: The `PackingSolver` class is stateless and can be reused in other projects.
- **Debugging**: Set `DEBUG = True` at the top to see internal calculations.
- **Coordinate system**: All internal computations are in **centimeters**, with (0,0) at the bottom-left.
- **Numerical tolerance**: Uses `1e-9` to handle floating-point precision issues in boundary checks.

---

## License

This tool is provided as open-source educational software. See accompanying license file for redistribution terms.

---

> **Tip for Users**: For industrial cutting or nesting applications, this tool provides a **theoretical upper bound**. Real-world constraints (kerf, margins, material defects) may require additional spacing.
