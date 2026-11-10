# Geographic Classification

A machine-learning notebook for classifying geographic change from polygon geometry, image color statistics, and observations across five dates. Developed as a CentraleSupélec machine-learning course project, the notebook explores a nearest-neighbor baseline, LightGBM, and a random forest.

## Classification task

The target column is `change_type`, with the following encoding:

| Label | Class |
| --- | --- |
| 0 | Demolition |
| 1 | Road |
| 2 | Residential |
| 3 | Commercial |
| 4 | Industrial |
| 5 | Mega Projects |

## Workflow

1. Load training and test GeoJSON files from a dataset ZIP.
2. Train a three-neighbor baseline using polygon area.
3. Engineer geometry, RGB, date, status-transition, and geographic-context features.
4. Impute missing values and encode categorical features.
5. Explore class weighting and resampling with LightGBM.
6. Build an alternative chronological feature representation and train a 300-tree random forest.
7. Export test predictions as CSV files.

The LightGBM experiment evaluates the first split of a five-fold stratified splitter; it does not run all five folds.

## Repository contents

The implementation and saved notebook outputs are in [`Final Project ML.ipynb`](Final%20Project%20ML.ipynb). The training/test dataset is **not included**.

## Dataset requirements

Supply the course dataset archive, named `2-el-1730-machine-learning-project-2026.zip` in the notebook. It must contain `train.geojson` and `test.geojson`; the loader searches subdirectories after extraction.

The feature pipeline uses:

- `geometry`, with a defined coordinate reference system.
- `change_type` in the training data.
- `urban_type` and `geography_type`.
- `date0` through `date4`, formatted as `DD-MM-YYYY`.
- `change_status_date0` through `change_status_date4`.
- `img_red_mean_date1` through `img_red_mean_date5`, with equivalent green and blue columns.

Use the original dataset schema. The notebook also retains other available input features in parts of its preprocessing.

## Run in Google Colab

1. Upload `Final Project ML.ipynb` to Colab.
2. Place the dataset archive in Google Drive.
3. Run the Drive-mount cell and authorize access to your Drive.
4. Set `zipped_file_path` in the data-import cell. Its default is `/content/drive/MyDrive/2-el-1730-machine-learning-project-2026.zip`.
5. Install any missing dependencies in a notebook cell:

   ```python
   %pip install pandas geopandas numpy matplotlib seaborn scikit-learn lightgbm
   ```

6. Execute the notebook in order, reviewing the evaluation limitations below before interpreting the scores.

## Run locally

In a Python 3 virtual environment, install the notebook dependencies:

```sh
python -m pip install jupyterlab pandas geopandas numpy matplotlib seaborn scikit-learn lightgbm
jupyter lab "Final Project ML.ipynb"
```

Skip the `google.colab` import and Drive-mount cell. Change `zipped_file_path` to your local dataset ZIP. The notebook extracts data into `./extracted_data` relative to the working directory.

Package versions are not pinned, so environment compatibility may require adjustment.

## Outputs

| File | Contents |
| --- | --- |
| `knn_sample_submission.csv` | Baseline predictions |
| `Sample_submission.csv` | Random-forest test predictions |
| `array.npy` | Engineered random-forest training features |
| `testarray.npy` | Engineered random-forest test features |
| `train_y.csv` | Training labels for the alternative feature pipeline |

Prediction CSVs contain an `Id` index and a `change_type` prediction. Re-running export cells overwrites these files.

## Evaluation limitations

This notebook records exploratory experiments. Its saved scores should not be treated as unbiased estimates of performance on unseen data:

- The KNN accuracy and macro F1 are computed on its training data.
- LightGBM preprocessing computes imputation values before the validation split, allowing validation information into preprocessing.
- The final random-forest cell creates a train/validation split but fits on the **full** feature matrix, including validation rows. Fit on `X_train, y_train` for a held-out evaluation, then refit on all training data separately for submission.
- The final random-forest feature path computes area and length directly from the original geometry. Check its CRS before interpreting these measurements; the earlier LightGBM feature builder explicitly reprojects to EPSG:3857.
- The notebook identifies weak performance on the minority Industrial and Mega Projects classes. Review per-class metrics and macro F1 alongside accuracy.

Run from a fresh kernel when reproducing experiments because cells modify shared dataframes in place. No automated tests or packaged inference interface are included.
