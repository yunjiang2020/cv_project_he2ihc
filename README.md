
# Tumor Ratio Estimation from Histological Images Using GANs

This project focuses on estimating tumor ratios in histological whole-slide images (WSIs) by generating immunohistochemistry (IHC)-like images from Hematoxylin and Eosin (H&E) stained slides using deep learning. 

## Project Overview

- **Goal**: To develop a computational method for estimating tumor-to-normal cell ratios from H&E-stained slides using generative models trained on IHC-labeled slides.
- **Use Case**: Cancer biology research, where quantifying tumor load is essential for evaluating treatment effectiveness, especially in preclinical models with high metastatic burden.

## Biological Background

- **H&E Staining**: Common but subjective method for detecting tumors based on morphology.
- **IHC Staining**: More consistent labeling of tumor cells but not directly aligned with H&E slides.
- **Challenge**: Pixel-wise alignment between H&E and IHC slides is imperfect due to sectioning differences.

### Data

- **Source**: 77 pairs of H&E and Ki67-stained IHC WSIs from [Zenodo](https://zenodo.org/records/11218961)
- **Preprocessing**:
  - Tiling and white background removal (IHC<0.6, HE<0.4)
  - Structural Similarity Index (SSIM) filtering (>0.4)
  - Final dataset: 206 high-quality patch pairs

**note** Evaluated image alignment using SSIM and cell-type distributions rather than IoU, due to expected spatial drift after H&E-to-IHC translation via CycleGAN. Prioritized biologically meaningful metrics to assess correspondence at a regional rather than pixel level.

### Ground Truth Generation

- **Method**: StarDist for cell nuclei segmentation (tutorial: https://www.youtube.com/watch?v=L3dZ6fgmllI code adapted from the author's GitHub)
- **Rationale**: More reliable than color thresholding in estimating nuclear areas
![image](https://github.com/user-attachments/assets/26305b0d-33b2-4f36-ae47-66a4e7c66657)

![image](https://github.com/user-attachments/assets/d362d3b8-99cf-4701-abc6-5a29b145da70)



## Baseline: ilastik Segmentation

- **Approach**: Manual annotation + ilastik random forest model
- **Output**: Tumor ratios from H&E and IHC compared
- **Results**: MAE = 0.0455; MSE = 0.0084; Accuracy = 75–93% depending on threshold
![image](https://github.com/user-attachments/assets/a6ce450e-8036-4059-a77b-910e35c7f298)

![image](https://github.com/user-attachments/assets/a7b6020d-8f4e-48e7-9bd1-bc6ae1a224dc)



## Deep Learning Models

### 1. Pix2Pix (Pyramid version) https://github.com/bupt-ai-cz/BCI
- **Model**: Pre-trained pyramid pix2pix
- **Challenges**: Poor SSIM, due to domain differences (HER2 vs. Ki67, 20× vs. 40× magnification)
![image](https://github.com/user-attachments/assets/1476dac8-3101-4c2f-8262-3945ebdef080)


### 2. CycleGAN https://github.com/junyanz/pytorch-CycleGAN-and-pix2pix
- **Advantage**: No need for paired data
- **Training**: 389 images, various crop sizes
- **Best Results**: Accuracy = 0.6 with load size of 4096
- **Limitation**: GPU constraints in Google Colab
![image](https://github.com/user-attachments/assets/de8f2a69-e102-4cac-9060-790974b3bf95)

![image](https://github.com/user-attachments/assets/bf10b3bf-bea2-4310-93d1-17da4eb965dc)
![image](https://github.com/user-attachments/assets/a6202823-a43e-4795-bb68-de2d773e1fa3)
![image](https://github.com/user-attachments/assets/c9ac1a6e-688b-4670-90f2-9aa2db27c201)

Confusion matrix with different load sizes. left to right: 256/2048/4096 pixels


## Future Directions

- Try [VirtualMultiplexer (CUT-based)](https://www.biorxiv.org/content/10.1101/2023.11.29.568996v1)
- Experiment with [Swin Transformer-based GANs](https://arxiv.org/abs/2403.18501)
- Explore alternative data sources with prominent macrometastases to facilitate more interpretable categorization and improved human-in-the-loop validation.

## Lessons Learned

This project deepened our understanding of:
- Basic segmentation methods (StarDist, ilastik)
- GAN-based translation (pix2pix, CycleGAN)
- The trade-off between biological realism and computational feasibility

## Demo 
- (https://drive.google.com/file/d/12IoIveg_qdps3BgCb3HgsSlf1vMamIky/view?usp=drive_link)

## Timeline

| Phase                            | Date Range (2024)  |
|----------------------------------|--------------------|
| Background & Preprocessing       | Oct 8 – Oct 24     |
| Ground Truth & Baseline          | Oct 25 – Nov 7     |
| Model Training & Evaluation      | Nov 8 – Nov 21     |
| Final Report & Presentation      | Nov 21 – Dec 5     |

## References

1. Petríková et al. (2024) — [Ki67 expression classification](https://doi.org/10.5220/0012535900003657)  
2. Isola et al. (2017) — [Pix2Pix](https://arxiv.org/abs/1611.07004)  
3. Zhang et al. (2022) — [Pyramid Pix2Pix](https://ieeexplore.ieee.org/document/9746963)  
4. Zhu et al. (2017) — [CycleGAN](https://arxiv.org/abs/1703.10593)
