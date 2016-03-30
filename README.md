## SICERpy: A friendly wrapper around the popular SICER peak caller

[SICER](http://home.gwu.edu/~wpeng/Software.htm) is a popular peak caller especially suitable for detection of broad ChIP-Seq marks. 
However, I found the original master script, `SICER.sh`, clunky to use. 

`SICER.py` is a re-write of `SICER.sh` aimed at improving the following aspects:

* **Friendly API** `SICER.py` takes command line arguments using the usual and convenient format `SICER.py --option arg`. In contrast, the original script
takes eleven (!) positional arguments. Also, each step of the pipeline is checked for clean exit so that if something goes wrong you now right away.

* **Parallel execution of multiple SICER runs** The original script caused temporary files from multiple instances of SICER to overwrite each other. 
`SICER.py` instead uses unique temporary directories so it can be run in parallel in the same directory from the same input files.

* **Less redundant output** By default, the only output produced is the table of candidate islands with their statistical significance.

* **Faster execution** Some steps of the pipeline have been parallelised.

## Requirements and Installation

Requirements are the same as the original `SICER`: python 2.6+ with `scipy` package. 

To install, first download the `SICERpy` directory and change permission of `SICER.py` to be executable: `chmod 755 SICER.py`.
Then use sicer in one of the following ways:

* *Best*: Create a symlink from `SICER.py` to a directory somewhere on your PATH and execute `SICER.py [options]`, e.g. 

```
ln -s /path/to/SICERpy/SICER.py ~/bin/
``` 

* Call using full path `/path/to/SICERpy/SICER.py [options]`
* Add the whole directory `SICERpy` to your PATH variable.

## Usage examples

These input files can be found in the `ex/` directory.

```
SICER.py -t ex/test.bed -c ex/control.bed -s hg19 -rt 0 > peaks.bed 2> sicer.log
```

Here, the table of candidate islands is sent to `peaks.bed`. The log is captured by stderr. 
Note that by setting the redundancy threshold to 0 we use directly the input files without further filtering. 
This is useful if the bed file have been already de-duplicated with external tools like picard/MarkDuplicates.

The output is in bed format with columns:

* chrom
* start
* end
* ChIP island read count,
* CONTROL island read count
* p value
* Fold change
* FDR threshold

The output file `peaks.bed` can be easily parsed to get the enriched regions. For example, the get regions with FDR < 0.01:

```
awk '$8 < 0.01' peaks.bed > peaks.01.bed
```

## Help on options

*The one shown here might be out of date*

```
SICER.py -h
usage: SICER.py [-h] --treatment TREATMENT --control CONTROL --species SPECIES
                [--effGenomeSize EFFGENOMESIZE] [--redThresh REDTHRESH]
                [--windowSize WINDOWSIZE] [--gapSize GAPSIZE]
                [--fragSize FRAGSIZE] [--keeptmp] [--version]

DESCRIPTION

Run the SICER pipeline using a control and an input file.
    
SEE ALSO:

https://github.com/dariober/SICERpy
    

optional arguments:
  -h, --help            show this help message and exit
  --treatment TREATMENT, -t TREATMENT
                        Treatment (pull-down) file in bed format
                                           
  --control CONTROL, -c CONTROL
                        Control (input) file in bed format
                                           
  --species SPECIES, -s SPECIES
                        Species to use. See or edit lib/GenomeData.py for available species. 
                                           
  --effGenomeSize EFFGENOMESIZE, -gs EFFGENOMESIZE
                        Effective Genome as fraction of the genome size. It depends on read length. Default 0.74.
                                           
  --redThresh REDTHRESH, -rt REDTHRESH
                        Redundancy threshold to keep reads mapping to the same position on the same strand. 
                        Set to 0 to skip filtering and use input bed as is. Default 1. 
                                           
  --windowSize WINDOWSIZE, -w WINDOWSIZE
                        Size of the windows to scan the genome. WINDOW_SIZE is the smallest possible island. Default 200.
                                           
  --gapSize GAPSIZE, -g GAPSIZE
                        Multiple of window size used to determine the gap size. Must be an integer. Default: 3.
                                           
  --fragSize FRAGSIZE, -f FRAGSIZE
                        Size of the sequenced fragment. The center of the the fragment will be taken as half the fragment size. Default 150.
                                           
  --keeptmp             For debugging: Do not delete temp directory at the end of run.
                                           
  --version             show program's version number and exit
```

## See also

* Original files: http://home.gwu.edu/~wpeng/Software.htm

Original papers describing SICER: 

* [Spatial Clustering for Identification of ChIP-Enriched Regions (SICER) to Map Regions of Histone Methylation Patterns in Embryonic Stem Cells](http://www.ncbi.nlm.nih.gov/pmc/articles/PMC4152844/)
* [A clustering approach for identification of enriched domains from histone modification ChIP-Seq data](http://bioinformatics.oxfordjournals.org/content/25/15/1952.full)

* Google group for SICER: https://groups.google.com/forum/#!forum/sicer-users
