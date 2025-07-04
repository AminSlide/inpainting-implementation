# 🖼️ Image Inpainting with Criminisi’s Algorithm

This repository contains a Python implementation of the exemplar-based inpainting algorithm described by Criminisi et al. in their 2004 paper:

📄 *Region Filling and Object Removal by Exemplar-Based Image Inpainting*  
[Microsoft Research](https://www.microsoft.com/en-us/research/publication/object-removal-by-exemplar-based-inpainting/)

---

## 📌 Objective

The goal is to reconstruct missing regions of an image (defined by a binary mask) by copying the most relevant patches from the known region. The algorithm follows a priority-based filling strategy, which favors areas where image structures (like edges or textures) are most likely to continue smoothly.

---

## 🧠 Mathematical Overview

At each iteration, the algorithm selects the **most promising patch** to inpaint based on a **priority function** defined for every pixel `p` on the boundary of the target region:

### Priority Function

    P(p) = C(p) * D(p)

- `P(p)`: Priority of the patch centered at pixel `p`
- `C(p)`: Confidence term — represents the reliability of surrounding pixels
- `D(p)`: Data term — promotes the continuation of strong image structures (isophotes)

---

## 🔷 1. Confidence Term `C(p)`

This term reflects how much of the patch `Ψ_p` around pixel `p` is already known. It is calculated as the **average confidence of the known pixels within the patch**:

    C(p) = (1 / |Ψ_p|) * Σ C(q), for all q in Ψ_p ∩ Ω^c

Where:
- `Ψ_p` is the square patch of fixed size centered at pixel `p`
- `Ω^c` is the known (source) region of the image
- `|Ψ_p|` is the number of pixels in the patch
- `C(q)` is the confidence value of pixel `q` (initially 1 for known pixels, 0 for unknown)

🧠 This term ensures that the algorithm prefers to fill patches that are **well surrounded by known pixels**.

---

## 🔷 2. Data Term `D(p)`

This term encourages the propagation of linear structures like edges into the missing region. It is computed using the **dot product between the isophote direction and the boundary normal** at `p`:

    D(p) = |∇I⊥(p) ⋅ n(p)| / α

Where:
- `∇I⊥(p)` is the isophote vector at `p`, i.e., the direction **perpendicular to the gradient** of the image intensity
- `n(p)` is the **unit normal vector** to the boundary of the missing region at `p`
- `α` is a normalization constant (typically 255)

📈 The **data term is maximal** when the isophote direction is **aligned with the normal vector**, i.e., when the structure is pointing straight into the region to be filled. This prioritizes pixels where edges are likely to **continue naturally** into the hole.

---

## 🔷 3. Patch Matching (Exemplar Search)

Once the pixel `p*` with highest priority is found, the algorithm searches for the **most similar patch `Ψ_q`** in the known region using the **Sum of Squared Differences (SSD)**:

    Ψ_q = argmin_{Ψ_r ∈ Ω^c} SSD(Ψ_p*, Ψ_r)

- Only the **known pixels** of `Ψ_p*` are used in the SSD computation
- The matched patch `Ψ_q` is copied into the unknown part of `Ψ_p*`

🎯 This allows the algorithm to **reuse existing textures** in the image in a visually coherent way.

---

## 🔁 Algorithm Steps

1. Detect the boundary (`∂Ω`) of the target region (the hole).
2. For each boundary pixel `p`:
    - Compute the confidence term `C(p)`
    - Compute the data term `D(p)`
    - Compute the priority `P(p) = C(p) * D(p)`
3. Choose the patch centered at `p*` with the **highest priority**.
4. Search for the best matching patch in the known region using SSD.
5. Copy pixels from the source patch to the unknown pixels.
6. Update the mask and confidence map.
7. Repeat until all missing pixels are filled.

