# A pipeline for scRNA-seq and scATAC-seq analysis
---
Use this pipeline to analyse scRNA-seq and scATAC-seq datasets:
- Mapping of scRNA-seq data via [Cellranger](https://www.10xgenomics.com/support/software/cell-ranger/latest/tutorials/cr-tutorial-ct) and of scATAC-seq data via [Cellranger-atac](https://support.10xgenomics.com/single-cell-atac/software/pipelines/latest/using/count)
- Clustering, marker identification, cell type annotation and visualisation of scRNA-seq data via [Scanpy](https://scanpy.readthedocs.io/en/stable/) or [Seurat](https://satijalab.org/seurat/)
- Removal of scRNA-seq doublets with [DoubletDetection](https://github.com/JonathanShor/DoubletDetection?tab=readme-ov-file) (as part of Scanpy analysis) or [DoubletFinder](https://github.com/chris-mcginnis-ucsf/DoubletFinder) (as part of Seurat analysis)
- Sample integration in scRNA-seq and scATAC-seq data via [Harmony](https://github.com/immunogenomics/harmony)
- mRNA velocity analysis via [velocyto](http://velocyto.org/velocyto.py/tutorial/cli.html#run10x-run-on-10x-chromium-samples) and [scvelo](https://scvelo.readthedocs.io/en/stable/)
- Clustering, cell type label transfer and visualisation of scATAC-seq data via [Signac](https://stuartlab.org/signac/)


## Usage

The pipeline is designed to be run on a high performance cluster, via the Slurm scheduler. Modules loaded in the scripts that we used to run the analysis are:
R-base/4.3.0, python-base/3.11.3,  htslib /1.18, samtools/1.17, cellranger/7.2.0, cellranger-atac/2.1.0

To run the pipeline:

1. Clone the repository via:
```
    git clone https://github.com/michaelweinberger/scRNA-seq_and_scATAC-seq_analysis_pipeline.git
```
   
2. Adjust the `User defined variables` section of the **1_PARENT_script.sh** script:
### General
- `project`   Name for the project
- `script_dir`   Directory containing scripts copied from https://github.com/michaelweinberger/scRNA-seq_and_scATAC-seq_analysis_pipeline/scripts/
- `out_dir`   Directory containing all output, will be created if non-existent
- `species`   Name of species that sequencing data was generated in, one of "human", "mouse" or "zebrafish"

### scRNA-seq mapping
- `scRNA_mapping`   Indicates if scRNA-seq mapping via Cellranger should be run ("Yes" or "No")
- `sample_info`   File path to a tab-delimited text file, the first column needs to be named "sample_id" (sample-specific prefixes within the fastq filenames, for samples that have been re-sequenced supply sequence/re-sequence prefixes as a comma-separated list) and the second column needs to be named "fastq_dir" (directory containing the fastq files identified by "sample_id", for samples that have been re-sequenced and are stored in multiple directories supply all directories as a comma-separated list). Further metadata columns may be added.

### scRNA-seq doublet removal, clustering and cluster marker identification
- `scRNA_clustering`   Indicates if should be run ("Yes" or "No")
- `scRNA_analysis`   Indicates if Scanpy or Seurat should be used for analysis ("scanpy" or "seurat")
- `cellranger_out_dir`   File path to a Cellranger output directory containing "barcodes.tsv.gz", "features.tsv.gz" and "matrix.mtx.gz" files (normally this is path/to/outs/count/filtered_feature_bc_matrix). If the scRNA-seq mapping part of the pipeline has been run, this variable does not need to be supplied and the mapping outputs generated will automatically be used for analysis.
- `metadata`   File path to a tab-delimited text file containing cell metadata. The file needs to contain columns named "barcode" (cell barcodes matching those in the Cellranger output) and "sample_id" (Sample identifiers that will be used during doublet detection). If the scRNA-seq mapping part of the pipeline has been run, this variable does not need to be supplied and the mapping outputs generated will automatically be used for analysis ([out_dir]/cellranger/cellranger_aggr_cell_metadata.tsv).
- `min_genes`   Indicates the minimum number of genes detected for a cell to be kept in the dataset
- `max_genes`   Indicates the maximum number of genes detected for a cell to be kept in the dataset
- `max_perc_mt`   Indicates the maximum percentage of mitochondrial gene counts for a cell to be kept in the dataset
- `min_cells`   Indicates the minimum number of cells in which a gene needs to be detected to be kept in the dataset
- `n_pcs`   Indicates the number of principal components to use for neighbourhood graph
- `harmony_var`   Indicates the name of the metadata column to perform data integration on
- `leiden_res`   Indicates the clustering resolution to be used

### scRNA-seq cell type annotation
- `scRNA_annotation`   Indicates if scRNA-seq annotation should be run ("Yes" or "No")
- `cluster_anno`   File path to a .csv file, needs to contain columns named "cluster" (cell cluster numbers) and "cell_type" (cell type labels). Optionally, a column named "order" may be added (numbers indicating the order in which cell types should appear in UMAP plot legend)
- `scRNA_annotation_input`   File path to a clustered Scanpy (with file ending ".h5ad") or Seurat (with file ending ".rds") object. If supplying a Scanpy object, metadata need to contain a column named "leiden" (cell cluster numbers). If supplying a Seurat object, metadata need to contain a column named "seurat_clusters" (cell cluster numbers). If the scRNA-seq clustering part of the pipeline has been run, this variable does not need to be supplied and the clustered object generated will automatically be used for analysis.

### scRNA-seq mRNA velocity analysis
- `scRNA_velocity`   Indicates if scRNA-seq clustering should be run ("Yes" or "No")
- `scRNA_velocity_cellranger_dir`   File path to a directory containing Cellranger "path/to outs/possorted_genome_bam.bam" scRNA-seq mapping BAM files. If the scRNA-seq mapping part of the pipeline has been run, this variable does not need to be supplied and the mapping outputs generated will automatically be used for analysis.
- `scvelo_input`   File path to a clustered Scanpy (with file ending ".h5ad") or Seurat (with file ending ".rds") object. Cell barcodes in the object should match those in the Cellranger output BAM files supplied. Object metadata need to contain columns named "barcode" (cell barcodes) and "cell_type" (cell type labels). If the scRNA-seq annotation part of the pipeline has been run, this variable does not need to be supplied and the annotated object generated will automatically be used for analysis.

### scATAC-seq mapping
- `scATAC_mapping`   Indicates if scATAC-seq mapping via Cellranger-ATAC should be run ("Yes" or "No")
- `sample_info_ATAC`   File path to a tab-delimited text file, the first column needs to be named "sample_id" (sample-specific prefixes within the fastq filenames, for samples that have been re-sequenced supply sequence/re-sequence prefixes as a comma-separated list) and the second column needs to be named "fastq_dir" (directory containing the fastq files identified by "sample_id", for samples that have been re-sequenced and are stored in multiple directories supply all directories as a comma-separated list). Further metadata columns may be added.

### scATAC-seq clustering and annotation  
- `scATAC_clustering`   Indicates if scATAC-seq clustering should be run ("Yes" or "No")
- `fragments_file`   File path to a Cellranger-ATAC output "fragments.tsv.gz" file. If the scATAC-seq mapping part of the pipeline has been run, this variable does not need to be supplied and the mapping outputs generated will automatically be used for analysis.
- `metadata_ATAC`   File path to a tab-delimited file containing sample metadata. File needs to contain columns named "barcode" (cell barcodes) and "sample_id" (sample identifiers used for data integration). Further metadata columns may be added. If the scATAC-seq mapping part of the pipeline has been run, this variable does not need to be supplied and the mapping outputs generated will automatically be used for analysis.
- `min_frag`   Indicates the minimum number of fragments detected for a cell to be kept in the dataset
- `min_count`   Indicates the minimum number of counts for a cell to be kept in the dataset
- `max_count`   Indicates the maximum number of counts for a cell to be kept in the dataset
- `min_perc_peaks`   Indicates the minimum percentage of fragments located in peak regions for a cell to be kept in the dataset
- `nucleosome_signal`   Indicates the maximum nucleosome signal for a cell to be kept in the dataset
- `tss_enrichment`   Indicates the minimum TSS enrichment for a cell to be kept in the dataset
- `n_dims`   Indicates the number of dimensions to use for neighbourhood graph
- `harmony_var_ATAC`   Indicates the name of the metadata column to perform data integration on
- `leiden_res_ATAC`   Indicates the clustering resolution to be used
- `scRNA_path`   Optional file path to a cell type annotated Scanpy (file ending ".h5ad") or Seurat (file ending ".rds") scRNA-seq object, to be used for cell type label transfer to scATAC-seq data. Object metadata need to contain a column named "cell_type" (cell type labels). If you would like to use the output of the scRNA-seq analysis above for cell type label transfer, set `scRNA_path` to  ${out_dir}/scanpy/{project}_scRNAseq_no_doublets_annotated.h5ad if Scanpy analysis has been run, or to ${out_dir}/seurat/{project}_scRNAseq_no_doublets_annotated.rds if Seurat analysis has been run.

3. Finally, start the analysis via
```
sbatch 1_PARENT_script.sh
```
It might be best to first run the "scRNA_mapping" and "scRNA_clustering" parts of the pipeline, and then to create an annotation .csv file, based on the "[project]_markers" .csv (Scanpy analysis) or .xlsx (Seurat analysis) file and the "leiden" (Scanpy analysis) or "res" (Seurat analysis) UMAP plot. Then supply the annotation file as `cluster_anno` to the "scRNA_annotation" part, run scRNA-seq annotation, mRNA velocity analysis and scATAC-seq analysis.

