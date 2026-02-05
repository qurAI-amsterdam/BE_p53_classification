
# Detecting aberrant p53 immunohistochemical expression patterns in patients with Barrett’s esophagus using artificial intelligence

This project was conducted in collaboration with the Department of Pathology at Amsterdam UMC and resulted in an accepted publication in the Journal of Medical Imaging. We developed AI models to automatically classify p53 immunohistochemical expression patterns in Barrett’s esophagus biopsies, an important biomarker for dysplasia assessment and cancer risk stratification.

Using both full-biopsy CNNs and attention-based multiple-instance learning (CLAM) models with RetCCL features, we focused on detecting clinically relevant TP53 mutation patterns: wild-type, overexpression, null mutation, and double clone. To improve recognition of rare but high-risk patterns, we introduced a double-binary classification strategy and synthetic double-clone augmentation, leading to substantially improved performance on challenging mutation phenotypes.

![Figure 2 – p53 IHC expression patterns](figures/figure_1.png)
*Examples of p53 immunohistochemical expression patterns: wild-type, overexpression, null mutation, and double clone.*

![Research Summary](figures/figure_3.png)
*Overview of the data flow for the FB and CLAM-based models. In the FB approach (top), full biopsy images are classified using ResNet-18. In the CLAM approach (bottom), biopsies are split into 256×256 patches, embedded with RetCCL, and aggregated with CLAM attention Class predictions are either four-class or double-binary, depending on the output type.*

## Main files
- `vis_data.ipynb`: Visualize some aspects of the data like distribution
- `preprocess_latents.py`: Preprocess the png images into latents using RetCCL
- `resnet.py`: Train the ResNet model
- `pl_clam.py`: Train the CLAM model
- `vis_results.ipynb`: Load the trained models and test them on the test set and save them, then visualize the results in confusion matrices, ROC curves and such
- `vis_interpretability.ipynb`: Load the trained models and visualize the attention maps of the CLAM models and the other visualization methods for the ResNet models

## Model files
- `resnet.py`: Contains the ResNet model architecture implemented in PyTorch Lightning
- `pl_clam.py`: Contains the CLAM model architecture implemented in PyTorch Lightning
- `clam_model/clam_utils.py`: Contains a few utility functions for the CLAM model, including its attention nets
- `RetCCL/`: Contains the RetCCL architecture and the code to download its pretrained weights
- `load_data.py`: Contains various dataset classes, utils and transforms for loading the data
- `eval.py`: Contains the evaluation functions to use during PyTorch Lightning training

## Running the code
`requirements.txt` contains the necessary packages to run the code. However, I just used pip freeze -> requirements.txt, so it might contain more than necessary (sorry about that). Python 3.11.5 was used for this project.

### Only running to visualize results
In `minimum_data/results.zip` you can find the results of the trained models on the test set. Without any additional data, you can run the `vis_results.ipynb` notebook once you have the `results/` folder from the zip file set up at the path specified in the first notebook cell. This will load the predictions and generate the statistics and figures inside the notebook.

### Training the models
Before training, the code expects a data root directory `../../data` to be present, but this path can be changed at the top of `load_data.py`. In that root should be directories for each dataset (for example "LANS" and "BOLERO"). The main dataset (LANS in my case) should have the following structure:
- `biopsies/` containing the biopsy pngs, with names like "11_3.png" for the 11th slide and 3rd biopsy on that slide
- `train.csv` and `test.csv` containing the image names and labels like:
```
id,label
11_3,1
11_4,0
...
```
where 0 is Wild Type, 1 is Overexpression, 2 is Null Mutation and 3 is Double Clones.

For BOLERO, the `biopsies/` directory is also expected, but aside from that a labels file `P53_BOLERO_T.csv` with all pathologist ratings is expected in the form of:
```
Case ID,{pathologist id}, ... ,GS
{case id},{pathologist rating}, ... ,{gold standard label}
...
```
The CLAM models expect files with preprocessed latent vectors for each image (their path can be specified in the arguments of the `pl_clam.py` script) which can be generated using the `preprocess_latents.py` script.

### Folder structure
The folder structure can of course be changed by adjusting the paths in the code, but the following structure is expected by default:
```
code
├── {this repository, should be cwd when running scripts}
│   ├── {all files in this repository}
│
results (only necessary if visualizing existing results, can be found in minimum_data/results.zip)
├── CLAM
│   ├── {results for CLAM models .pt files}
│
├── {other model directories}
│
data (only necessary if training or evaluating models)
├── biopsies_s1.0_anon_new
│   ├── biopsies
│   │   ├── 0_0.png
│   │   ├── 0_1.png
│   │   ├── ...
│   │
│   ├── masks (optional)
│   │   ├── 0_0.png
│   │   ├── 0_1.png
│   │   ├── ...
│   │
│   ├── train.csv
│   ├── test.csv
│   ├── (latent vector files for CLAM models will be saved here)
│
├── BOLERO
│   ├── biopsies
│   │   ├── 0_0.png
│   │   ├── 0_1.png
│   │   ├── ...
│   │   
│   ├── P53_BOLERO_T.csv
│   ├── (latent vector files for CLAM models will be saved here)
│
models (only necessary if evaluating existing models)
├── CLAM
│   ├── {trained CLAM checkpoints}
│
├── {other model directories}
```
