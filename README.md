scATAC-pro
=================

[![DOI](https://zenodo.org/badge/187909420.svg)](https://zenodo.org/badge/latestdoi/187909420)

A comprehensive workbench for single cell ATAC-seq data processing, analysis and visualization


   * [scATAC-pro](#scatac-pro)
      * [Workflow](#workflow)
      * [Updates](#updates)
      * [Installation](#installation)
      * [FAQs](#FAQs)
      * [Dependencies](#dependencies)
         * [Programming language users should install](#programming-language-users-should-install)
         * [Software packages required](#software-pacakges-required)
      * [Quick start guide](#quick-start-guide)
      * [Step by step guide to running scATAC-pro](#step-by-step-guide-to-running-scATAC-pro)
      * [Detailed usage](#detailed-usage)
      * [Run scATAC-pro through docker or singularity](#run-scATAC-pro-through-docker-or-singularity)
      * [Citation](#citation)


Workflow
--------

scATAC-pro consists of two units, the data processing unit and the downstream analysis unit. The data processing unit takes raw fastq files as input and outputs peak-by-cell count matrix, QC report and genome track files. It consists of the following modules: demultiplexing, adaptor trimming, read mapping, peak calling, cell calling, genome track file generation and quality control assessment. The downstream analysis unit consists of the following modules: dimension reduction, cell clustering, differential accessibility analysis, gene ontology analysis, TF motif enrichment analysis, TF footprinting analysis, linking regulatory DNA sequences with gene promoters, and integration of multiple datasets. We provide flexible options for most of analysis modules.


<p align="center">
  <img src="doc/fig1.png" width="480" title="">
</p>

Installation
------------

-   Note: It is not necessary to install scATAC-pro from scratch. You can use the docker or singularity version if you prefer (see [Run scATAC-pro through docker or singularity](#run-scATAC-pro-through-docker-or-singularity) )
-   Run the following command in your terminal, scATAC-pro will be installed in YOUR\_INSTALL\_PATH/scATAC-pro\_1.1.2

<!-- -->

    $ git clone https://github.com/wbaopaul/scATAC-pro.git
    $ cd scATAC-pro
    $ make configure prefix=YOUR_INSTALL_PATH
    $ make install
     
Updates
------------
- Current version: 1.1.2
- May, 2020
    * *integrate*: add VFACS (Variable Features Across ClusterS) option for the integration module,
      **which reselect variable features across cell clusters after an initial clustering, followed by 
        another round of dimension reduction and clustering**, specify *Integrate_by = VFACS* in configure file
    * *clustering*: filter peaks before clustering (accessible in less than 0.5% of cells) and
       remove rare peaks (accessible in less than 1% of cells) from the variable features list
    * *reConsMtx*: enable specifying a path for saving reconstructed matrix (optional)
- Complete update history can be viewd [here](complete_update_history.md)


FAQs
--------------
- [How to proceed using 10x cellranger-atac output?](https://github.com/wbaopaul/scATAC-pro/wiki/FAQs)
- [How to merge different peaks called from different data sets?](https://github.com/wbaopaul/scATAC-pro/wiki/FAQs)
- [How to reconstruct peak-by-cell matrix after updating peak file?](https://github.com/wbaopaul/scATAC-pro/wiki/FAQs)
- [How to access downstream analysis results in R?](https://htmlpreview.github.io/?https://github.com/wbaopaul/scATAC-pro/blob/master/doc/AccessResultsInR.html)


Dependencies
------------

### Programming language users should install

-   R (&gt;=3.6.1)
-   Python (&gt;=3.6.0)

### Software packages required

**The following packages will be automatically installed if NOT detected by the installation script.**

-   BWA (&gt;=0.7.17), bowtie, bowtie2
-   MACS2 (&gt;=2.2.5)
-   samtools (&gt;=1.9)
-   bedtools (&gt;=2.27.1)
-   deepTools (&gt;=3.2.1)
-   trim\_galore (&gt;=0.6.3), Trimmomatic (&gt;=0.6.3)
-   Regulratory Genomics Toolbox (RGT, for footprinting analysis, will ask whether you want to install it since the installation is done through conda, which takes a while and you may not want to conduct footprinting analysis)
-   g++ compiler, bzip2, ncurses-devel
-   R packaages: devtools, flexdashboard, png, data.table, Matirx, Rcpp, ggplot2, flexmix, optparse, magrittr, readr, Seurat, bedr, gridExtra, ggrepel, kableExtra, viridis, xlsx, RColorBrewer,pheatmap,motifmatchr, chromVAR, chromVARmotifs, SummarizedExperiment, BiocParallel, DESeq2, clusterProfiler, BSgenome.Hsapiens.UCSC.hg38, BSgenome.Mmusculus.UCSC.mm10, VisCello.atac

Quick start guide
-----------

-   **IMPORTANT**: The parameters and options should be specified in a configurartion file in plain text format. Copy and edit the configure\_user.txt file in this repository and then in your terminal run the following commands:

- **NOTE**: some mapping index and genome annotation files can be downloaded [rgtdata](https://chopri.box.com/s/dlqybg6agug46obiu3mhevofnq4vit4t) 

<!-- -->

    $ scATAC-pro -s process 
                 -i pe1_fastq,pe2_fastq,index_fastq 
                 -c configure_user.txt 

    $ scATAC-pro -s downstream 
                 -i output/filtered_matrix/PEAK_CALLER/CELL_CALLER/matrix.mtx 
                 -c configure_user.txt
    ## PEAK_CALLER and CELL_CALLER is specified in your configure_user.txt file

-   For data processing, if fastq files have been demultiplexed as the required format with the barcode recorded in the name of each read as @barcode:ORIGIN\_READ\_NAME , you can skip the demultiplexing step by running the following command:

<!-- -->

    $ scATAC-pro -s process_no_dex 
                 -i pe1_fastq,pe2_fastq
                 -c configure_user.txt 

-   The **output** will be saved under ./output as default
-   --verbose (or -b) will print the running message on screen, otherwise the message will only be saved under output/logs/MODULE.txt


Step by step guide to running scATAC-pro
---------------------------

-   **IMPORTANT**: you can run scATAC-pro sequentially. The input of a later analysis module is the output of the previous analysis modules. The following tutorial uses fastq files downloaded from [PBMC10k 10X Genomics](https://support.10xgenomics.com/single-cell-atac/datasets/1.1.0/atac_v1_pbmc_10k?) 

-   *Combine data from different sequencing lanes*

<!-- -->


    $ cat atac_pbmc_10k_v1_S1_L001_R1_001.fastq.gz atac_pbmc_10k_v1_S1_L002_R1_001.fastq.gz > pe1_fastq.gz

    $ cat atac_pbmc_10k_v1_S1_L001_R3_001.fastq.gz atac_pbmc_10k_v1_S1_L002_R3_001.fastq.gz > pe2_fastq.gz

    $ cat atac_pbmc_10k_v1_S1_L001_R2_001.fastq.gz atac_pbmc_10k_v1_S1_L002_R2_001.fastq.gz > index_fastq.gz

-   *Run scATAC-pro sequentially*

<!-- -->

    $ scATAC-pro -s demplx_fastq 
                 -i pe1_fastq.gz,pe2_fastq.gz,index_fastq.gz 
                 -c configure_user.txt 

    $ scATAC-pro -s trimming 
                 -i output/demplxed_fastq/demplxed_pe1_fastq.gz,
                    output/demplxed_fastq/demplxed_pe2_fastq.gz
                 -c configure_user.txt 


    $ scATAC-pro -s mapping 
                  -i output/trimmed_fastq/trimmed_pe1_fastq.gz,
                     output/trimmed_fastq/trimmed_pe2_fastq.gz 
                  -c configure_user.txt 

    $ scATAC-pro -s call_peak 
                 -i output/mapping_result/pbmc10k.positionsort.MAPQ30.bam
                 -c configure_user.txt 

    $ scATAC-pro -s aggr_signal 
                 -i output/mapping_result/pbmc10k.positionsort.MAPQ30.bam 
                 -c configure_user.txt 
                 
    $ scATAC-pro -s get_mtx 
                 -i output/summary/pbmc10k.fragments.txt,output/peaks/MACS2/pbmc10k_features_BlacklistRemoved.bed 
                 -c configure_user.txt 

    $ scATAC-pro -s qc_per_barcode 
                 -i output/summary/pbmc10k.fragments.txt,output/peaks/MACS2/pbmc10k_features_BlacklistRemoved.bed 
                 -c configure_user.txt

    $ scATAC-pro -s call_cell
                 -i output/raw_matrix/PEAK_CALLER/matrix.mtx
                 -c configure_user.txt
                 
    $ scATAC-pro -s get_bam4Cells
                 -i output/mapping_result/pbmc10k.positionsort.bam,
                    output/filtered_matrix/PEAK_CALLER/CELL_CALLER/barcodes.txt
                 -c configure_user.txt
    
    ## after running the above module, you can run module report (list below)
    ## to generate first page of the summary report

    $ scATAC-pro -s clustering
                 -i output/filtered_matrix/PEAK_CALLER/CELL_CALLER/matrix.mtx 
                 -c configure_user.txt

    $ scATAC-pro -s motif_analysis
                 -i output/filtered_matrix/PEAK_CALLER/CELL_CALLER/matrix.mtx 
                 -c configure_user.txt
                 
    $ scATAC-pro -s split_bam
                 -i output/downstream_analysis/PEAK_CALLER/CELL_CALLER/cell_cluster_table.txt
                 -c configure_user.txt

    $ scATAC-pro -s footprint ## supporting comparison two clusters, and one-vs-rest
                 -i 0,1  ## or '0,rest' (means cluster1 vs rest) or 'one,rest' (all one-vs-rest)
                 -c configure_user.txt
                 
    $ scATAC-pro -s runDA
                 -i 0:1:3,2  ## group1 consist of cluster 0,1,and 3; group2 cluster2 
                 -c configure_user.txt
                 
    $ scATAC-pro -s runGO
                 -i output/filtered_matrix/PEAK_CALLER/CELL_CALLER/differential_peak_cluster_table.txt 
                 -c configure_user.txt
      
                 
    $ scATAC-pro -s report
                 -i output/summary
                 -c configure_user.txt
                 
    ## merge peaks that are within 200bp distance of each other            
    $ scATAC-pro -s mergePeaks
                 -i peak_file1,peak_file2,(peak_file3...),200
                 -c configure_user.txt

    ## perform integrated analysis, assuming all data sets are processed by scATAC-pro
    ## which means each fragments.txt and barcodes.txt files can be found correspondingly            
    ## the integration methods includes 'VFACS', 'pool', 'seurat', and 'harmony', for instance, 
    ## you can specify the integration method with 'Integrate_by = VFACS' in the configure file
    $ scATAC-pro -s integrate
                 -i peak_file1,peak_file2,(peak_file3...)   ## 
                 -c configure_user.txt
    
    ## if you have the reconstructed matrix for data set (meaning using the merged peaks)
    ## you can run the *integrate_seu* whtich is second part of the module *integrate*            

    $ scATAC-pro -s integrate_seu
                 -i mtx_file1,mtx_file2,(mtx_file3...)   
                 -c configure_user.txt

- After clustering, user can interactively visualize and analyze the data with module *visualize* 

```
scATAC-pro -s visualize -i output/downstream_analysis/PEAK_CALLER/CELL_CALLER/VisCello_obj -c configure_user.txt

```
- Note that the visualization can also be done through R/Rstudio:

```
devtools::install_github("qinzhu/VisCello", ref="VisCello-atac") ## install the package 

library(VisCello.atac)

cello('output/downstream_analysis/PEAK_CALLER/CELL_CALLER/VisCello_obj') ## launch VisCello in your web browser with prepared data
```

- More details about the visualization module can be found at [VisCello](https://github.com/qinzhu/VisCello/tree/VisCello-atac)

Detailed Usage
--------------

    $ scATAC-pro --help
    usage : scATAC-pro -s STEP -i INPUT -c CONFIG [-o] [-h] [-v]
    Use option -h|--help for more information

    scATAC-pro 1.1.2
    ---------------
    OPTIONS

       [-s|--step ANALYSIS_STEP] : run an analysis module (or some combination of several modules) of the scATAC-pro workflow, supported modules include:
          demplx_fastq: perform demultiplexing
                               input: fastq files for both reads and index, separated by comma like:
                                      PE1_fastq,PE2_fastq,index1_fastq,inde2_fastq,index3_fastq...
                               output: Demultiplexed fastq1 and fastq2 files with index information embedded
                                       in the read name as:  @index3_index2_index1:original_read_name, saved in
                                       output/demplxed_fastq/ 
          trimming: trim read adapter
                               input: demultiplexed fastq1 and fastq2 files
                               output: trimmed demultiplexed fastq1 and fastq2 files, saved in output/trimmed_fastq/
          mapping: perform reads alignment
                             input: fastq files, separated by comma for each paired end
                             output: position sorted bam file saved in output/mapping_result, mapping qc stat and 
                                     fragment.txt files saved in output/summary
          call_peak: call peaks using aggregated data
                               input: BAM file, outputted from the mapping module
                               output: peaks in plain text format, saved as output/peaks/PEAK_CALLER/
                                       OUTPUT_PREFIX_features_Blacklist_Removed.bed
          get_mtx: build raw peak-by-cell matrix
                             input: fragment.txt file, outputted from the mapping module, and features/peak file, 
                                    outputted from the call_peak module, separated by a comma
                             output: sparse peak-by-cell count matrix in Matrix Market format, barcodes and feature files
                                     in plain text format, saved in output/raw_matrix/PEAK_CALLER/
          aggr_signal: generate aggregated signal, which can be uploaded to and viewed
                                 in genome browser
                                 input: BAM file, outputted from the mapping module
                                 output: Aggregated data in .bw and .bedgraph file, saved in output/signal/
          qc_per_barcode: generate quality control metrics for each barcode
                                    input: fragment.txt file (outputted from module mapping) and peak/feature file, 
                                           (outputted from module call_peak), separated by comma
                                     output: qc_per_barcode.txt file, saved in output/summary/
          call_cell: perform cell calling
                               input: raw peak-by-barcode matrix file, outputted from the get_mtx module
                               output: filtered peak-by-cell matrix in Market Matrix format, barcodes and features,
                                       saved in output/filtered_matrix/PEAK_CALLER/CELL_CALLER/
          get_bam4Cells: extract bam file for cell barcodes and calculate mapping stats correspondingly
                               input: A bam file for aggregated data outputted from mapping module and a barcodes.txt file
                                      outputted from module call_cell, separated by comma
                               output: A bam file saved in output/mapping_results and mapping stats (optional) saved
                                         in output/summary for cell barcodes                          
          process: processing data - including demplx_fastq, mapping, call_peak, get_mtx,
                                aggr_signal, qc_per_barcode, call_cell and get_bam4Cells
                                input: fastq files for both reads and index, separated by comma like:
                                       fastq1,fastq2,index_fastq1,index_fastq2, index_fastq3...; 
                                output: peak-by-cell matrix and all intermediate results 
          process_no_dex: processing data without demultiplexing
                                input: demultiplexed fastq files for both reads and index, separated by comma like:
                                       fastq1,fastq2; 
                                output: peak-by-cell matrix and all intermediate results 
          process_with_bam: processing from bam file
                                input: bam file for aggregated data, outputted from the mapping module 
                                output: filtered peak-by-cell matrix and all intermediate results 
          clustering: cell clustering
                               input: filtered peak-by-cell matrix file, outputted from the call_cell module
                               output: seurat objects with clustering label in the metadata (.rds file) and 
                                       barcodes with cluster labels (cell_cluster_table.txt file), and umap plot colorred
                                       clustering label, saved in output/downstream_analysiss/PEAK_CALLER/CELL_CALLER/
          motif_analysis: perform TF motif analysis
                               input: filtered peak-by-cell matrix file, outputted from the call_cell module
                               output: TF-by-cell enrichment matrix in chromVAR object, a table and heatmap indicating 
                                       TF enrichment for each cell cluster, saved in output/downstream_analysiss/
                                        PEAK_CALLER/CELL_CALLER/
          runDA: preform differential accessibility analysis
                           input: either two groups named as '0:1,2' in which group1 consists of cluster 0 and 1,
                                  and group2 consists of cluster2 or specified as '0,rest', or 'one,rest'
                           output: differential accessibility peaks in txt format saved in the same in 
                                   output/downstream_analysiss/PEAK_CALLER/CELL_CALLER/
          runGO: preform GO term enrichment analysis
                           input: differential accessible features file, outputted from runDA module (.txt file)
                           output: enriched GO terms in .xlsx format saved in the same directory as the input file
          runCicero: run cicero for calculating gene activity score and predicting cis chromatin interactions
                           input: seurat_obj.rds file outputted from the clustering module
                           output: cicero gene activity in .rds format and predicted interactions in .txt format, saved
                                   in output/downstream_analysiss/PEAK_CALLER/CELL_CALLER/
          split_bam: split bam file into different clusters
                               input: barcodes with cluster label (cell_cluster_table.txt file, outputted from 
                                      clustering module
                               output: .bam file (saved in output/downstream/PEAK_CALLER/CELL_CALLER/data_by_cluster), 
                                       .bw, .bedgraph (saved in output/signal/) file for each cluster
          footprint: perform TF footprinting analysis, supports comparison between two clusters and one cluster vs
                     the rest of cell clusters (one-vs-rest)
                               input: 0,1  ## or '0,rest' (means cluster1 vs rest) or 'one,rest' (all one-vs-rest)
                               output: footprinting summary statistics in tables and heatmap,
                                       saved in output/downstream/PEAK_CALLER/CELL_CALLER/
          downstream: perform all downstream analyses, including clustering, motif_analysis, 
                                split_bam (optional) and footprinting analysis (optional)
                                input: filtered peak-by-cell matrix file, outputted from call_cell module
                                output: all outputs from each module
          report: generate summary report in html file
                            input: directory to QC files, output/summary as default
                            output: summary report in html format, saved in output/summary and .eps figures for each panel
                                    saved in output/summary/Figures
          convert10xbam: convert bam file in 10x genomics format to bam file in scATAC-pro format 
                         input: bam file (position sorted) in 10x format
                         output: position sorted bam file in scATAC-pro format saved in output/mapping_result,
                                 mapping qc stat and fragment.txt files saved in output/summary/
          mergePeaks: merge peaks (called from different data sets) if the distance is
                            less than a given #basepairs (200 if not specified) 
                         input: peak files and a distance parameter separated by comma: 
                                peakFile1,peakFile2,peakFile3,200
                         output: merged peaks saved in file output/peaks/merged.bed
          reconstMtx: reconstruct peak-by-cell matrix given peak file, fragments.txt file, barcodes.txt and 
                      an optional path for reconstructed matrix 
                         input: different files separated by comma:
                                peakFilePath,fragmentFilePath,barcodesPath,reconstructMatrixPath
                         output: reconstructed peak-by-cell matrix saved in reconstructMatrixPath, 
                                 if reconstructMatrixPath not specified, a sub-folder reConstruct_matrix will be created
                                 under the same path as the input barcodes.txt file
          integrate: perform integration of two ore more data sets
                           input: peak/feature files, separated by comma: peak_file1,peak_file2
                           output: merged peaks, reconstructed matrix, integrated seurat obj and umap plot, saved in
                                   output/integrated/
          integrate_seu: perform integration of two ore more data sets given the reconstructed peak-by-cell matrix
                           input: mtx1,mtx2, separated by comma like, mtx_file1,mtx_file2
                           output: integrated seurat obj and umap plot, saved in output/integrated/
          visualize: interactively visualize the data through VisCello
                         input: VisCello_obj directory, outputted from the clustering module
                         output: launch VisCello through web browser for interactively visualization"

       -i|--input INPUT : input data, different types of input data are required for different analysis
       -c|--conf CONFIG : configuration file for parameters (if exist) for each analysis module
       [-o|--output_dir : folder to save results, default output/ under the current directory; sub-folder will be created automatically for each analysis
       [-h|--help]: print help infromation on screen
       [-v|--version]: display current version numbe of scATAC-pro on screen
       [-b|--verbose]: print running message on screen




Run scATAC-pro through docker or singularity
----------------------------------
In case you have problem in installing dependencies, you can run scATAC-pro without installing dependencies in **one of** the following ways:

1. Run the pre-built dockerized version, pull the docker image [here](https://hub.docker.com/r/wbaopaul/scatac-pro)

2. Run it through singularity (which is more friendly with high performance cluster or HPC, and linux server) by running the following command:

```
$ singularity pull -F docker://wbaopaul/scatac-pro 
## will generate scatac-pro_latest.sif in current directory

$ singularity exec -H YOUR_WORK_DIR --cleanenv scatac-pro_latest.sif scATAC-pro -s XXX -i XXX -c XXX

```

3. To use it on HPC cluster:

```
# write a script mapping.sh for mapping as an example:
#!/bin/bash
module load singularity

singularity pull -F docker://wbaopaul/scatac-pro  ## you just need run this line once
## will generate scatac-pro_latest.sif in the current directory

singularity exec --cleanenv -H /mnt/isilon/tan_lab/yuw1/run_scATAC-pro/PBMC10k scatac-pro_latest.sif \ 
scATAC-pro -s mapping -i fastq_file1,fastq_file2 -c configure_user.txt

# and then qsub mapping.sh
```

- **NOTE**: YOUR_WORK_DIR is your working directory, where the outputs will be saved and all data under YOUR_WORK_DIR will be available to scATAC-pro

- **NOTE**: all inputs including data paths specified in configure_user.txt should be available under YOUR_WORK_DIR

- **NOTE**: if running the *footprint* module, remember to download the reference data [rgtdata](https://chopri.box.com/s/dlqybg6agug46obiu3mhevofnq4vit4t) folder and put it under YOUR_WROK_DIR





Citation
--------------------------------------
Yu W, Uzun Y, Zhu Q, Chen C, Tan K. [*scATAC-pro: a comprehensive workbench for single-cell chromatin accessibility sequencing data.*](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-020-02008-0) Genome Biology; 2020 
