# 🖼️ Image Inpainting with Criminisi’s Algorithm

This repository contains a Python implementation of the exemplar-based image inpainting algorithm proposed by **Antonio Criminisi et al.** in their 2004 paper:  
📄 *Region Filling and Object Removal by Exemplar-Based Image Inpainting* ([paper link](https://www.microsoft.com/en-us/research/publication/object-removal-by-exemplar-based-inpainting/))

---

## 📌 Objective

The goal is to **reconstruct missing regions** in an image (defined by a binary mask) by copying similar patches from the known region, using a priority-driven mechanism. The algorithm fills the missing area **iteratively**, starting from the most “promising” pixels along the boundary.

---

## 🧠 Mathematical Overview

The algorithm relies on a **priority function** defined for each boundary point \( p \) of the target region:

\[
P(p) = C(p) \cdot D(p)
\]

- \( C(p) \): **Confidence term** – how reliable the pixel's neighborhood is.
- \( D(p) \): **Data term** – how strong the image structure (isophote) is at point \( p \).

---

### 1. Confidence Term \( C(p) \)

Measures how much information around pixel \( p \) is already known. For a patch \( \Psi_p \) centered at \( p \):

\[
C(p) = \frac{1}{|\Psi_p|} \sum_{q \in \Psi_p \cap \Omega^c} C(q)
\]

- \( \Omega^c \): known (source) region.
- \( |\Psi_p| \): area of the patch.

---

### 2. Data Term \( D(p) \)

Encourages continuation of linear structures (isophotes) into the target region:

\[
D(p) = \frac{|\nabla I^{\perp}_p \cdot n_p|}{\alpha}
\]

- \( \nabla I^{\perp}_p \): isophote at point \( p \) (i.e., direction perpendicular to the image gradient).
- \( n_p \): normal vector to the boundary at \( p \).
- \( \alpha \): normalizing factor (e.g. 255).

In practice:
- The **isophote** is computed via the image gradient:
  \[
  \nabla I = \left[ \frac{\partial I}{\partial x}, \frac{\partial I}{\partial y} \right]
  \]
  and rotated by 90° (via cross product with the z-axis).
  
- The **normal** to the contour is calculated from boundary points using:
  \[
  n_p = \frac{(p_{i+1} - p_i) \times \mathbf{z}}{\| (p_{i+1} - p_i) \times \mathbf{z} \|}
  \]

---

## 🔁 Inpainting Loop

The following steps are repeated until the mask is fully filled:

1. **Extract boundary** \( \delta \Omega \) of the target region.
2. **Compute priority** \( P(p) \) for each \( p \in \delta \Omega \).
3. Select the patch \( \Psi_p \) with **maximum priority**.
4. Find the best matching source patch \( \Psi_q \in \Omega^c \) by minimizing the **Sum of Squared Differences (SSD)**:
   \[
   \Psi_q = \arg\min_{\Psi_r \in \Omega^c} \| \Psi_r - \Psi_p \|^2
   \]
   (only over known pixels).
5. Copy pixels from \( \Psi_q \) to the unknown pixels of \( \Psi_p \).
6. Update confidence map \( C(p) \) and the mask.

---

## 🗂️ Project Structure

```bash
.
├── images/             # Input images to inpaint
├── masks/              # Binary masks (_mask.jpg) for target regions
├── output/             # Output images and debug visualizations
├── inpainting.py       # Main class implementing Criminisi's algorithm
└── README.md           # This file
