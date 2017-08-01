# informME
An information-theoretic pipeline for methylation analysis of WGBS data.

LATEST RELEASE: v0.2.1

----------------------------------------------------------------
# DIRECTORY informME
----------------------------------------------------------------

This directory contains the MATLAB, C++ and R software that 
implements informME, as well as BASH submission scripts used 
to run the main cores of informME on a Sun Grid Engine (SGE) 
computer cluster. 

informME is an information-theoretic approach to methylation 
analysis developed in [1] and [2], using the Ising model of 
statistical physics. This method is applied on BAM files with 
reads aligned to a reference genome that are generated during 
a whole-genome bisulphite sequencing (WGBS) experiment, and 
produces genome-wide information about the statistical 
properties of methylation in a single-sample and in a 
differential analysis framework. The resulting information 
is stored in multiple MATLAB MAT files for subsequent 
processing and is summarized by bedGraph genomic tracks that 
can be visualized using a genome browser (such as the UCSC 
genome browser, see https://genome.ucsc.edu).  

The current implementation of informME has been tested within 
the following environment:

o Red Hat Enterprise Linux Server Release 6.5 (Santiago)

o Sun Grid Engine OGS/GE 2011.11

o MATLAB R2013b 64-bit with Bioinformatics and Symbolic Math Toolboxes 

and by using the following tools:

o Bismark Bisulphite Mapper - v0.13.1 (to make BAM files)

o SAMtools - v0.1.19 (to parse BAM files)

o gcc/g++ compiler - v4.4.7-4 (to compile MATLAB MEX code)


A. DIRECTORY STRUCTURE
----------------------

informME includes the following directories:

informME/SubmissionScripts - contains the scripts used for 
                             running the informME software on  
                             a SGE computer cluster

informME/MexSource         - contains the C++ portion of informME

informME/ParseBAMfile      - contains the MATLAB code that  
                             extracts genomic information from 
                             the reference genome and generates, 
                             from methylation reads available 
                             in BAM files, the input methylation 
                             data required by informME

informME/Modeling          - contains the MATLAB code that 
                             estimates the parameters of the 
                             Ising probability distribution,  
                             which models the stochastic 
                             methylation state within equally 
                             sized (in base pairs) 
                             non-overlapping regions of the 
                             genome

informME/SingleAnalysis    - contains the code that performs 
                             methylation analysis of individual 
                             samples 

informME/DiffAnalysis      - contains the code that performs 
                             differential methylation analysis 
                             between a test and a reference 
                             sample 

informME/BEDfiles          - used to store the output bedGraph 
                             files generated by informME

informME/PostProcess       - contains code that performs a 
                             number of post processing tasks 


B. DEPENDENCIES
---------------

Before using informME, the user must complete the following 
tasks:

1. Properly configure a MATLAB compiler (see 
   https://www.mathworks.com/products/compiler.html 
   for details.

2. Install the GNU MPFR C library for multiple-precision 
   floating-point computations with correct rounding (see 
   http://www.mpfr.org/). The current version of informME 
   is based on MPFR v3.1.2 and GMP v5.1.3, but newer 
   versions may work as well. The required mpreal.h file 
   can be found at the author's webpage:
   http://www.holoborodko.com/pavel/mpfr/

3. Install the EIGEN C++ template library for linear algebra 
   (see http://eigen.tuxfamily.org).  


C. INSTALLING InformME
----------------------

Installation of InformME requires the user to complete the 
following tasks:

1. Edit the submission script 

   runCompileMEX.sh 

   to specify IPATH, the path where the informME files are 
   located, MATLICE, the path of the MATLAB license file 
   [filename].lic, PATH1, the installation path of the EIGEN 
   package, and PATH2, the installation path of MPFR. 

2. Edit the submission scripts 

   runGenomeAnalysis.sh
   runDataMatrixGeneration.sh
   runModelEstimation.sh 
   runSingleMethAnalysis.sh
   runDiffMethAnalysis.sh

   to specify IPATH, the path where the informME files are 
   located, and MATLICE, the path of the MATLAB license file 
   [filename].lic. 

3. (Optional) Edit the submission scripts

   runSingleMethAnalysis.sh
   runDiffMethAnalysis.sh

   to modify the default (zero) values of ESIflag and MCflag, 
   depending on whether entropic sensitivity analysis (ESIflag=1) 
   or infomation-theoretic analysis of methylation channels 
   (MCflag=1) is desired.

4. Make all submission scripts executable by using the command 

   chmod u+x *.sh  

   within each subdirectory of informME/SubmissionScripts.

5. Within the directory 

   informME/SubmissionScripts/MexSource
 
   run the command

   ./runCompileMEX.sh

   This compiles and installs the C++ MATLAB MEX code required 
   by informME. The generated binary MEX files are automatically 
   placed in the informME/Modeling and informME/SingleAnalysis 
   directories. 


D. RUNNING informME
-------------------

Run the informME software using the following steps in the 
indicated order:

1. MOVE FILES TO BE ANALYZED

   Place within the directory 

   informME/ParseBAMfile/genome/speciesName

   the reference genome assembly FASTA file used for alignment. 
   For example, if the species under consideration is "Human" 
   and if the h19 assembly was used, then place within the 
   directory 

   informME/ParseBAMfile/genome/Human

   the file Homo_sapiens_assembly19.fasta, which can be dowloaded 
   from 

   http://archive.broadinstitute.org/ftp/pub/seq/
   references/Homo_sapiens_assembly19.fasta

   Subsequently, place within the directory 

   informME/ParseBAMfile/indexedBAMfiles/speciesName

   where speciesName is the name of the species under 
   consideration (e.g., Human), indexed BAM files (both .bam 
   and .bai) whose reads have been aligned to the previous 
   reference genome assembly FASTA file. These files are to 
   be processed by informME in order to produce information 
   about the methylation status of CpG sites in the genome. 
   
2. REFERENCE GENOME ANALYSIS

   Within the directory 

   informME/SubmissionScripts/ParseBAMfile
 
   run, ONLY ONCE per reference genome, the command 

   ./runGenomeAnalysis.sh "ReferenceAssembly.fasta" "spc" 

   where ReferenceAssembly.fasta is the FASTA file used for 
   alignment (e.g., Homo_sapiens_assembly19.fasta) and spc 
   indicates the corresponding species (e.g., Human). This 
   step analyzes the reference genome and produces results 
   that are organized within subdirectories of 
   informME/ParseBAMfile/genome for each species (Human, 
   Mouse, etc.), with each species folder including a MATLAB 
   MAT file CpGlocationChr#.mat for each chromosome. Each 
   MAT file contains the following information: 
   o location of CpG sites 
   o CpG density for each CpG site 
   o distance between neighboring CpG sites
   o location of the last CpG site in the chormosome
   o length of chromosome (in base pairs)

   NOTE: This step must be completed before proceeding with the 
         next step, but is only done once per reference genome. 

3. METHYLATION DATA MATRIX GENERATION

   Within the directory 

   informME/SubmissionScripts/ParseBAMfile

   run, for EACH BAM FILE, the command

   ./runDataMatrixGeneration.sh "BAMfileName" "spc" 
   "[trim1,trim2]"

   where BAMfileName is the name of the BAM file to be 
   processed (e.g., lungnormal for file lungnormal.bam), 
   spc is the name of the species (e.g., Human), trim1 is 
   the number of bases to be trimmed from the first read in 
   a read pair (e.g., 15), and trim2 is the number of bases 
   to be trimmed from the second read in the pair (e.g., 17). 

   MULTIPLE BAM files can be processed in parallel during this 
   step, depending on the number of computing nodes available 
   to the user. 

   This step generates methylation data matrices, which 
   are organized within subdirectories of 
   informME/ParseBAMfile/matrices  with a subdirectory for 
   each species (Human, Mouse, etc.), and with each species 
   folder including subdirectories for each chromosome (chr1, 
   chr2, etc.). Each chromosome subdirectory includes the 
   MATLAB file BAMfileName.mat for each BAM file processed. 
   Each file contains the following information for each 
   genomic region, which will be subsequently used in model 
   estimation: 
   o data matrix with -1,0,1 values for methylation status
   o CpG locations broken down by region.

   NOTE: This step must be completed before proceeding with 
         the next step. 

4. MODEL ESTIMATION

   Within the directory 

   informME/SubmissionScripts/Modeling

   run, for a set of BAM files associated with the SAME 
   phenotype, the command 

   ./runModelEstimation.sh "{'BAMfileName1','BAMfileName2',...}" "PhenoName" "spc"

   where BAMfileName1 is the name of the first BAM file in 
   the set, BAMfileName2 is the name of the second file, etc., 
   PhenoName is a unique name for the phenotype (e.g., 
   lungnormal-1), and spc is the name of the associated 
   species (e.g., Human). 

   MULTIPLE SETS OF PHENOTYPES can be processed in parallel 
   during this step depending on the number of computing nodes 
   available to the user. 

   This step estimates the parameters of the Ising probability 
   distribution used to model methylation within equally sized 
   (in base pairs) non-overlapping regions of the genome. 
   The results are organized within the directory 
   informME/Modeling/results with a subdirectory for each 
   species (Human, Mouse, etc.), and with each species folder 
   containing a subdirectory for each chromosome (chr1, chr2, 
   etc.). Each chromosome subdirectory contains the MATLAB 
   file phenoName.mat for each phenotypic methylation sample 
   (lungnormal-1, lungcancer-1, etc.), with each phenoName.mat 
   file containing the following information for each genomic 
   region used in model estimation: 
   o CpG distances
   o CpG densities
   o estimated alpha, beta, and gamma parameters of the Ising 
     model
   o initial and transition probabilities of the inhomogeneous 
     Markov chain representation of the Ising model
   o marginal probabilities at each CpG site
   o the log partition function of the estimated Ising model.

   NOTE: This step must be completed before proceeding with 
         the next step. 

5. SINGLE SAMPLE METHYLATION ANALYSIS

   Within the directory 

   informME/SubmissionScripts/SingleAnalysis

   run, for a SINGLE PHENOTYPE, the command

   ./runSingleMethAnalysis.sh "PhenoName" "spc"

   where PhenoName is the name of the phenotype (e.g., 
   lungnormal-1), as specified in STEP 4 above, and spc 
   is the name of the associated species (e.g., Human). 

   MULTIPLE PHENOTYPES can be analyzed in parallel during this 
   step depending on the number of computing nodes available to 
   the user. 

   This step performs methylation analysis of the given 
   phenotype by computing a number of statistical summaries of 
   the methylation state, including probability distributions 
   of methylation levels, mean methylation levels, and 
   normalized methylation entropies, as well as mean and 
   entropy based classifications. If desired, this step also 
   computes entropic sensitivity indices, as well 
   information-theoretic quantities associated with methylation 
   channels, such as turnover ratios, channel capacities, and 
   relative dissipated energies. The results are organized 
   within the directory informME/SingleAnalysis/results, with 
   a subdirectory for each species (Human, Mouse, etc.), and 
   with each species folder containing a subdirectory for each 
   chromosome (chr1, chr2, etc.). Each chromosome subdirectory 
   contains the MATLAB file phenoName_Analysis.mat for each 
   phenotypic methylation sample (lungnormal-1, lungcancer-1, 
   etc.), with each phenoName_Analysis.mat file containing 
   the following information for each genomic region used in 
   model estimation: 
   o The locations of the CpG sites within the genomic region.
   o Numbers of CpG sites within the analysis subregions. 
   o Which analysis subregions are modeled and which are not.
   o Estimated parameters of Ising model in genomic region
   o Methylation level probabilities in modeled subregions.
   o Coarse methylation level probabilities.
   o Mean methylation levels.
   o Normalized methylation entropies.
   o Entropic sensitivity indices (if ESIflag = 1).
   o Turnover ratios (if MCflag = 1).
   o Channel capacities (if MCflag = 1).
   o Relative dissipated energies (if MCflag = 1).

   NOTE: This step must be completed before proceeding with 
         the next step. 

6. DIFFERENTIAL METHYLATION ANALYSIS

   Within the directory 

   informME/SubmissionScripts/DiffAnalysis

   run, for a pair of test/reference phenotypes, the command

   ./runDiffMethAnalysis.sh "tPhenoName" "rPhenoName" "spc"

   where tPhenoName is the name of the test phenotype (e.g., 
   lungcancer-1), as specified in STEP 3 above, rPhenoName is 
   the name of the reference phenotype (e.g., lungnormal-1), 
   as specified in STEP 3, and spc is the name of the 
   associated species (e.g., Human). 

   MULTIPLE pairs of phenotypes can be processed in parallel 
   during this step depending on the number of computing nodes 
   available to the user. 

   This step performs differential methylation analysis between 
   a test and a reference phenotype by computing a number of 
   statistical summaries of the differential methylation state, 
   including differences in mean methylation levels and 
   normalized methylation entropies, Jensen-Shannon distances, 
   and differential mean-based and entropy-based classifications. 
   This step also optionally computes differences in entropic 
   sensitivity indices (if ESIflag = 1), as well differences 
   in information-theoretic quantities associated with 
   methylation channels (if MCflag = 1), such as channel 
   capacities and relative dissipated energies. 

7. POST-PROCESSING

   7.1. BED TO BW CONVERSION

        The user can use a provided utility to convert BED 
        files generated by informME to much smaller BigWig 
        (BW) files. 

        Within the directory 

        informME/PostProcess

        run the command

        ./bed2bw.sh "path" "asy"

        where "path" is the path containing the BED files and 
        "asy" is the assembly of the reference genome used to 
        generate the BED files (for example, "asy" must be set 
        to "hg19" when the Human assembly hg19 is used). The 
        resulting BW files within /path/BWfiles. 
  
        NOTE: For this utility, the following tools must be 
              installed on $PATH: bedtools, bedClip, 
              bedGraphToBigWig, and fetchChromSizes

   7.2. DMR DETECTION

        The user can use a provided utility to perform DMR
        detection using the Jensen-Shannon distance (JSD) 
        based on the method described in [2]. This utility 
        must be run within an R session.

        usage (when replicate reference data is available): 
 
        setwd("/path/to/informME/PostProcess/")
        source("jsDMR.R") 
        runReplicateDMR(refVrefFiles,testVrefFiles,
                        inFolder,outFolder)
   
        where refVrefFiles is a vector of BED file names that 
        contain the JSD values of all pairwise reference 
        comparisons, testVrefFiles is a vector of BED file 
        names that contain the JSD values of test/reference 
        comparisons, inFolder is the directory that contains 
        all JSD files, and outFolder is the directory used to 
        write the results. 
   
        usage (when no replicate reference data is available) 
   
        setwd("/path/to/informME/PostProcess/")
        source("jsDMR.R") 
        runNoReplicateDMR(JSDfile,inFolder,outFolder)
   
        where JSDfile is the name of a BED file that contains 
        the JSD values of a test/reference comparison, inFolder 
        is the directory that contains the JSD file, and 
        outFolder is the directory used to write the result.
   
        NOTE: For this utility, the following tools must be 
              installed in R: rtracklayer, logitnorm, mixtools.

   7.3. GENE RANKING

        The user can use a provided utility to rank all 
        Human genes in the Bioconductor library 
        TxDb.Hsapiens.UCSC.hg19.knownGene using the 
        Jensen-Shannon distance (JSD) based on the 
        method described in [2]. This utility must be 
        run within an R session.

        usage (when replicate reference data is available):

        setwd("path/to/informME/PostProcess/")
        source("jsGrank.R")
        rankGenes(refVrefFiles,testVrefFiles,inFolder,outFolder,
                  tName,rName)

        where refVrefFiles is a vector of BED files that contain 
        the JSD values of a test/reference comparison, 
        testVrefFiles is a vector of BED files that contain the 
        JSD values of available test/reference comparisons, 
        inFolder is the directory that contains the JSD files, 
        outFolder is the directory used to write the result 
        in an .xlsx file, and tName and rName are strings 
        providing names for the test and reference phenotypes.

        usage (when no replicate reference data is available):  

        setwd("path/to/informME/PostProcess/")
        source("jsGrank.R")
        rankGenes(c(),testVrefFiles,inFolder,outFolder,
                  tName,rName)

        where testVrefFiles is a vector of BED files that 
        contain the JSD values of available test/reference 
        comparisons, inFolder is the directory that contains 
        the JSD files, outFolder is the directory used to write 
        the result in an .xlsx file, and tName and rName are 
        strings providing names for the test and reference 
        phenotypes.
     
        NOTE: For this utility, the following tools must be 
              installed in R: GenomicFeatures, GenomicRanges, 
              Homo.sapiens, rtracklayer, 
              TxDb.Hsapiens.UCSC.hg19.knownGene, XLConnect.


E. OUTPUT FILES
---------------

informME generates the following methylation tracks in the form 
of BED files, which are written within the directory 
/informME/BEDfiles organized for each species (Human, Mouse, etc.):

SINGLE SAMPLE ANALYSIS

  o MML-PhenoName.bed
        mean methylation levels

  o NME-PhenoName.bed
        normalized methylation entropy

  o METH-PhenoName.bed
         methylation-based classification (non-variable)

  o VAR-PhenoName.bed
        methylation-based classification (variable)

  o ENTR-PhenoName.bed
         entropy-based classification

  o ESI-PhenoName.bed (if ESIflag = 1) 
        entropic sensitivity indices

  o TURN-PhenoName.bed (if MCflag = 1)
         turnover ratios

  o CAP-PhenoName.bed (if MCflag = 1) 
        channel capacities

  o RDE-PhenoName.bed (if MCflag = 1) 
        relative dissipated energies

DIFFERENTIAL ANALYSIS

  o dMML-tPhenoName-VS-rPhenoName.bed
         differences in mean methylation levels

  o dNME-tPhenoName-VS-rPhenoName.bed
         differences in normalized methylation entropies

  o DMU-tPhenoName-VS-rPhenoName.bed
        differential mean-based classification

  o DEU-tPhenoName-VS-rPhenoName.bed
        differential entropy-based classification

  o JSD-tPhenoName-VS-rPhenoName.bed
        Jensen-Shannon distances

  o dESI-tPhenoName-VS-rPhenoName.bed 
         (if ESIflag = 1) 
         differences in entropic sensitivity indices

  o dCAP-tPhenoName-VS-rPhenoName.bed 
         (if MCflag = 1) 
         differences in channel capacities

  o dRDE-tPhenoName-VS-rPhenoName.bed 
         (if MCflag = 1) 
         differences in relative dissipated energies

In addition, the following files are generated when using the 
provided utilities.

  o DMR-JSD-tPhenoName-VS-rPhenoName.bed
        differentially methylated regions
        (when the jsDMR.R utility is used)

  o gRank-JSD-tName-VS-rName.xlsx
        JSD based gene ranking when no replicate reference 
        data is available
        (when the jsGRank.R utility is used)

  o gRankRRD-JSD-tName-VS-rName.xlsx
        JSD based gene ranking when replicate reference 
        data is available
        (when the jsGRank.R utility is used)


# REFERENCES
------------

[1] Jenkinson, G., Pujadas, E., Goutsias, J., and Feinberg, A.P. 
    (2017), Potential energy landscapes indentify the 
    information-theoretic nature of the epigenome, Nature 
    Genetics, 49: 719-729.

[2] Jenkinson, G., Feinberg, A.P., and Goutsias, J. (2017) 
    An information-theoretic approach to the modeling and 
    analysis of whole-genome bisulfite sequencing data, 
    Submitted.



# VERSION HISTORY
-----------------

v0.2.1 -   	Added R utilities for DMR detection and gene ranking
            using the Jensen-Shannon distance (JSD). Various 
           	documentation improvements.

v0.2.0 - 	Code reorganized into more specialized directories,
	    	streamlined, and general SGE submission scripts
	    	provided as a guide. Updated README's. Variable
	    	names changed to reflect published notation.

v0.1.0 -	Initial release. Code widely tested internally. 
            Code used to create results for ref [1].



# LICENCING
-----------

All code authored by Garrett Jenkinson in informME is 
licensed under a GPLv3 license; exceptions to GPL 
licensing are the files contained in the following 
directories:

informME/Modeling/private

informME/SingleAnalysis/private

These files have their own licensing information in their 
headers. Thanks to Arnold Neumaier and Ali Mohammad-Djafari 
for their permissions to modify and distribute their 
software with informME.

