# CS184/284A Final Project
## Multiple Importance Sampling (MIS) + ML-guided Adaptive Sampling (SBAS)

This repository hosts the **project webpage (HTML)** for our CS184/284A final project: a path tracer that renders **cleaner images with fewer wasted samples** by improving:

1) **How we choose random light directions** (MIS), and  
2) **Where we spend samples in the image** (SBAS with a ViT/CLIP saliency prior)

**Project webpage:** https://eduardo1100.github.io/cs-184-final-project-webpage/final_materials/index.html

---

## What we built (plain-English + rigorous)

### 1) Advanced materials (BSDFs)
A **BSDF** (Bidirectional Scattering Distribution Function) is the rule that tells the renderer how a surface redirects incoming light. Different BSDFs produce different visual effects and also change what sampling strategies work well.

We implemented:

- **Mirror (delta reflection)**  
  A perfect mirror reflects light into exactly one direction (a “delta” distribution). Almost all random directions contribute nothing, so correct sampling matters.

- **Glass (delta reflection + refraction with Fresnel)**  
  Glass splits light into reflection and transmission. The split is governed by **Fresnel**, which depends on incident angle and indices of refraction. We handle **total internal reflection** as a special case.

- **Microfacet conductor (metal)**  
  A metal surface is modeled as many tiny mirror-like facets with a statistical distribution.  
  We use **Beckmann** for the normal distribution function (NDF) and exact conductor Fresnel with complex IOR parameters \((\eta, k)\).

For reference, the microfacet model evaluates (for valid hemispheres):
\[
f(\omega_o,\omega_i)=\frac{D(\mathbf{h})\,F(\omega_i\!\cdot\!\mathbf{h})\,G(\omega_o,\omega_i)}{4\,\cos\theta_i\,\cos\theta_o},
\quad
D(\mathbf{h}) = \frac{e^{-\tan^2\theta_h/\alpha^2}}{\pi \alpha^2 \cos^4\theta_h}.
\]

---

### 2) Multiple Importance Sampling (MIS) for direct lighting
To compute **direct lighting** at a surface point, we estimate an integral over incoming light directions:
\[
L_{\text{dir}}(\mathbf{x},\omega_o)=\int_{\Omega} f(\omega_o,\omega_i)\,L_i(\omega_i)\,|\cos\theta_i|\,d\omega_i.
\]

The challenge: depending on the scene/material, one sampling strategy may be much better than another.

We therefore combine two strategies:

- **Light sampling:** pick directions that aim at the light sources  
- **BSDF sampling:** pick directions the material itself prefers

Each estimator is unbiased by dividing by the PDF:
\[
\widehat{L}=\frac{f(\omega_o,\omega_i)\,L_i(\omega_i)\,|\cos\theta_i|}{p(\omega_i)}.
\]

We blend them using the **power heuristic** (\(\beta=2\)):
\[
w_{\text{light}}=\frac{p_{\text{light}}^2}{p_{\text{light}}^2+p_{\text{bsdf}}^2},
\qquad
w_{\text{bsdf}}=\frac{p_{\text{bsdf}}^2}{p_{\text{light}}^2+p_{\text{bsdf}}^2}.
\]

**Delta-aware handling (important):**  
For mirror/glass (delta BSDFs), light sampling almost never proposes the exact specular direction. That can create rare, extremely bright samples (“fireflies”). We reduce this by biasing weights toward the BSDF branch on delta surfaces.

---

### 3) SBAS: Saliency-Biased Adaptive Sampling (ViT-guided)
**Adaptive sampling** reduces wasted work by stopping sampling in pixels that have already converged (low uncertainty). We extend this with a **saliency prior**:

- We compute a **ViT/CLIP saliency map** \(s(x,y)\in[0,1]\) predicting which pixels are visually important.
- We use it to bias per-pixel sample budgets and/or convergence thresholds:
  - **Higher saliency → more samples / stricter convergence**
  - **Lower saliency → fewer samples / allow earlier stop**

Crucially, we include **safety rails** so the saliency prior cannot “wreck” the render if it’s wrong:
- A hard **minimum** and **maximum** samples per pixel
- Variance-based **confidence interval gating** still decides when a pixel is done
- Optional **annealing**: trust saliency more early, then let variance dominate later

---

## Visual comparison: Baseline vs SBAS (CBdragon @ 4096 spp)

### Baseline (adaptive sampling + MIS)

**Pixel sampling visualization (where the renderer spent samples):**  
![Baseline sampling](https://eduardo1100.github.io/cs-184-final-project-webpage/final_materials/CBdragon_4096spp_rate.png)

**Resulting render:**  
![Baseline render](https://eduardo1100.github.io/cs-184-final-project-webpage/final_materials/CBdragon_4096spp.png)

---

### SBAS (ViT-guided)

**Pixel sampling visualization (saliency-biased allocation):**  
![SBAS sampling](https://eduardo1100.github.io/cs-184-final-project-webpage/final_materials/CBdragon_SBAS_4096spp_rate.png)

**Resulting render:**  
![SBAS render](https://eduardo1100.github.io/cs-184-final-project-webpage/final_materials/CBdragon_SBAS_4096spp.png)

---

## Reading tip
If you’re new to rendering:
- Start with the project webpage’s **Overview** and **Glossary**
- Then read **MIS** (how we sample directions) before **SBAS** (how we allocate pixel budgets)

If you already know path tracing:
- Jump straight to the MIS section and look for the **delta-aware weighting** and **PDF definitions**
- Then check SBAS for the **clamps + CI gating** that keep the saliency prior safe
