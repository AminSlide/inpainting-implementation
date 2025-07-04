# 🖼️ Image Inpainting with Criminisi’s Algorithm

This repository provides a Python implementation of **exemplar-based image inpainting** using the algorithm described by Criminisi et al. in their paper:

📄 *Region Filling and Object Removal by Exemplar-Based Image Inpainting*  
[Criminisi, Pérez & Toyama, 2004 – Microsoft Research](https://www.microsoft.com/en-us/research/publication/object-removal-by-exemplar-based-inpainting/)

---

## 📌 Objective

The goal is to reconstruct missing regions in an image (defined by a binary mask) by filling them with similar patches from the known region, following a mathematically defined priority mechanism.

---

## 🧠 Mathematical Overview

The algorithm fills pixels along the boundary of the target region by computing a **priority score** for each pixel `p`:

### Priority Function

    P(p) = C(p) * D(p)

Where:
- `C(p)`: Confidence term
- `D(p)`: Data term (structure propagation)

![Priority Function](docs/formula_priority.png)

---

### 1. Confidence Term `C(p)`

Measures how much of the patch around pixel `p` is already known:

    C(p) = (1 / |Ψ_p|) * ∑_{q ∈ Ψ_p ∩ Ω^c} C(q)

- `Ω^c`: known (source) region  
- `|Ψ_p|`: size of patch centered at `p`

![Confidence Term](docs/formula_confidence.png)

---

### 2. Data Term `D(p)`

Encourages the continuation of strong image structures (isophotes) into the missing region:

    D(p) = |∇I⊥_p ⋅ n_p| / α

- `∇I⊥_p`: isophote at point `p` (direction orthogonal to gradient)  
- `n_p`: boundary normal at `p`  
- `α`: normalizing constant (e.g., 255)

![Data Term](docs/formula_data.png)

---

### 3. Patch Matching

Once the pixel with highest priority is chosen, its patch `Ψ_p` is filled by copying pixels from the best matching source patch `Ψ_q`:

    Ψ_q = argmin_{Ψ_r ∈ Ω^c} SSD(Ψ_p, Ψ_r)

Only known pixels are used in the SSD (Sum of Squared Differences) computation.

---

## 🔁 Algorithm Steps

1. Identify the boundary of the masked region.
2. Compute `P(p)` for all boundary pixels.
3. Select the patch `Ψ_p` with highest priority.
4. Search for best matching patch `Ψ_q` in the known region.
5. Copy known pixels from `Ψ_q` into the unknown area of `Ψ_p`.
6. Update the mask and confidence map.
7. Repeat until the region is fully filled.

---

## ▶️ How to Run

Place your input images and corresponding masks in the appropriate folders:

```bash
python inpainting.py
