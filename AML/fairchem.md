# Fair Chem - Meta Open Materials 2024 (OMat24) Models in Azure Machine Learning

## Introduction

- How to run Meta Open Materials 2024 (OMat24) Models in Azure Machine Learning
- using CPU compute
- Running in python 3.12 Environment

```
conda create --name py312
conda activate py312
conda install python==3.12
conda install ipykernel
python -m ipykernel install --user --name py312 --display-name "Python (py312)"
```

## Step 1: Install the libraries

- First we need to install few packages to run the OMat24 models in Azure Machine Learning

```
%pip install fairchem-core
%pip install torch_geometric
%pip install torch_scatter
%pip install torch_sparse
```

## Step 2: Code

- Now lets display all the models

```
from fairchem.core.models.model_registry import available_pretrained_models
print(available_pretrained_models)
```

- output 

```
('CGCNN-S2EF-OC20-200k', 'CGCNN-S2EF-OC20-2M', 'CGCNN-S2EF-OC20-20M', 'CGCNN-S2EF-OC20-All', 'DimeNet-S2EF-OC20-200k', 'DimeNet-S2EF-OC20-2M', 'SchNet-S2EF-OC20-200k', 'SchNet-S2EF-OC20-2M', 'SchNet-S2EF-OC20-20M', 'SchNet-S2EF-OC20-All', 'DimeNet++-S2EF-OC20-200k', 'DimeNet++-S2EF-OC20-2M', 'DimeNet++-S2EF-OC20-20M', 'DimeNet++-S2EF-OC20-All', 'SpinConv-S2EF-OC20-2M', 'SpinConv-S2EF-OC20-All', 'GemNet-dT-S2EF-OC20-2M', 'GemNet-dT-S2EF-OC20-All', 'PaiNN-S2EF-OC20-All', 'GemNet-OC-S2EF-OC20-2M', 'GemNet-OC-S2EF-OC20-All', 'GemNet-OC-S2EF-OC20-All+MD', 'GemNet-OC-Large-S2EF-OC20-All+MD', 'SCN-S2EF-OC20-2M', 'SCN-t4-b2-S2EF-OC20-2M', 'SCN-S2EF-OC20-All+MD', 'eSCN-L4-M2-Lay12-S2EF-OC20-2M', 'eSCN-L6-M2-Lay12-S2EF-OC20-2M', 'eSCN-L6-M2-Lay12-S2EF-OC20-All+MD', 'eSCN-L6-M3-Lay20-S2EF-OC20-All+MD', 'EquiformerV2-83M-S2EF-OC20-2M', 'EquiformerV2-31M-S2EF-OC20-All+MD', 'EquiformerV2-153M-S2EF-OC20-All+MD', 'SchNet-S2EF-force-only-OC20-All', 'DimeNet++-force-only-OC20-All', 'DimeNet++-Large-S2EF-force-only-OC20-All', 'DimeNet++-S2EF-force-only-OC20-20M+Rattled', 'DimeNet++-S2EF-force-only-OC20-20M+MD', 'CGCNN-IS2RE-OC20-10k', 'CGCNN-IS2RE-OC20-100k', 'CGCNN-IS2RE-OC20-All', 'DimeNet-IS2RE-OC20-10k', 'DimeNet-IS2RE-OC20-100k', 'DimeNet-IS2RE-OC20-all', 'SchNet-IS2RE-OC20-10k', 'SchNet-IS2RE-OC20-100k', 'SchNet-IS2RE-OC20-All', 'DimeNet++-IS2RE-OC20-10k', 'DimeNet++-IS2RE-OC20-100k', 'DimeNet++-IS2RE-OC20-All', 'PaiNN-IS2RE-OC20-All', 'GemNet-dT-S2EFS-OC22', 'GemNet-OC-S2EFS-OC22', 'GemNet-OC-S2EFS-OC20+OC22', 'GemNet-OC-S2EFS-nsn-OC20+OC22', 'GemNet-OC-S2EFS-OC20->OC22', 'EquiformerV2-lE4-lF100-S2EFS-OC22', 'SchNet-S2EF-ODAC', 'DimeNet++-S2EF-ODAC', 'PaiNN-S2EF-ODAC', 'GemNet-OC-S2EF-ODAC', 'eSCN-S2EF-ODAC', 'EquiformerV2-S2EF-ODAC', 'EquiformerV2-Large-S2EF-ODAC', 'Gemnet-OC-IS2RE-ODAC', 'eSCN-IS2RE-ODAC', 'EquiformerV2-IS2RE-ODAC')
```

- now load the model
- Some models like Gemma are not available, when downloaded it will give error
- So i am choosing SchNet-S2EF-ODAC

```
from fairchem.core.models.model_registry import model_name_to_local_file

checkpoint_path = model_name_to_local_file('SchNet-S2EF-ODAC', local_cache='/tmp/fairchem_checkpoints/')
checkpoint_path
```

- Execute the model with data and display the output

```
from fairchem.core.common.relaxation.ase_utils import OCPCalculator
from ase.build import fcc111, add_adsorbate
from ase.optimize import BFGS
import matplotlib.pyplot as plt
from ase.visualize.plot import plot_atoms

# Define the model atomic system, a Pt(111) slab with an *O adsorbate!
slab = fcc111('Pt', size=(2, 2, 5), vacuum=10.0)
add_adsorbate(slab, 'O', height=1.2, position='fcc')

# Load the pre-trained checkpoint!
calc = OCPCalculator(checkpoint_path=checkpoint_path, cpu=False)
slab.set_calculator(calc)

# Run the optimization!
opt = BFGS(slab)
opt.run(fmax=0.05, steps=100)

# Visualize the result!
fig, axs = plt.subplots(1, 2)
plot_atoms(slab, axs[0]);
plot_atoms(slab, axs[1], rotation=('-90x'))
axs[0].set_axis_off()
axs[1].set_axis_off()
```

- output

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/fairchem-1.jpg 'RagChat')