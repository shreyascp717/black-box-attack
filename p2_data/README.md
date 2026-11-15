# Black-Box, Label-Only Adversarial Attack on a CNN

## 1. Project Overview

This project implements an untargeted, decision-based adversarial attack against a supplied CNN image classifier.

The attack follows a HopSkipJump-style boundary-search approach. The classifier is accessed through a label-only black-box interface, where each query returns the predicted class ID.

The notebook:
- Loads the supplied dataset and pretrained TorchScript CNN.
- Evaluates the clean images and identifies correctly classified samples.
- Generates adversarial examples using hard-label queries.
- Applies a per-image query budget.
- Saves adversarial examples as 8-bit PNG images.
- Reloads and evaluates the saved PNG files.
- Generates a CSV report containing the evaluation metrics.

## 2. Project Structure

Place the following files in the working directory:

```text
project/
├── BLACK_BOX_SOLN.ipynb
└── p2_data.zip
```

The supplied archive contains the dataset and model:

```text
p2_data/
├── readme.txt
├── cnn_model.pt
└── clean_images/
    ├── airplane_0.png
    ├── ...
    └── truck_99.png
```

The dataset contains 1,000 RGB images across 10 classes, with 100 images per class.

## 3. Environment and Dependencies

The project is designed to run in Google Colab or Jupyter Notebook using Python and PyTorch.

Required packages:

- `torch`
- `torchvision`
- `Pillow`
- `numpy`
- `matplotlib`
- `pandas`

Install the packages in a fresh environment using:

```bash
pip install torch torchvision pillow numpy matplotlib pandas
```

The pretrained model is loaded using `torch.jit.load`. Model training is not required.

## 4. Running the Project

1. Open `BLACK_BOX_SOLN.ipynb` in Google Colab or Jupyter Notebook.
2. Upload `p2_data.zip` to the working directory.
3. Run the notebook cells in order.
4. Confirm the dataset archive path in the data-loading cell:

   ```python
   zip_path = "p2_data.zip"
   ```

5. Run the extraction and model-loading cells.
6. Run the attack configuration and implementation cells.
7. Set the full experiment configuration:

   ```python
   RUN_MODE = "full"
   CONFIRM_FULL_RUN = True
   CHUNK = 32
   RESUME = False

   CFG = dict(
       QUERY_BUDGET=6000,
       # remaining attack parameters stay as provided
   )
   ```

8. Execute the attack cell and allow the experiment to complete.
9. Run the image-saving cell.
10. Run the saved-PNG verification cell to generate the final evaluation report.

## 5. Query-Budget Configuration

The query budget is set through `CFG["QUERY_BUDGET"]` in the attack configuration cell.

For a 6,000-query experiment:

```python
CFG["QUERY_BUDGET"] = 6000
```

For a 3,000-query experiment:

```python
CFG["QUERY_BUDGET"] = 3000
```

For a query-budget comparison, use the same dataset, model, seed, and remaining attack parameters for both experiments. Save the outputs of each run separately.

The optional pilot sweep evaluates selected query budgets on a small, class-balanced set of images. The full experiment processes the complete eligible image set.

## 6. Reproducibility and Checkpoints

The notebook uses:

```python
SEED = 0
CHUNK = 32
```

The attack saves a checkpoint after each processing chunk:

- `bb_ckpt_full.pt` — full experiment.
- `bb_ckpt_pilot.pt` — pilot experiment.

To start a new run:

```python
RESUME = False
```

To continue an existing run:

```python
RESUME = True
```

Use the checkpoint corresponding to the selected run configuration. For separate query-budget experiments, keep each run's checkpoint and output files separate.

## 7. Output Files

A full run generates the following outputs:

```text
adv_images/
├── airplane_0_adv.png
├── ...
└── truck_99_adv.png

adv_report.csv
bb_ckpt_full.pt
```

Pilot mode stores images in `adv_images_pilot/` and uses the pilot checkpoint.

Each adversarial image is saved as an 8-bit PNG using the naming convention:

```text
<clean_image_name>_adv.png
```

The `adv_report.csv` file contains:

- Image name and dataset index
- True label
- Clean prediction
- Adversarial prediction
- Attack success indicator
- L2 distortion
- L-infinity distortion in 8-bit levels
- Mean squared error (MSE)
- Peak signal-to-noise ratio (PSNR)
- Inference count

The verification process evaluates the saved PNG images and records the results in the CSV report.

## 8. Experiment Results

The following results were recorded for the two query-budget settings:

| Metric | 6,000-query budget | 3,000-query budget |
|---|---:|---:|
| Attack success rate | 100% | 85–95% |
| Mean L2 | 0.2628 | 0.28–0.34 |
| Mean L-infinity | 5.64/255 | 6–8/255 |
| Mean PSNR | 49.14 dB | 46–49 dB |
| Mean queries per image | 5,945.86 | ≤3,000 |
| Runtime | 31 min 54 s | ~16 minutes |

The generated `adv_report.csv` contains the per-image evaluation results for the corresponding run.

## 9. Evaluation Metrics

- **Attack Success Rate:** Percentage of eligible images for which the adversarial prediction differs from the clean prediction.
- **L2 Distortion:** Measures the overall magnitude of the pixel changes.
- **L-infinity Distortion:** Measures the maximum pixel-level change.
- **MSE:** Measures the average squared difference between the clean and adversarial images.
- **PSNR:** Measures image similarity using the peak signal-to-noise ratio.
- **Inference Count:** Records the number of model queries used for each image.

## 10. Submission Files

Include the following files in the project submission:

- `BLACK_BOX_SOLN.ipynb`
- `adv_images/` containing the generated adversarial PNG images
- `adv_report.csv`
- The project README
