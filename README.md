# CellConnect

CellConnect is a Jupyter Notebook project for quality control and clustering analysis of the 10x Genomics PBMC 3K single-cell RNA sequencing dataset.

The workflow loads the dataset, filters low-quality cells, normalizes and log-transforms expression counts, identifies highly variable genes, reduces dimensionality with principal component analysis, builds a cell-cell nearest-neighbor graph, clusters cells with the Leiden algorithm, and displays the clusters in a UMAP embedding.

## Dataset

The notebook uses the PBMC 3K dataset from 10x Genomics.

- Initial dimensions: 2,700 cells x 32,738 genes
- AnnData cell metadata columns: none
- AnnData gene metadata column: `gene_ids`

The notebook loads `data/pbmc3k_raw.h5ad` when it is available. If the local file does not exist, it downloads the PBMC 3K dataset with Scanpy and saves it to that path as a gzip-compressed H5AD file.

## Workflow

### 1. Import environment

The notebook imports:

- `anndata`
- `matplotlib`
- `numpy`
- `pandas`
- `scanpy`
- `seaborn`

It sets NumPy's random seed to 42 and uses Scanpy verbosity level 2.

### 2. Quality control

The notebook calculates:

- Number of detected genes per cell
- Total RNA counts per cell
- Percentage of mitochondrial counts per cell

Mitochondrial genes are identified as gene names beginning with `MT-`.

Before filtering, the notebook creates violin plots for these three QC metrics.

Cells are retained when:

- Detected genes are between 200 and 2,500, inclusive
- Mitochondrial counts are less than 5%

The filtering step retains 2,638 of 2,700 cells, or 97.7%.

### 3. Normalization and highly variable genes

Remaining counts are:

- Normalized to 10,000 counts per cell
- Log-transformed with `log1p`
- Backed up in `adata.raw` before selecting highly variable genes

Highly variable genes are selected with:

- `min_mean=0.0125`
- `max_mean=3`
- `min_disp=0.5`

The notebook identifies 2,013 highly variable genes from 32,738 genes.

### 4. Principal component analysis

The notebook:

- Restricts the data to highly variable genes
- Scales genes with a maximum value of 10
- Computes up to 40 principal components using the ARPACK solver
- Reports the PCA coordinates and variance explained by the first two components

The PCA output in the notebook has 40 components for 2,638 cells.

### 5. Cell-cell nearest-neighbor graph

The notebook builds a neighbor graph from the PCA coordinates with:

- 10 neighbors
- 40 principal components
- Random seed 42

The resulting graph has dimensions of 2,638 x 2,638. The notebook reports 41,698 nonzero weighted connections, including both directions.

### 6. Leiden clustering

The Leiden algorithm is applied with:

- Resolution 0.5
- Random seed 42
- `leidenalg` flavor
- Cluster labels stored in `adata.obs["leiden"]`

The notebook identifies 6 Leiden clusters.

### 7. UMAP visualization

A UMAP 2D embedding is calculated from the neighbor graph and visualized with the Leiden cluster assignment.

UMAP is intended to preserve local neighborhood structure in a nonlinear two-dimensional representation.

## Outputs

The notebook saves the following figures in `results/`:

| File | Description |
|---|---|
| `01_quality_control.png` | Violin plots of QC metrics before filtering |
| `02_highly_variable_genes.png` | Highly variable gene selection plot |
| `03_umap_leiden_clusters.png` | UMAP visualization of Leiden clusters |
| `run_metadata.json` | Dataset dimensions, QC thresholds, HVG count, cluster count, neighbor count, and resolution |


## Running the Notebook

Run `CellConnect.ipynb` from the project root or from a folder named `notebooks`.

The notebook creates the `data/` and `results/` directories if they do not already exist.
