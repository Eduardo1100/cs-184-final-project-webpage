# CS184/284A Final Project — MIS + ML-guided Adaptive Sampling (SBAS)

This repo hosts the **GitHub Pages** write-up for our CS184/284A final project: extending a path tracer with:

- **Advanced BSDFs**: mirror (delta reflection), glass (delta reflection + refraction with Fresnel), and **microfacet conductor** (Beckmann NDF + conductor Fresnel with \(\eta, k\)).
- **Multiple Importance Sampling (MIS)** for direct lighting: **light sampling + BSDF sampling** blended with the **power heuristic** (with delta-aware handling).
- **Saliency-Biased Adaptive Sampling (SBAS)**: a **ViT/CLIP saliency prior** used to bias per-pixel sampling budgets and convergence thresholds.

The site is a single static HTML page (with KaTeX for math) intended to be served via **GitHub Pages**.

---

[Live Site](https://eduardo1100.github.io/cs-184-final-project-webpage/final_materials/index.html)



