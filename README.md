# AI Imaging Analysis Pipeline

An experimental AI-assisted biomedical imaging pipeline for **nuclei image analysis**, combining deep learning segmentation, classical image processing, Vision-Language Models (VLMs), and Large Language Models (LLMs).

The project investigates segmentation performance, image interpretation, explainability, auditability, and the risk of hallucination in AI-assisted biomedical imaging workflows.

> **Academic project:** This repository was developed for coursework and is not intended for clinical deployment.

---

## Overview

This project investigates an AI-assisted biomedical imaging pipeline for nuclei image analysis.

The pipeline combines:

* Exploratory Data Analysis (EDA)
* Deep learning-based image segmentation
* Classical image processing
* Vision-Language Models (VLMs)
* Large Language Models (LLMs)
* Numerical feature extraction
* Error analysis
* Hallucination risk reduction
* Explainability and auditability

A trained **U-Net segmentation model** is compared with a classical **Otsu thresholding** approach. The project also investigates how LLMs can interpret image-derived measurements while reducing the risk of unsupported or hallucinated descriptions.

---

## Project Objectives

The project addresses five main questions:

1. Which description is more useful and trustworthy: a direct VLM description or a numbers-first description?
2. Did U-Net improve on classical Otsu segmentation?
3. What do the U-Net Dice and IoU scores mean, and where does the model make mistakes?
4. Where can an LLM hallucinate in the pipeline, and what design choices reduce that risk?
5. Considering accuracy, auditability, and dataset limitations, could any part of this system be trusted in a real clinical setting?

---

## Overall Pipeline

The project is organised into five notebooks, with each notebook performing a distinct stage of the analysis.

```text
                         Nuclei Dataset
                              │
                              ▼
                    1. Exploratory Data Analysis
                              │
                              ▼
                    2. U-Net Model Architecture
                              │
                              ▼
                    3. Model Evaluation
                              │
                              ▼
                    4. Hybrid AI Pipeline
                              │
                              ▼
                    5. Classical Post-Processing
                              │
                              ▼
                       Final Analysis
                         & Comparison
```

The notebooks are designed to follow a logical progression while avoiding unnecessary repetition between stages.

---

## Repository Structure

```text
Assignment_3/
│
├── notebooks/
│   ├── 01_EDA.ipynb
│   ├── 02_Model_Architecture.ipynb
│   ├── 03_Model_Evaluation.ipynb
│   ├── 04_Hybrid_Pipeline.ipynb
│   └── 05_Post_Processing.ipynb
│
├── outputs/
│   ├── figures/
│   └── ...
│
├── report/
│   └── Project Report.pdf
│
├── README.md
│
└── .gitignore
```

> **Note:** The original dataset and trained U-Net checkpoint are not included in the GitHub repository because of their size. The notebooks contain the analysis and processing code used to generate the reported results.

---

# Notebook 1 — Exploratory Data Analysis

## Purpose

The first notebook performs **Exploratory Data Analysis (EDA)** on the nuclei imaging dataset.

The purpose of this stage is to understand the dataset before developing the segmentation model.

The notebook investigates:

* Dataset structure
* Image dimensions
* Image intensity characteristics
* Segmentation masks
* Foreground/background distribution
* Training and validation data
* Representative image and mask examples
* Dataset quality and consistency

The EDA provides the foundation for the modelling decisions made in the following notebooks.

---

# Notebook 2 — U-Net Model Architecture

## Purpose

The second notebook defines and trains the **U-Net segmentation model** used to identify nuclei regions.

U-Net was selected because its encoder-decoder structure and skip connections are well suited to biomedical image segmentation.

The model was trained using segmentation losses and evaluated using overlap-based metrics.

## Training

The notebook contains:

* U-Net architecture definition
* Encoder and decoder blocks
* Skip connections
* Training configuration
* Loss calculation
* Validation monitoring
* Model checkpointing
* Training history

The best trained model was saved as a U-Net checkpoint and subsequently used for inference and evaluation.

---

# Notebook 3 — Model Evaluation

## Purpose

The third notebook evaluates the trained U-Net model independently from the training process.

The evaluation focuses on determining how accurately the trained model segments nuclei on unseen validation/test examples.

## Final Validation Results

| Metric          | Result |
| --------------- | -----: |
| Validation Loss | 0.0559 |
| Mean Dice       | 0.9948 |
| Mean IoU        | 0.9897 |
| Mean Precision  | 0.9915 |
| Mean Recall     | 0.9982 |

The best Dice and IoU scores were obtained at epoch 14:

```text
Best Dice = 0.99595
Best IoU  = 0.99194
```

The lowest validation loss occurred at epoch 15:

```text
Validation Loss = 0.05590
```

## Error Analysis

The notebook also identifies the lowest-performing validation images.

The five lowest-performing cases were examined using:

* Dice
* IoU
* Precision
* Recall

The lowest Dice score was observed for:

```text
Image 12
Dice = 0.9891
IoU  = 0.9784
```

The errors were generally small and associated with:

* Segmentation boundaries
* Small structures
* Minor shape differences
* Minor localisation differences

---

# Notebook 4 — Hybrid AI Pipeline

## Purpose

The fourth notebook combines image segmentation, numerical feature extraction, and LLM-based interpretation.

The goal is to investigate how an LLM can be used as an **interpretation layer** without allowing it to become the source of the underlying image measurements.

## Pipeline

```text
Input Image
     │
     ▼
U-Net Segmentation
     │
     ▼
Segmentation Mask
     │
     ▼
Connected Components
     │
     ▼
Region Properties
     │
     ▼
Numerical Summary
     │
     ▼
LLM Interpretation
     │
     ▼
Structured JSON
     │
     ▼
Human-readable Narrative
```

The pipeline successfully processed all **12 unseen test images**.

## Extracted Measurements

The numerical analysis includes:

* Number of objects
* Mean object area
* Median object area
* Eccentricity
* Solidity
* Extent
* Perimeter
* Mean intensity
* Foreground fraction

The final results table contains:

```text
image_id
n_objects
mean_area
density_class
quality_flag
shape_regularity
mean_eccentricity
mean_solidity
foreground_fraction
llm_status
narrative
```

---

# Numbers-First LLM Approach

A major design decision in the hybrid pipeline was to avoid relying on unrestricted LLM interpretation of the raw image.

Instead, measurable image features are extracted first and then supplied to the LLM.

For example:

```json
{
  "n_objects": 8,
  "density_class": "low",
  "shape_regularity": "regular",
  "quality_flag": "good"
}
```

The LLM then generates an interpretation based on these structured measurements.

This creates a clear separation between:

```text
Image
  ↓
Measurement
  ↓
Structured Data
  ↓
LLM Interpretation
```

The numerical measurements remain the **source of truth**, while the LLM provides a human-readable interpretation.

This improves auditability because the measurements can be independently inspected without relying on the generated narrative.

---

# Direct VLM Analysis

The project also investigated direct Vision-Language Model interpretation.

An initial freely worded prompt produced fluent but incorrect descriptions of the biomedical images, including interpretations resembling astronomical objects such as galaxies or star clusters.

This demonstrated the risk of **visual hallucination**.

An optimised prompt was therefore introduced to:

* Restrict the model to observable information
* Avoid diagnosis
* Avoid unsupported assumptions
* Allow uncertainty
* Require structured JSON output

The experiment demonstrated why unconstrained visual descriptions should not automatically be treated as reliable scientific observations.

---

# Notebook 5 — Classical Post-Processing

## Purpose

The fifth notebook investigates a transparent classical image-processing alternative to deep learning segmentation.

The approach uses **Otsu thresholding** followed by post-processing and region-property extraction.

## Classical Pipeline

```text
Input Image
     │
     ▼
Otsu Thresholding
     │
     ▼
Binary Mask
     │
     ▼
Morphological Processing
     │
     ▼
Connected Components
     │
     ▼
Region Properties
     │
     ▼
Object Measurements
```

## Extracted Features

The classical processing stage extracts:

* Object count
* Mean area
* Median area
* Eccentricity
* Solidity
* Extent
* Perimeter
* Mean intensity
* Foreground fraction

These measurements are also suitable for numbers-first LLM interpretation.

---

# Otsu Results

The classical Otsu method was evaluated on **12 test images**.

| Metric              | Result |
| ------------------- | -----: |
| Mean Dice           | 0.9776 |
| Mean IoU            | 0.9561 |
| Mean Precision      | 0.9776 |
| Mean Recall         | 0.9775 |
| Best Dice           | 0.9819 |
| Worst Dice          | 0.9694 |
| Mean Otsu Threshold | 0.1350 |

The results show that a relatively simple classical method can perform strongly on this dataset.

---

# U-Net vs Otsu

## U-Net

```text
Mean Dice = 0.9948
Mean IoU  = 0.9897
```

## Otsu

```text
Mean Dice = 0.9776
Mean IoU  = 0.9561
```

U-Net achieved higher segmentation scores in the reported evaluation.

However, the U-Net and Otsu results were obtained on different evaluation stages/splits. Therefore, the values should **not be interpreted as a perfectly controlled head-to-head comparison**.

The comparison provides an indication of performance, but a stronger experiment would evaluate both methods on exactly the same images using identical evaluation procedures.

---

# Key Findings

## 1. Direct VLM vs Numbers-First LLM

The direct VLM is useful for exploratory visual description, but the experiment demonstrated that it can produce confident descriptions that are not supported by the actual biomedical image.

The numbers-first approach is more trustworthy because its inputs are measurable and independently verifiable.

---

## 2. U-Net vs Classical Otsu

U-Net produced higher Dice and IoU values in the reported evaluation.

However, Otsu remains valuable because it is:

* Simple
* Fast
* Transparent
* Easy to reproduce
* Easy to audit

Therefore, classical methods remain useful as baselines and interpretability tools even when a deep learning model performs better.

---

## 3. Segmentation Quality

The U-Net achieved very high validation performance:

```text
Dice = 0.9948
IoU  = 0.9897
```

**Dice** measures the overlap between predicted and ground-truth segmentation masks.

**IoU (Intersection over Union)** measures the intersection of the predicted and ground-truth regions relative to their union.

Values close to 1 indicate strong agreement between the predicted and reference masks.

---

# LLM Hallucination and Risk Reduction

An LLM can hallucinate at several points in the pipeline.

For example, it may:

* Invent visual characteristics
* Infer biological information that is not present
* Misinterpret numerical values
* Produce unsupported classifications
* Generate plausible but incorrect narratives

Several design choices were therefore used to reduce this risk:

1. Constrained prompts
2. Structured JSON output
3. Explicit uncertainty options
4. Numerical feature extraction before LLM interpretation
5. Keeping structured measurements as the source of truth
6. Separating segmentation from language generation

The structured JSON output is particularly important because the numerical measurements can be inspected independently of the LLM-generated narrative.

---

# Trustworthiness and Clinical Considerations

This project is a **coursework investigation and not a clinical system**.

The high U-Net segmentation scores are encouraging, but they do not demonstrate clinical readiness.

Important limitations include:

* Relatively small dataset
* Limited evaluation population
* Lack of external multi-site validation
* Limited evidence of generalisation to different imaging conditions
* Potential LLM hallucination
* U-Net and Otsu were not evaluated using an identical controlled test set

The LLM should therefore remain an **interpretation layer** rather than a source of medical measurements or clinical decisions.

A larger, patient-level, externally collected validation study across multiple sites and imaging conditions would be an important requirement before considering real-world deployment.

---

# Technologies Used

The project uses the following technologies:

* **Python**
* **PyTorch**
* **NumPy**
* **Pandas**
* **OpenCV**
* **scikit-image**
* **Matplotlib**
* **Ollama**
* **Qwen2.5-VL**
* **Jupyter Notebook**

---

# Outputs

The `outputs/` directory contains generated results from the analysis, including:

* Segmentation metrics
* CSV result tables
* Evaluation summaries
* Prediction visualisations
* VLM/LLM outputs
* Hybrid pipeline results
* Classical post-processing results
* Representative figures

---

# Repository Contents

| Notebook                      | Main Purpose                                    |
| ----------------------------- | ----------------------------------------------- |
| `01_EDA.ipynb`                | Dataset exploration and understanding           |
| `02_Model_Architecture.ipynb` | U-Net architecture and model training           |
| `03_Model_Evaluation.ipynb`   | U-Net validation, metrics and error analysis    |
| `04_Hybrid_Pipeline.ipynb`    | U-Net + feature extraction + LLM interpretation |
| `05_Post_Processing.ipynb`    | Otsu segmentation and classical post-processing |

---

# Important Limitations

This repository was developed for academic coursework and is **not intended for clinical deployment**.

The main limitations are:

* Small dataset size
* Limited external validation
* No multi-site clinical validation
* Potential domain shift between datasets
* LLM-generated narratives may still contain unsupported information
* Segmentation performance does not automatically imply clinical usefulness
* U-Net and Otsu evaluations were not performed as a fully controlled head-to-head experiment

---

# References

1. Ronneberger, O., Fischer, P. & Brox, T. (2015). *U-Net: Convolutional Networks for Biomedical Image Segmentation*. MICCAI 2015.
2. Otsu, N. (1979). *A Threshold Selection Method from Gray-Level Histograms*. IEEE Transactions on Systems, Man, and Cybernetics, 9(1), 62–66.
3. Bai, S. et al. (2025). *Qwen2.5-VL Technical Report*.
4. Huang, L. et al. (2024). *A Survey on Hallucination in Large Language Models: Principles, Taxonomy, Challenges, and Open Questions*.
5. Dice, L. R. (1945). *Measures of the Amount of Ecologic Association Between Species*. Ecology, 26(3), 297–302.

---

# Academic Use

This repository was developed for academic coursework and demonstrates an experimental AI imaging analysis pipeline.

The reported results are intended for **educational and research purposes only** and should not be interpreted as clinical recommendations or evidence that the system is suitable for deployment in healthcare.

---

## Project Workflow

```text
EDA
 │
 ▼
U-Net Model Architecture
 │
 ▼
Model Evaluation
 │
 ▼
Hybrid AI Pipeline
 │
 ├── U-Net Segmentation
 │
 ├── Feature Extraction
 │
 ├── Numbers-First LLM
 │
 └── Direct VLM Analysis
 │
 ▼
Classical Post-Processing
 │
 └── Otsu Thresholding
 │
 ▼
Final Analysis & Comparison
```
