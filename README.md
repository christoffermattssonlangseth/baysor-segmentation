# Baysor Segmentation Pipeline

Julia notebooks for running [Baysor](https://github.com/kharchenkolab/Baysor) cell segmentation on spatial transcriptomics data.

## Requirements

- **Julia 1.10.x** (not compatible with Julia 1.11)
- **Baysor** package
- **IJulia** for Jupyter notebook support
- **Python 3** + `scanpy`, `pandas`, `numpy` (for the h5ad export notebook)

### Installation

1. Install Julia 1.10:
```bash
curl -fsSL https://install.julialang.org | sh
```

2. Install Baysor:
```julia
using Pkg
Pkg.add(PackageSpec(url="https://github.com/kharchenkolab/Baysor.git"))
Pkg.build()
```

3. Install IJulia for notebook support:
```julia
using Pkg
Pkg.add("IJulia")
Pkg.add("DataFrames")
Pkg.add("CSV")
```

### Multi-threading Setup

Baysor benefits significantly from multi-threading. To run Julia with multiple threads, you can either:

1. **Create a custom Jupyter kernel** (recommended):
```julia
using IJulia
installkernel("Julia-16-threads", env=Dict("JULIA_NUM_THREADS"=>"16"))
```

2. **Set environment variable** before starting Julia:
```bash
JULIA_NUM_THREADS=16 julia
```

## Notebooks

### `run_baysor_batch.ipynb`

Batch processing notebook that:
- Searches a folder recursively for `*_cleaned.csv` transcript files
- Automatically skips files that already have completed segmentation
- Runs Baysor on all remaining files

**Features:**
- Looks for existing segmentation folders (`m50_s4/`, `baysor_m50_scale4/`, etc.)
- Outputs results to parameter-named folders (e.g., `m50_s4/segmentation.csv`)
- Progress tracking and error handling

### `baysor_analysis.ipynb`

Starter notebook for running Baysor on individual files with documentation of key parameters.

### `baysor_diagnostics.ipynb`

Summarizes Baysor outputs, counts, and basic QC across output folders.

### `running_baysor_slide_1.ipynb`

Example single-run notebook with hardcoded paths and parameters.

### `baysor_to_h5ad.ipynb`

Python notebook that scans two root folders for Baysor outputs with `m50` + `s4` or `m50` + `scale4`,
builds an expression matrix from `segmentation.csv`, attaches `segmentation_cell_stats.csv`, and writes
one `.h5ad` per output to `/Volumes/processing2/output_spinal_cord_injury`.

## Input Data Format

Baysor expects a CSV file with transcript coordinates. Required columns:
- `x`, `y` (and optionally `z`) - spatial coordinates
- `gene` - gene name

Example:
```csv
gene,global_x,global_y,global_z
Acta2,1234.5,5678.9,0.0
Pecam1,1235.1,5679.2,1.0
```

## Key Baysor Parameters

| Parameter | Description |
|-----------|-------------|
| `scale` | Expected cell radius in coordinate units (required) |
| `min_molecules_per_cell` | Minimum transcripts for a valid cell |
| `n_clusters` | Number of clusters for neighborhood composition |
| `x_column`, `y_column`, `z_column` | Column names for coordinates |
| `gene_column` | Column name for gene names |

## Data Cleaning

The batch notebook filters out non-gene entries matching these patterns:
- `Blank`, `Blank_1`, etc.
- `NegControl*`, `NegativeControl*`
- `Unassigned`, `None`
- `DeprecatedCodeword*`, `FalseCode*`

## Output

Baysor outputs several files:
- `segmentation.csv` - Transcripts with cell assignments
- `segmentation_cell_stats.csv` - Per-cell statistics
- `segmentation_counts.tsv` - Gene-by-cell count matrix

## License

MIT
