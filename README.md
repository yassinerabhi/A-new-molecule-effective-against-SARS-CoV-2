# A-new-molecule-effective-against-SARS-CoV-2

We combined a recurrent neural network generative model with transfer learning methods and active learning based algorithms to design novel small molecules capable of effectively inhibiting the 3CL protease in human cells. We then analyze these small molecules to find the correct binding site that matches the structure of the 3CL protease of our target virus as well as other analyses performed in this study. Based on these screening results, some molecules have achieved a good binding score close to -18 kcal/mol, which we can consider as good potential candidates for further synthesis and testing against SARS-CoV-2.

![final results PyRx screenshot](https://github.com/yassinerabhi/A-new-molecule-effective-against-SARS-CoV-2/blob/main/Flowchart%20of%20the%20strategy%20to%20identify%20candidate%20SARS-CoV-2%20drugs.JPG "Flowchart of the strategy to identify candidate SARS-CoV-2 drugs")

## Requirements

This model is built using Python 3.7, and utilizes the following packages;

* numpy 1.18.2
* tensorflow 2.1.0
* tqdm 4.43.0
* Bunch 1.0.1
* matplotlib 3.1.2
* RDKit 2019.09.3
* scikit-learn 0.22.2.post1

```console
!sudo apt install python-rdkit
!pip install numpy
!wget -c https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
!chmod +x Miniconda3-latest-Linux-x86_64.sh
!time bash ./Miniconda3-latest-Linux-x86_64.sh -b -f -p /usr/local
!time conda install -q -y -c conda-forge rdkit
!pip install bunch
%matplotlib inline
import matplotlib.pyplot as plt
import sys
import os
sys.path.append('/usr/local/lib/python3.7/site-packages/')
```

We strongly recommend to use the GPU version of tensorflow. Learning this model with all the data is very slow in CPU mode.
RDKit and matplotlib are used for SMILES cleaning, validation and visualization of molecules and their properties. To install RDKit, we strongly recommend to use Anaconda (See [this document](https://www.rdkit.org/docs/Install.html)). It is difficult to build RDKit from source.
Scikit-learn is used for PCA.

## Approach

According to the paper, the dataset was preprocessed and duplicates, salts, and stereochemical information were removed, SMILES strings with lengths from 34 to 74 (tokens). So I made SMILES clean up script. Run the following to get cleansed SMILES. It takes about 10 miniutes or more. Please wait.

The dataset was preprocessed and duplicates, salts and stereochemical information were removed. We then used the SMILES cleaning script.
```console
$ python cleanup_smiles.py datasets/dataset.smi datasets/dataset_cleansed.smi
```

### Train The model - 'Train the Network.ipynb'

The proposed model allowed us to generate about 25,000 new SMILES. It is possible to generate more than this to start with a larger set of molecules to evaluate before focusing on those that react well with the SARS-CoV-2 target, but the time factor was a major constraint in this outbreak since the generation process takes several hours.
In fact, the model was trained over 230 epochs, giving us a training accuracy of 99.86% and a validation accuracy of 99.63%. The model achieved 99.66% accuracy on a sample of test data.
Originally generated 25,000 smiles are saved to './generations/gen0_smiles.smi'


### Find the best candidates from the initial SMILES chemical universe - 'Refinement and evaluation.ipynb'

Once we generated ~25,000 new valid molecules, our biggest constraint was time: assessing the binding affinity of each molecule to the SARS-COV2 protease via PyRx is a time-consuming process, with ~1.5 molecules assessed per minute. Therefore, it was not possible to perform an analysis of 25,000 molecules, as this would have been time consuming.

To minimize this time, the initialize_generation_from_mols() function randomly selects 50 molecules, then iterates through the rest of the list by calculating the similarity scores of these molecules with the molecules added to the list so far, and only accepts the molecule if the maximum similarity score is below a certain threshold. This ensures that a small sample will present a diverse set of molecules.

We were then able to save the SMILES to a pandas data frame (csv), and also convert the SMILES to molecules and write those molecules to an SDF file that can be manually imported into PyRx for analysis. PyRx then produces a csv of molecules and their binding scores after analysis. In order to link the SMILES in our pandas/csv to the molecules as SDFs in PyRx, we used Chem.PropertyMol.PropertyMol(mol).setProp() to set the property to a unique four-letter identifier and a generation number.

### Evaluating Molecule <> SARS-COV2 - PyRx

We used PyRx to analyze the binding scores of the molecules with the target. We recommend this tutorial to get started:
[PyRX ligand docking tutorial](https://www.youtube.com/watch?v=2t12UlI6vuw)

### Transfer Learning & active learning - 'Refinement and evaluation.ipynb'

After evaluating approximately 1500 gen0 SMILES in PyRx, we obtained varied scores for a diverse set of molecules. We then used the techniques and principles of active learning and transfer learning to apply the original network's knowledge of realistic molecule generation to the field of generating molecules specifically tailored to SARS-COV2 binding.

For each generation, we performed the steps presented in the manuscript.

## Final evaluation - 'Last results.ipynb'

We took the best candidates we found (score <-14 and weight <900 daltons) and rerun them in PyRx to order the best binding score of each molecule, and the similarity of each molecule to existing HIV inhibitor drugs and the drug Remdesivir.

We can see that the model generated significantly higher binding scores than the existing drugs.

See './generations/Results_table_final.csv' for the full table of final results.
