This is a small library and a bunch of clients to perform various operations on FASTQ files (such as merging partial overlaps, or quality filtering). For now it only works with,

* Illumina output files generated by CASAVA version 1.8.0 or higher (it will support earlier versions of CASAVA soon),
* Paired-end runs.

In this README file you will find information about the following items:

* Obtaining the source code. 
* Config file format library requires.
* Merging partially overlapping illumina pairs.
* Quality filtering script for "Complete Overlap" approach described by [Eren _et al_](http://www.plosone.org/article/info:doi/10.1371/journal.pone.0066643).
* Quality filtering script that uses the method described by [Minoche _et al_](http://genomebiology.com/2011/12/11/R112).
* Quality filtering script that uses the method described by [Bokulich _et al_](http://www.nature.com/nmeth/journal/v10/n1/full/nmeth.2276.html).

You can get in touch with me via `meren at mbl dot edu`.


# Contents

- [Contents](#contents)
- [Obtaining the Source Code](#obtaining-the-source-code)
    - [Requirements](#requirements)
- [Config File Format](#config-file-format)
    - [[general] section](#general-section)
    - [[files] section](#files-section)
    - [[prefixes] section](#prefixes-section)
- [Merging Partially Overlapping Illumina Pairs](#merging-partially-overlapping-illumina-pairs)
    - [Recovering high-quality reads from merged reads file](#recovering-high-quality-reads-from-merged-reads-file)
- [Quality Filtering](#quality-filtering)
    - ["Complete Overlap" analysis for V6](#complete-overlap-analysis-for-v6)
        - [Example STATS output](#example-stats-output)
    - [Minoche et al.](#minoche-et-al)
        - [Example STATS output](#example-stats-output)
        - [Example PNG files](#example-png-files)
    - [Bokulich et al.](#bokulich-et-al)
        - [Example STATS output:](#example-stats-output)
        - [Example PNG files](#example-png-files)
- [Questions?](#questions)


# Obtaining the Source Code

You can create a copy of the codebase by simply installing `git` and running this command in your terminal window:

     git clone git://github.com/meren/illumina-utils.git

This will generate the `illumina-utils` directory within the directory from which you run this command. Or you can simply download the zipped codebase through your browser and unzip it. Although it is not mandatory, having `git` installed and learning how to use it have advantages such as being able to keep your copy updated by synchronizing it with the master repository by simply typing `git pull` in your terminal window. However, if you do not wish to use `git`, you can always reach the zipped archive file of the latest version of the codebase via this link:

     https://github.com/meren/illumina-utils/archive/master.zip

Once you have the codebase, you should update your `PYTHONPATH` and `PATH` environment variables for easy access to the scripts and required libraries. You can do it by adding the following two lines in your `.bashrc` file (it is a hidden file in your home directory, you can reach it by opening a terminal window and typing "nano ~/.bashrc"):

     export PYTHONPATH=$PYTHONPATH:/path-to/illumina-utils/
     export PATH=$PATH:/path-to/illumina-utils/scripts

Please remember to replace _`path-to`_ place holder in the lines above with the actual path that points the where `illumina-utils` directory is on your file system.

## Requirements

In order to use this software package fully, you need following items available on your system:

- [matplotlib](http://matplotlib.org/) (required for visualizations)
- [python-Levenshtein](https://pypi.python.org/pypi/python-Levenshtein/) (required for fast merge of partially overlapping reads)
- [EMBOSS](http://emboss.sourceforge.net/) (`merger` is a part of EMBOSS software distribution and it is used for alignment of partially overlapping reads (when `--slow-merge` option is not used))
- [R](http://r-project.org) (required for visualizations today and will be used for statistical analyses)
    - [ggplot2](http://ggplot2.org/) (the R module that needs to be installed for R requirement)



# Config File Format

Clients under the [scripts directory](https://github.com/meren/illumina-utils/tree/master/scripts) require information to be passed along with the same config file format, if they require a config file as an input. Following is a config file template (there is also a [sample](https://github.com/meren/illumina-utils/blob/master/sample-files/general-config-SAMPLE.ini) file in the codebase):

    [general]
    project_name = project name
    researcher_email = meren@mbl.edu
    input_directory = test_input
    output_directory = test_output
    
    
    [files]
    pair_1 = pair_1_aaa, pair_1_aab, pair_1_aac, pair_1_aad, pair_1_aae, pair_1_aaf 
    pair_2 = pair_2_aaa, pair_2_aab, pair_2_aac, pair_2_aad, pair_2_aae, pair_2_aaf
    
    [prefixes]
    pair_1_prefix = ^....TACGCCCAGCAGC[C,T]GCGGTAA.
    pair_2_prefix = ^CCGTC[A,T]ATT[C,T].TTT[G,A]A.T


## [general] section

This is a mandatory section that contains `project_name`, `researcher_email`, `input_directory` and `output_directory` directives.

Two critical declerations in `[general]` section are `input_directory` and `output_directory`:

* `input_directory`: Full path to the directory where FASTQ files reside.
* `output_directory`: Full path to the directory where the output of the operation you will perform on this config to be stored. Since when it is Illumina we are dealing with huge files, the codebase is pretty conservative to protect users from making simple mistakes which may result in huge losses. So, if you don't create the `output_directory`, you will get an error (it will not be automatically generated). If there is already a file in the `output_directory` with the same name with one of the outputs, you will get an error (it will not be overwritten). `project_name` will be used as a prefix for the naming convention of output files, so it would be wise to choose something descriptive and UNIX-compatible.

## [files] section

`files` section is where you list your _file names_ to be found in the `output_directory`. Each file name has to be comma separated. The index of each file name in the comma seperated list, *must match* with its pair in the second list (see the example config file above).

## [prefixes] section

`prefixes` section is optional. If you have barcodes and primers in your reads, and you want them to be trimmed, you can use [regular expression](http://en.wikipedia.org/wiki/Regular_expressions)s to specify them. If prefixes are defined, results would contain only pairs that matched them.


# Merging Partially Overlapping Illumina Pairs

Pairs generated by paritally overlapping library preperation can be merged using [merge-illumina-pairs](https://github.com/meren/illumina-utils/blob/master/scripts/merge-illumina-pairs). Once you create your config file, you simply call it with the config file as a parameter.

By default, merging program uses Levenshtein distance to find the best merging strategy for two reads in a pair, starting from the minimum expected overlap (15 nt is the default, and can be changed through the appropriate command line parameter). Default merging strategy requires [Levenshtein module](http://code.google.com/p/pylevenshtein/) to be installed. Other merging alternative is using Needleman-Wunsch algorithm for the alignment of two reads in a pair. It can be done by declaring `--slow-merge` as a parameter. However it is, expectedly, *very slow*. Needleman-Wunsch alignment performs better only if there are a lot of insertion/deletion errors present in the dataset, which is very common in Roche/454 platform. In/del errors is not a common type of sequencing error for Illumina and there is no need to use Needleman-Wunsch for alignment for Illumina HiSeq or MiSeq runs. 

[merge-illumina-pairs](https://github.com/meren/illumina-utils/blob/master/scripts/merge-illumina-pairs) will create FASTA files for reads that were merged successfuly, or failed to merge. In the FASTA file for merged reads, the length of the overlapped region and the number of mismatches found in the overlapped part will be reported in the header line for each entry. The place of mismatch will be shown with capital letters in the sequences. Successful merge depends on the `o/r` value (which is also reported in the header). This value shows the ratio of the number of mismatches to the number of overlapping nucleotides. IF this value is less than 0.3, the pair will be put in the failed file. A successful merge does not necessarily indicate anything about the accuracy. But for maximum accuracy, you can use only 0 mismatches from the FASTA file that contains merged reads. Here is an example to show how the content of the merged file looks like:


     >D4ZHLFP1:26:C0WEFACXX:1:1101:3814:2456 1:N:0:TAAGGCGATAGATCGC|o:36|o/m:0.000000|mismatches:0
     acctgagactggatagcatataaagaaaaacagccacaattctgttggctgaaagattgggcatgtgcggagggcctctggctgcctccgctcttgaagaaagataaaggggagccggtttgcagagactgcatggcgagagaggaagcaagagaaagaggaggag
     >D4ZHLFP1:26:C0WEFACXX:1:1101:3992:2475 1:N:0:TAAGGCGATAGATCGC|o:27|o/m:0.037037|mismatches:1
     accaacagtttaagactggtttactcaaaacgaattcattgttttacatagctgaacacagaaatgataaataatgttctataaatatttgttggttgaCagctttggaacaatgtcctctagaaaaagctgaacgtagccacccaccttgtacgagagccgcaaagaatagcca
     >D4ZHLFP1:26:C0WEFACXX:1:1101:4113:2306 1:N:0:TAAGGCGATAGATCGC|o:36|o/m:0.000000|mismatches:0
     ctcctggggtcctgcccctcttttcagggccaggctcgtgggctctgggtgtctggaagtatgacactaagctggatgtgccagtcacacattagctgggggttctcagagcagcaaaacctgtgaaggcagaagctagaagcggactcaccctctctgctcaaag


If the program runs successfully, these files will appear in the `output_directory`:

* `project_name_MERGED` (successfuly merged reads)
* `project_name_FAILED` (failed reads)
* `project_name_FAILED_WITH_Ns` (reads with ambiguous bases)
* `project_name_MISMATCHES_BREAKDOWN` (number of mismatches breakdown)
* `project_name_STATS` (numbers regarding the run)

`project_name_MISMATCHES_BREAKDOWN` file can be visualized using the R script, [mismatches-distribution.R](https://github.com/meren/illumina-utils/blob/master/scripts/R/mismatches-distribution.R), included in the codebase (it will require ggplot2 to be available on the system). Here is an example:



![Example output](http://meren.org/tmp/breakdown.png)


When [merge-illumina-pairs](https://github.com/meren/illumina-utils/blob/master/scripts/merge-illumina-pairs) is run with `--compute-qual-dicts` it will also generate visualization of quality scores for different number of mismatch levels. Please see command line options for more information.

## Recovering high-quality reads from merged reads file

If [merge-illumina-pairs](https://github.com/meren/illumina-utils/blob/master/scripts/merge-illumina-pairs) finishes successfuly, it will generate `project_name_MERGED` for successfuly merged reads. But this file will contain any read that could have been merged. However, they are not necessarily the highest quality reads. They have various number of mismatches at the overlapped region. It is a very good practice to use merged reads with minimum number of mismatches at the overlapped region to eliminate most noise.

Program [filter-merged-reads](https://github.com/meren/illumina-utils/blob/master/scripts/filter-merged-reads) can be used to retain high-quality reads from `project_name_MERGED` file. To retain reads with 0 mismatches at the overlapped region you can simply run this command on your `project_name_MERGED` to generate a file with filtered reads `project_name_FILTERED`:

     filter-merged-reads project_name_MERGED --max-mismatches 0 --output project_name_FILTERED

Resulting file would be the file to use in downstream analyses.


# Quality Filtering

## "Complete Overlap" analysis for V6

[analyze-illumina-v6-overlaps](https://github.com/meren/illumina-utils/blob/master/scripts/analyze-illumina-v6-overlaps) can be used to generate very high quality short reads from sequences that were generated by a short insert size library preperation method. Library preperation method and the efficacy of the complete overlap analysis is described in [Eren _et al_](http://www.plosone.org/article/info:doi/10.1371/journal.pone.0066643). Once the manuscript is published, the reference will be available here. The output of the analyze-illumina-v6-overlaps script include these files:

* `project_name-STATS.txt` (an example output can be seen below)
* `project_name-PERFECT_reads.fa` (FASTA file for reads that passed the complete overlap analysis)
* `project_name-Q_DICT.cPickle.z` (gzipped cPickle object for Python that holds the machine reported quality scores for each group of failed and passed reads)
* `project_name-READ_IDs.cPickle.z` (gzipped cPickle object for Python that holds the read fate information)

If the program is run with `--visualize-quality-curves` option, following files will also be generated in the output directory:

* `project_name-PASSED.png` (visualization of mean machine reported quality scores per tile for pairs that passed the the complete overlap analysis)
* `project_name-FAILED_MISMATCH.png` (same for pairs that failed due to one or more mismatches at the region of read of interest)
* `project_name-FAILED_RP.png` (same for pairs that lacked a proper reverse primer)
* `project_name-FAILED_FP.png` (same for pairs that lacked a proper forward primer)

### Example STATS output

    $ cat 9022_B9-STATS.txt
    number of pairs                 : 828243
    total pairs passed              : 618589 (%74.69 of all pairs)
      perfect pairs with Ns         : 0 (%0.00 of perfect pairs)
      recovered ambiguous bases (p1): 0 (%0.00 of perfect pairs)
      recovered ambiguous bases (p2): 0 (%0.00 of perfect pairs)
    total pairs failed              : 209654 (%25.31 of all pairs)
      FP failed in both pairs       : 5086 (%2.43 of all failed pairs)
      FP failed only in pair 1      : 386 (%0.18 of all failed pairs)
      FP failed only in pair 2      : 116822 (%55.72 of all failed pairs)
      RP failed in both pairs       : 7076 (%3.38 of all failed pairs)
      RP failed only in pair 1      : 6767 (%3.23 of all failed pairs)
      RP failed only in pair 2      : 7647 (%3.65 of all failed pairs)
      FAILED_MISMATCH               : 65870 (%31.42 of all failed pairs)
      FAILED_RP                     : 21490 (%10.25 of all failed pairs)
      FAILED_FP                     : 122294 (%58.33 of all failed pairs)


## Minoche et al.

Quality filtering suggestions made by [Minoche _et al_](http://genomebiology.com/2011/12/11/R112) is implemented in [analyze-illumina-quality-minoche](https://github.com/meren/illumina-utils/blob/master/scripts/analyze-illumina-quality-minoche) script. The output of the scripts include these files:

* `project_name-STATS.txt` (file that contains all the numbers about quality filtering process, an example output can be seen below)
* `project_name-QUALITY_PASSED_R1.fa` (pair 1's that passed quality filtering)
* `project_name-QUALITY_PASSED_R2.fa` (matching pair 2's)
* `project_name-READ_IDs.cPickle.z` (gzipped cPickle object for Python that keeps the fate of read IDs, this file may be required by other scripts in the library for purposes such as visualization, or extracting a particular group of reads from the original FASTQ files)

If the program is run with `--visualize-quality-curves` option, these files will also be generated in the output directory:

* `project_name-PASSED.png` (visualization of mean quality scores per tile for pairs that passed the quality filtering)
* `project_name-FAILED_REASON_C33.png` (visualization of mean quality scores per tile for pairs that failed quality filtering due to C33 filtering (C33: less than 2/3 of bases were Q30 or higher in the first half of the read following the B-tail trimming))
* `project_name-FAILED_REASON_N.png` (same above, but for pairs that contained an ambiguous base after B-tail trimming)
* `project_name-FAILED_REASON_P.png` (same above, but for pairs that were too short after B-tail trimming)
* `project_name-Q_DICT.cPickle.z` (gzipped cPickle object for Python that holds mean quality scores for each group of reads)

### Example STATS output

    $ cat 9022_B9-STATS.txt
    number of pairs analyzed      : 122929
    total pairs passed            : 109041 (%88.70 of all pairs)
      total pair_1 trimmed        : 6476 (%5.94 of all passed pairs)
      total pair_2 trimmed        : 9059 (%8.31 of all passed pairs)
    total pairs failed            : 13888 (%11.30 of all pairs)
      pairs failed due to pair_1  : 815 (%5.87 of all failed pairs)
      pairs failed due to pair_2  : 12193 (%87.80 of all failed pairs)
      pairs failed due to both    : 880 (%6.34 of all failed pairs)
      FAILED_REASON_P             : 12223 (%88.01 of all failed pairs)
      FAILED_REASON_N             : 38 (%0.27 of all failed pairs)
      FAILED_REASON_C33           : 1627 (%11.72 of all failed pairs)

### Example PNG files

![Example output](http://meren.org/tmp/minoche.gif)

## Bokulich et al.

Quality filtering suggestions made by [Bokulich _et al_](http://www.nature.com/nmeth/journal/v10/n1/full/nmeth.2276.html) is implemented in [analyze-illumina-quality-bokulich](https://github.com/meren/illumina-utils/blob/master/scripts/analyze-illumina-quality-bokulich) script. The output of the scripts include these files:

* `project_name-STATS.txt`
* `project_name-QUALITY_PASSED_R1.fa`
* `project_name-QUALITY_PASSED_R2.fa`
* `project_name-READ_IDs.cPickle.z`

If the program is run with `--visualize-quality-curves` option, these files will also be generated in the output directory:

* `project_name-PASSED.png`
* `project_name-FAILED_REASON_P.png` (visualization of mean quality scores per tile for pairs that failed quality filtering for being too short after quality trimming)
* `project_name-FAILED_REASON_N.png` (same above, but having more ambiguous bases than `n` after quality trimming)
* `project_name-Q_DICT.cPickle.z`

### Example STATS output:

    number of pairs analyzed      : 122929
    total pairs passed            : 111598 (%90.78 of all pairs)
      total pair_1 trimmed        : 1994 (%1.79 of all passed pairs)
      total pair_2 trimmed        : 9227 (%8.27 of all passed pairs)
    total pairs failed            : 11331 (%9.22 of all pairs)
      pairs failed due to pair_1  : 738 (%6.51 of all failed pairs)
      pairs failed due to pair_2  : 10159 (%89.66 of all failed pairs)
      pairs failed due to both    : 434 (%3.83 of all failed pairs)
      FAILED_REASON_P             : 11299 (%99.72 of all failed pairs)
      FAILED_REASON_N             : 32 (%0.28 of all failed pairs)

### Example PNG files

![Example output](http://meren.org/tmp/bokulich.gif)


# Questions?

Please don't hesitate to get in touch with me via `meren at mbl dot edu`.
