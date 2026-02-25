
# Ki-67 Estimation from H&E Histology Images

This project aims to predict Ki-67 proliferation score directly from H&E whole-slide images, using paired Ki-67 IHC slides to generate weak labels for supervised learning.

## Overview

Hematoxylin and eosin (H&E) staining is the most widely used histological technique for visualizing tissue morphology, but it lacks molecular specificity and does not directly report cellular proliferation activity. In contrast, immunohistochemistry (IHC) markers such as Ki-67 provide explicit labeling of proliferating cells, but are costly, less scalable, and often unavailable for large datasets.

This project investigates quantitative estimation of Ki-67 proliferation index directly from H&E whole-slide images under weak supervision. Paired Ki-67 IHC slides are used to derive approximate proliferation scores despite spatial misalignment between consecutive tissue sections. Unlike many clinical deep learning studies that focus on binary classification or coarse staging, this work targets continuous proliferation score prediction, which is critical for studying tumour biology, growth dynamics, and treatment response.

## Key Challenge

H&E and IHC slides are obtained from adjacent but non-identical tissue sections, making pixel-wise correspondence unreliable and preventing direct cell-level supervision.

To address this, the project reframes the task as a local proliferation ratio prediction problem, based on the assumption that regional Ki-67–derived proliferation statistics remain approximately consistent across consecutive sections, even when exact spatial alignment is imperfect.

## Dataset and Preprocessing

- **Source**: 77 pairs of H&E and Ki67-stained IHC WSIs from [Zenodo](https://zenodo.org/records/11218961)
- **Preprocessing**:
  - Tiling and white background removal (IHC<0.6, HE<0.4)
  - Structural Similarity Index (SSIM) filtering (>0.4)
  - Final dataset: 206 high-quality patch pairs

**note** Because H&E and IHC slides originate from adjacent but non-identical tissue sections, pixel-wise alignment is unreliable. Therefore, alignment quality was assessed using regional structural similarity (SSIM) and consistency of nuclei density distributions, rather than IoU-based overlap. This prioritizes biologically meaningful correspondence over strict spatial matching.

### Ground Truth Generation

Instead of colour-thresholding IHC images (which are highly sensitive and prone to overestimation), a nucleus-level approach was adopted: 
- IHC patches were converted to the HED colour space
- StarDist was used to segment nuclei in: 
    Hematoxylin channel (total nuclei)
    Ki-67 channel (tumour-positive nuclei)
- Tumour ratio was computed as:
    Tumour-positive nuclei area / total nuclei area
This produced a more biologically meaningful and reproducible ground truth.

## Modelling Approaches
Four distinct modelling strategies were evaluated:
- **Pixel-level random forest segmentation (ilastik)** as an interpretable baseline
- **CycleGAN** for H&E → IHC translation, enabling indirect tumour estimation via synthetic IHC images
- **ResNet18 regression**, directly predicting tumour ratio from H&E patches
- **ViT-B16 regression**, leveraging global morphological context via transformers
For regression models:
- Slide-level splitting was strictly enforced to prevent patch-level data leakage.
- Stratified sampling was applied to mitigate label skew toward low proliferation ratios.

## Results and Insights

- Random forest segmentation (ilastik) achieved high nominal accuracy but was highly sensitive to class imbalance, demonstrating that pixel-level tree-based methods can overfit skewed proliferation distributions without careful rebalancing.
- CycleGAN-based stain translation produced visually plausible synthetic IHC images. However, training was constrained by limited patch-level context and GPU memory requirements. The absence of true pixel-wise alignment between serial sections further limited its utility for precise quantitative estimation.
- ResNet18 and ViT-B/16 regression models achieved comparable Ki-67 ratio prediction performance. The Vision Transformer showed modest improvement, suggesting that increased receptive field and global context provide some benefit, but gains are limited under weak and noisy supervision. 
- While regression models were affected by label imbalance, they exhibited greater robustness compared to tree-based segmentation approaches, likely due to continuous optimization and loss smoothing effects.
- Training curves plateaued early, indicating that performance was constrained less by model capacity and more by supervision quality — specifically, residual H&E–IHC misalignment and proxy-label noise.
- Cross-modality feature transfer from H&E morphology to Ki-67 proliferation signal appears feasible but limited, suggesting that improved contextual modelling, stronger regularization, or better proxy supervision may be required for substantial performance gains.

## Conclusion

These findings demonstrate that quantitative Ki-67 proliferation index estimation from H&E images is feasible under weak supervision. However, performance is fundamentally constrained by:
 - Imperfect spatial correspondence between adjacent sections
 - Construct mismatch between morphology and proliferation
 - Limited dataset scale and distribution skew

Improvements are therefore more likely to come from enhanced supervision quality and alignment strategies than from increasing model complexity alone.

## Future Directions

- Improve patch-level H&E–IHC correspondence by incorporating alignment-aware sampling and automated quality scoring. This may include regional structural similarity metrics, nuclei density consistency checks, and exclusion of poorly registered areas prior to supervision transfer.
- Extend cross-modality supervision to additional clinically relevant biomarkers, such as HER2 and Vimentin, to evaluate whether morphology-to-marker prediction generalizes beyond proliferation signals and across distinct molecular expression patterns.
- Investigate hybrid modelling approaches that combine generative stain translation with direct regression, where synthetic IHC outputs provide auxiliary supervision while regression models retain quantitative robustness.
- Scale from patch-level prediction to slide-level inference using aggregation strategies (e.g., weighted pooling, stratified sampling of high-variance regions) and uncertainty-aware modelling to improve reliability of whole-slide proliferation estimation.

## Keywords
Computational Pathology · Histology · Weak Supervision · Tumor Burden Estimation · CNN · Vision Transformer · GAN · Biomedical AI

## References

1. Petríková et al. (2024) — [Ki67 expression classification](https://doi.org/10.5220/0012535900003657)  
2. Isola et al. (2017) — [Pix2Pix](https://arxiv.org/abs/1611.07004)  
3. Zhang et al. (2022) — [Pyramid Pix2Pix](https://ieeexplore.ieee.org/document/9746963)  
4. Zhu et al. (2017) — [CycleGAN](https://arxiv.org/abs/1703.10593)
