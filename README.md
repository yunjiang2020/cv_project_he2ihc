
# Tumour Ratio Estimation from H&E Histology Images

This project focuses on estimating tumour ratios in histological whole-slide images (WSIs) by generating immunohistochemistry (IHC)-like images from Hematoxylin and Eosin (H&E) stained slides using deep learning. 

## Overview

Hematoxylin and eosin (H&E) staining is the most widely used histological technique for visualizing tissue morphology, but it lacks molecular specificity and cannot directly label tumour cells. In contrast, immunohistochemistry (IHC) stains such as Ki-67 provide explicit tumour cell labelling but are costly, less scalable, and often unavailable for large datasets.

This project addresses the problem of quantitatively estimating tumour burden from H&E images under weak supervision, using Ki-67 IHC slides as imperfect ground truth despite spatial misalignment between consecutive tissue sections. Unlike most clinical deep learning approaches that focus on binary classification or coarse staging, this work targets tumour ratio estimation, a task critical for cancer biology and treatment efficacy studies.

## Key Challenge

H&E and IHC slides are typically obtained from adjacent but non-identical tissue sections, making pixel-wise correspondence unreliable. Direct segmentation-based supervision is therefore infeasible.

To address this, the project reframes tumour estimation as a ratio prediction problem, based on the assumption that tumour-to-tissue ratios remain locally consistent across consecutive sections even when exact alignment is lost.

## Dataset and Preprocessing

- **Source**: 77 pairs of H&E and Ki67-stained IHC WSIs from [Zenodo](https://zenodo.org/records/11218961)
- **Preprocessing**:
  - Tiling and white background removal (IHC<0.6, HE<0.4)
  - Structural Similarity Index (SSIM) filtering (>0.4)
  - Final dataset: 206 high-quality patch pairs

**note** Evaluated image alignment using SSIM and cell-type distributions rather than IoU, due to expected spatial drift after H&E-to-IHC translation via CycleGAN. Prioritized biologically meaningful metrics to assess correspondence at a regional rather than pixel level.

### Ground Truth Generation

Instead of colour-thresholding IHC images (which are highly sensitive and prone to overestimation), a nucleus-level approach was adopted: 
- IHC patches were converted to the HED colour space
- StarDist was used to segment nuclei in: 
    Hematoxylin channel (total nuclei)
    Ki-67 channel (tumour-positive nuclei)
- Tumour ratio was computed as:
    Tumour-positive nuclei / total nuclei
This produced a more biologically meaningful and reproducible ground truth.

## Modelling Approaches
Four distinct modelling strategies were evaluated:
- **Pixel-level random forest segmentation (ilastik)** as an interpretable baseline
- **CycleGAN** for H&E → IHC translation, enabling indirect tumour estimation via synthetic IHC images
- **ResNet18 regression**, directly predicting tumour ratio from H&E patches
- **ViT-B16 regression**, leveraging global morphological context via transformers
For regression models:
- Slide-level train/test splitting was enforced to prevent data leakage
- Sampling strategies were applied to partially mitigate severe label skew toward low tumour ratios

## Results and Insights

- Random forest segmentation achieved high nominal accuracy but was strongly influenced by class imbalance
- CycleGAN generated visually plausible IHC-like images, but was constrained by GPU memory and patch resizing
- ResNet18 and ViT achieved comparable regression performance, with ViT slightly outperforming ResNet
- Training curves plateaued early, indicating that ground truth pairing quality—not model capacity—was the dominant performance bottleneck

Overall, the results show that quantitative tumour ratio estimation from H&E images is feasible without pixel-level supervision, but fundamentally limited by imperfect H&E–IHC alignment.

## Conclusion

This project presents a research-oriented computational pathology pipeline that systematically compares segmentation-based, generative, and discriminative approaches for tumour ratio estimation under weak supervision. It highlights the trade-offs between interpretability, robustness, and data requirements, and demonstrates that modern CNN and transformer models can match generative approaches without strictly paired inputs.

## Future Directions

- Improve H&E–IHC pairing at the patch level
- Explore hybrid approaches combining generative translation with direct regression
- Extend the pipeline to whole-slide tumor burden estimation

## Keywords
Computational Pathology · Histology · Weak Supervision · Tumor Burden Estimation · CNN · Vision Transformer · GAN · Biomedical AI

## References

1. Petríková et al. (2024) — [Ki67 expression classification](https://doi.org/10.5220/0012535900003657)  
2. Isola et al. (2017) — [Pix2Pix](https://arxiv.org/abs/1611.07004)  
3. Zhang et al. (2022) — [Pyramid Pix2Pix](https://ieeexplore.ieee.org/document/9746963)  
4. Zhu et al. (2017) — [CycleGAN](https://arxiv.org/abs/1703.10593)
