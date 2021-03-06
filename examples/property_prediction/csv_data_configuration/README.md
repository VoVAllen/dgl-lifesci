# Molecular Property Prediction on a New CSV Dataset

The scripts in this directory are helpful for quickly prototyping a GNN-based model for molecular property 
prediction on a new CSV dataset.

## Data Preparation

For training, we assume that the molecular properties are recorded in a CSV file, where one column holds the SMILES strings 
of molecules and some other column(s) hold one or multiple molecular properties.

For inference, there are two options:
1) A `.csv`/`.csv.gz` file, where one column holds the SMILES strings of molecules
2) A `.txt` file, where each line corresponds to the SMILES string of a molecule

## Data Analysis (Optional)

We can do a quick analysis to molecules in a CSV file with 

```bash
python analysis.py -c X -sc Y
```

where `X` specifies the path to the CSV file and `Y` specifies the header for the SMILES column in the CSV file.

Other optional arguments include:
- **Number of Processes**: `-np processes` [default=1]
    - Specifies the number of processes to use for computing
- **Result Path**: `-p path` [default=analysis_results]
    - Specifies the path to save the analysis results
    
The analysis results will be saved to the following files in the result path above:
- `valid_canonical_smiles.txt`: Canonical SMILES for valid molecules
- `summary.txt`: A file of the analysis summarized, including
    - Number/percentage of valid molecules
    - Average number of atoms/bonds/rings per molecule
    - Number of molecules for a particular atom/bond descriptor value to appear

## Modeling

Currently we use a default setting as follows:
- Construct molecular graphs for each molecule, where nodes are atoms and edges are bonds
- Prepare initial atom features with 
[CanonicalAtomFeaturizer](https://lifesci.dgl.ai/generated/dgllife.utils.CanonicalAtomFeaturizer.html#dgllife.utils.CanonicalAtomFeaturizer).
- The dataset is split using scaffold split
- Perform early stopping and save the model that achieves the best validation performance
- (Optional) Automatically perform hyperparameter search using Bayesian optimization

## Regression

### Training

To train a model for predicting real-valued properties, we can use 

```bash
python regression_train.py -c X -sc Y
```

where `X` specifies the path to the CSV file and `Y` specifies the header for the SMILES column in the CSV file.

Other optional arguments include:
- **Model**: `-m model` [default=gcn]
    - Specifies the model to use. 
- **Number of Hyperparameter Search Trials**: `-ne num_trials` [default=None]
    - Specifies the number of trials for hyperparameter search. If not specified, we use the setting specified in 
    `model_configuration/{model_name}.json`
- **Task**: `-t task1,task2,task3,...`
    - Specifies the headers for task columns in the CSV file. If not specified, 
    we assume all columns are molecular properties except for the SMILES column.
- **Take the logarithm of the labels** `-lv` [default=False]
    - Whether to take logarithm of the labels for modeling
- **Split Ratio**: `-sr a,b,c` [default=0.8,0.1,0.1]
    - Specifies the proportion of the dataset to be used for training, validation and test. 
- **Evaluation Metric**: `-me metric` [default=r2]
    - Specifies the evaluation metric. 
    - By default we use Pearson correlation coefficient. Alternatively, we can use `mae` for mean absolute error, 
    and `rmse` for root mean square error.
- **Num Epochs**: `-n number` [default=1000]
    - Specifies the maximum number of epochs for training. Early stopping will be performed based on validation metric.
- **Print Every**: `-pe number` [default=20]
    - The training progress will be printed every `number` minibatches.
- **Result Path**: `-p path` [default=regression_results]
    - Specifies the path to save training results.

Once the training is completed, we can find the following files in the `Result Path` mentioned above:
- The trained model `model.pth`. If hyperparameter search is performed, this will be the best model found.
- The evaluation result `eval.txt`. If hyperparameter search is performed, this will be the result for the best 
hyperparameters found.
- The experiment configuration `configure.json`. If hyperparameter search is performed, this will be the 
best hyperparameters found, otherwise it will be just the setting specified in `model_configuration/{model_name}.json`.

### Inference

To use the model trained above for prediction on new molecules

```bash
python regression_inference.py -f X
```

where `X` specifies the path to a `.csv`/`.txt` file of SMILES strings

Other optional arguments include:
- **SMILES Column**: `-sc column` [default=None]
    - Specifies the column of SMILES strings in the input `.csv` file. Can be omitted if the input file is a 
      `.txt` file or the `.csv` file only has one column of SMILES strings
- **Train Result Path**: `-tp path` [default=regression_results]
    - Path to the training results saved, which will be used for loading the trained model and related configurations
- **Inference Result Path**: `-ip path` [default=regression_inference_results]
    - Path to the inference results, which will be used to save:
        - `prediction.csv`: A file of predicted properties associated with SMILES strings
- **Task**: `-t task1,task2,task3,...`
    - Task names for saving model predictions in the CSV file to output, which should be the same as the ones 
    used for training. If not specified, we will simply use task1, task2, ...

## (Multi-label) Binary Classification

### Training

To train a model for predicting binary labels, we can use 

```bash
python classification_train.py -c X -sc Y
```

where `X` specifies the path to the CSV file and `Y` specifies the header for the SMILES column in the CSV file.

Other optional arguments include:
- **Model**: `-m model` [default=gcn]
    - Specifies the model to use. 
- **Number of Hyperparameter Search Trials**: `-ne num_trials` [default=None]
    - Specifies the number of trials for hyperparameter search. If not specified, we use the setting specified in 
    `model_configuration/{model_name}.json`
- **Task**: `-t task1,task2,task3,...` 
    - Specifies the headers for task columns in the CSV file. If not specified, 
    we assume all columns are molecular properties except for the SMILES column.
- **Split Ratio**: `-sr a,b,c` [default=0.8,0.1,0.1]
    - Specifies the proportion of the dataset to be used for training, validation and test. 
- **Evaluation Metric**: `-me metric` [default=roc_auc_score]
    - Specifies the evaluation metric. 
    - By default we use ROC AUC score. 
- **Num Epochs**: `-n number` [default=1000]
    - Specifies the maximum number of epochs for training. Early stopping will be performed based on validation metric.
- **Print Every**: `-pe number` [default=20]
    - The training progress will be printed every `number` minibatches.
- **Result Path**: `-p path` [default=classification_results]
    - Specifies the path to save training results.

Once the training is completed, we can find the following files in the `Result Path` mentioned above:
- The trained model `model.pth`. If hyperparameter search is performed, this will be the best model found.
- The evaluation result `eval.txt`. If hyperparameter search is performed, this will be the result for the best 
hyperparameters found.
- The experiment configuration `configure.json`. If hyperparameter search is performed, this will be the 
best hyperparameters found, otherwise it will be just the setting specified in `model_configuration/{model_name}.json`.

### Inference

To use the model trained above for prediction on new molecules

```bash
python classification_inference.py -f X
```

where `X` specifies the path to a `.csv`/`.txt` file of SMILES strings

Other optional arguments include:
- **SMILES Column**: `-sc column` [default=None]
    - Specifies the column of SMILES strings in the input `.csv` file. Can be omitted if the input file is a 
      `.txt` file or the `.csv` file only has one column of SMILES strings
- **Train Result Path**: `-tp path` [default=classification_results]
    - Path to the training results saved, which will be used for loading the trained model and related configurations
- **Inference Result Path**: `-ip path` [default=classification_inference_results]
    - Path to the inference results, which will be used to save:
        - `prediction.csv`: A file of predicted properties associated with SMILES strings
- **Task**: `-t task1,task2,task3,...` [default=None]
    - Task names for saving model predictions in the CSV file to output, which should be the same as the ones 
    used for training. If not specified, we will simply use task1, task2, ...
- **Soft Classification**: `-s` [default=False]
    - By default we will perform hard classification with binary labels. 
    This flag allows performing soft classification instead.
