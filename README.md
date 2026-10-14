# GraphWater-Net

GraphWater-Net is a graph neural network framework for protein-ligand binding
affinity prediction with explicit water-network information. The model represents
protein atoms, ligand atoms, predicted water molecules, and their interactions as
topological graphs, and uses a Graphormer-style architecture to learn node, edge,
and spatial interaction features for affinity regression.

This repository contains the implementation of GraphWater-Net and several
baseline/comparison methods used in the accompanying paper.

## Paper

**Incorporating Water Molecules into Highly Accurate Binding Affinity Prediction
for Proteins and Ligands**

Diya Zhang, Qiaozhen Meng, and Fei Guo  
*International Journal of Molecular Sciences*, 2024, 25, 12676  
DOI: [10.3390/ijms252312676](https://doi.org/10.3390/ijms252312676)

If this code is useful for your research, please cite the paper:

```bibtex
@article{zhang2024graphwaternet,
  title   = {Incorporating Water Molecules into Highly Accurate Binding Affinity Prediction for Proteins and Ligands},
  author  = {Zhang, Diya and Meng, Qiaozhen and Guo, Fei},
  journal = {International Journal of Molecular Sciences},
  volume  = {25},
  number  = {23},
  pages   = {12676},
  year    = {2024},
  doi     = {10.3390/ijms252312676}
}
```

## Overview

Water molecules can mediate protein-ligand interactions through hydrogen-bond
bridges, desolvation effects, and solvent reorganization. However, many binding
affinity prediction methods use only protein and ligand structures and ignore the
water network near the binding pocket.

GraphWater-Net incorporates predicted hydration sites into the complex graph. In
the paper, hydration sites are predicted by two complementary approaches:

- **HydraMap**, a knowledge-based hydration-site prediction method.
- **3D-RISM**, a statistical-mechanics-based solvent distribution method.

The predicted water molecules are integrated with the protein-ligand complex,
converted into graph structures, and encoded by Graphormer layers. The extracted
features are used to predict binding affinity.

## Main Results

The model was trained on the PDBbind refined set 2016 and tested on CASF-2016.

| Model | Rp | RMSE |
| --- | ---: | ---: |
| GraphWater-Net | 0.868 | 1.27 |
| Without water network | 0.714 | 1.61 |
| GraphWater-Net without 3D-RISM | 0.809 | 1.28 |
| GraphWater-Net without HydraMap | 0.647 | 2.01 |

Compared with 17 representative scoring and deep-learning methods on CASF-2016,
GraphWater-Net achieved the highest Pearson correlation coefficient in the paper.

## Repository Structure

```text
.
├── model.py                    # Graphormer-style GraphWater-Net model
├── gen_hydramap.sh             # HydraMap-based water-site generation script
├── getwat                      # Utility executable used in water processing
├── PARA_LIG.TXT                # Ligand atom parameters
├── PARA_PRO.TXT                # Protein atom parameters
├── PARA_WAT.TXT                # Water atom parameters
├── PPS-ML.py                   # PPS-ML baseline implementation
├── HACNet.py                   # HAC-Net baseline implementation
├── AEScore/                    # AEScore comparison method
├── CGraphDTA/                  # CGraphDTA comparison method
├── DataDTA/                    # DataDTA comparison method
├── ECIFGraph HM-Holo-Apo/      # ECIFGraph comparison method
├── GIGN/                       # GIGN comparison method
├── GraphscoreDTA/              # GraphscoreDTA comparison method
├── SIGN/                       # SIGN comparison method
└── README.md
```

## Requirements

The code was developed for a Linux environment with GPU support. Typical Python
dependencies include:

- Python 3.8+
- PyTorch
- PyTorch Geometric
- PyTorch Lightning
- NumPy
- pandas
- SciPy
- scikit-learn
- h5py
- tqdm
- BioPandas
- OpenBabel / Pybel
- PyMOL, for visualization-related utilities

Example environment setup:

```bash
conda create -n graphwater python=3.8
conda activate graphwater

pip install numpy pandas scipy scikit-learn h5py tqdm pytorch-lightning biopandas
```

Install PyTorch and PyTorch Geometric according to your CUDA version. For
OpenBabel/Pybel and PyMOL, Conda installation is usually more reliable:

```bash
conda install -c conda-forge openbabel pymol-open-source
```

## Data

The experiments in the paper use:

- **Training set:** PDBbind refined set, version 2016
- **Test set:** CASF-2016 core set, 285 complexes

The expected data preparation includes:

1. Protein structures and ligand structures from PDBbind.
2. Binding affinity labels from PDBbind.
3. Hydration sites predicted by HydraMap and 3D-RISM.
4. Graph features for protein atoms, ligand atoms, and water molecules.
5. Baseline-specific input files for comparison methods.

Large datasets and generated features are not included in this repository. Please
download PDBbind/CASF data from the official sources and update script paths to
match your local directory layout.

## Method Summary

GraphWater-Net constructs graph representations for protein-ligand complexes:

- Nodes represent protein atoms, ligand atoms, and predicted water molecules.
- Edges represent covalent bonds or distance-based interactions.
- Protein-ligand, protein-water, ligand-water, and water-water interactions are
  encoded through graph edges.
- Graphormer centrality encoding, spatial encoding, and edge encoding are used to
  model structural relationships.
- Features from water-network graphs are concatenated and used for binding
  affinity regression.

In the reported optimal setting:

- Edge threshold: 6 Å
- Attention heads: 12
- Graphormer layers: 4

## Usage Notes

Several scripts in this repository use hard-coded paths from the original
experimental environment. Before running the code, check and update paths in the
corresponding scripts.

### Generate HydraMap Water Sites

The HydraMap workflow is started with:

```bash
sh gen_hydramap.sh apo
```

Make sure the PDBbind path and PDB list path inside `gen_hydramap.sh` match your
local environment.

### Build or Inspect the Model

The main GraphWater-Net architecture is implemented in:

```text
model.py
```

The model contains:

- Multi-head attention
- Graphormer encoder layers
- Node, edge, spatial-position, and degree encoders
- Graph-token based graph-level prediction

### Baseline Methods

The following folders contain comparison methods:

```text
AEScore/
CGraphDTA/
DataDTA/
ECIFGraph HM-Holo-Apo/
GIGN/
GraphscoreDTA/
SIGN/
```

Each baseline may have its own training or prediction script and may require its
own processed input format.

## Important Notes

- The repository contains research code and may require path adjustment before
  direct execution.
- Water-site prediction requires external tools such as HydraMap and 3D-RISM.
- PDBbind and CASF data are not distributed with this repository.
- Generated features, model checkpoints, and large intermediate files should not
  be committed to Git.

Recommended ignored files and folders:

```text
data/
outputs/
checkpoints/
*.pt
*.pth
*.pkl
*.h5
*.npy
*.npz
```

## License

The paper is published as an open-access article under the journal license.
Please check the repository license or contact the authors before reusing the
code for redistribution or commercial purposes.

## Contact

For questions about the paper or implementation, please refer to the author
information in the publication.
