# 🖼️ Image Inpainting with Criminisi’s Algorithm

This repository contains a Python implementation of the exemplar-based inpainting algorithm described by Criminisi et al. in their 2004 paper:

📄 *Region Filling and Object Removal by Exemplar-Based Image Inpainting*  
[Microsoft Research](https://www.microsoft.com/en-us/research/publication/object-removal-by-exemplar-based-inpainting/)

---

## 📌 Objective

The goal is to reconstruct missing regions in an image (defined by a binary mask) by copying the most relevant patches from the known region. The algorithm follows a priority-based filling strategy, which favors areas where image structures (like edges or textures) are most likely to continue smoothly.

---

## 🧠 Mathematical Overview

At each iteration, the algorithm selects the **most promising patch** to inpaint based on a **priority function** defined for every pixel `p` on the boundary of the target region:

### Priority Function

    P(p) = C(p) * D(p)

Where:
- `P(p)`: Priority of the patch centered at pixel `p`
- `C(p)`: Confidence term — represents the reliability of surrounding pixels
- `D(p)`: Data term — promotes the continuation of strong image structures (isophotes)

---

## 🔷 1. Confidence Term `C(p)`

This term reflects how much of the patch `Ψ_p` around pixel `p` is already known. It is calculated as the **average confidence of the known pixels within the patch**:

    C(p) = (1 / |Ψ_p|) * Σ C(q), for all q ∈ Ψ_p ∩ Ω^c

Where:
- `Ψ_p`: square patch of fixed size centered at pixel `p`
- `Ω^c`: known (source) region of the image
- `|Ψ_p|`: total number of pixels in the patch
- `C(q)`: confidence value of pixel `q` (1 for known pixels, 0 for unknown)

🧠 This term favors patches that are well surrounded by known (reliable) pixels.

---

## 🔷 2. Data Term `D(p)`

This term encourages the continuation of edges or linear structures into the unknown region. It is computed as:

    D(p) = |∇I⊥(p) ⋅ n(p)| / α

Where:
- `∇I⊥(p)`: the isophote vector at `p` (i.e. the direction orthogonal to the image gradient ∇I)
- `n(p)`: the unit normal vector to the boundary of the missing region at `p`
- `α`: normalization constant (e.g., 255)

📈 The data term is **maximal when the isophote direction is aligned with the boundary normal**, meaning an edge is pointing directly into the missing region — this encourages structure continuation.

---

## 🔷 3. Patch Matching (Exemplar Search)

After selecting the patch centered at the highest-priority pixel `p*`, the algorithm searches for the **most similar source patch** `Ψ_q` within the known region. The similarity is computed using the **Sum of Squared Differences (SSD)** over known pixels only.

### SSD Formula

    SSD(Ψ_p*, Ψ_q) = Σ [I_p(i) - I_q(i)]², for all i ∈ K

Where:
- `Ψ_p*`: the target patch centered at the selected boundary pixel `p*`
- `Ψ_q`: a candidate source patch fully inside the known region `Ω^c`
- `I_p(i)`, `I_q(i)`: intensity of pixel `i` in each patch
- `K`: the set of pixels in `Ψ_p*` that are already known (i.e., not masked)

### Patch Selection

    Ψ_q = argmin_{Ψ_r ∈ Ω^c} SSD(Ψ_p*, Ψ_r)

The best patch `Ψ_q` is the one minimizing the SSD. Its known pixels are copied into the unknown parts of `Ψ_p*`.

🎯 This process helps maintain texture and structural consistency across the filled region.

---

## 🔁 Algorithm Steps

1. Detect the boundary (`∂Ω`) of the target region (the mask).
2. For each pixel `p` on the boundary:
    - Compute `C(p)`, the confidence term
    - Compute `D(p)`, the data term
    - Compute `P(p) = C(p) * D(p)`
3. Select the patch `Ψ_p*` with the highest priority.
4. Find the best matching patch `Ψ_q` in the known region that minimizes SSD.
5. Copy the known pixels from `Ψ_q` to the corresponding unknown pixels in `Ψ_p*`.
6. Update the mask and the confidence map.
7. Repeat until all missing regions are filled.
