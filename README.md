# Knead For Speed
### Quality control of host-associated metagenomic data
Nothing fancy.  
Just a BASH wrapper script that does quality trimming/filtering of Illumina FASTQ reads and removes host DNA "contamination".  
If something doesn't work, or you want support for additional tools, then let me know by raising an issue.  

## Quick Links
[Dependencies](https://github.com/cazzlewazzle89/knead_for_speed#dependencies)  
[Installation](https://github.com/cazzlewazzle89/knead_for_speed#installation)  
[Host Database Download](https://github.com/cazzlewazzle89/knead_for_speed#host-database-download)  
[Usage](https://github.com/cazzlewazzle89/knead_for_speed#usage)  
[Note on Defaults](https://github.com/cazzlewazzle89/knead_for_speed#note-on-defaults)  
[Note on Host Removal](https://github.com/cazzlewazzle89/knead_for_speed#note-on-host-removal)

### Dependencies
| Software  | Version Tested |
| --- | --- |
| [BBTools](https://jgi.doe.gov/data-and-tools/software-tools/bbtools/) | 39.01 |
| [Bowtie2](https://github.com/BenLangmead/bowtie2) | 2.5.0  |
| [fastp](https://github.com/OpenGene/fastp) | 0.23.2 |
| [SAMTools](https://www.r-project.org/) | 1.16.1  |
| [sra-human-scrubber](https://github.com/shenwei356/seqkit) | 2.0.0 |
| [Trimmomatic](https://github.com/usadellab/Trimmomatic) | 0.39  |  

### Installation  
Conda/Mamba are definitely the easiest ways to install.  
The provided file `knead_for_speed.yml` contains all the info needed to recreate my conda env on a linux machine.  
Just clone the repo and use the command below to build the env.  
```bash
conda env create -f knead_for_speed/knead_for_speed.yml
```  

## Host Database Download
When you build the env, there will be a message from `sra-human-scrubber` describing how to install the database.  
I think it is just a single command but I can't remember if off the top of my head. Run it if you intend to use the tool at some point. I tested it briefly and didn't really like it, the version I used required uncompressed, interleaved FASTQ files, and since the reason for writing this script was to avoid the decompresseing and recompressing that I needed to do with the original kneaddata, it seemed pointless.  
It is included as an option though, using `reformat.sh` from `bbtools` to interleave and decompress input FASTQ, pipe to `sra-human-scrubber` for host removal, then pipe the output back through `reformat.sh` to deinterleave (is that a word?) and compress the output FASTQ.  
This issue might have been addressed by now, if it has, then ignore my ramblings.    

If you plan on always using bowtie2 for host DNA removal, then don't bother running that command and skip to the next paragraph.  
Or skip the next paragraph entirely if you are ahead of the game and have a bowtie2-formatted host genome index ready-to-go.  

You can download Bowtie2-formatted databases (indexes (indices?) - right hand side of the page) for most model organisms [here](https://bowtie-bio.sourceforge.net/bowtie2/index.shtml).  
They are ready to go after downloading and unzipping.  

For example, to use the latest (at time of writing) build of the human reference genome, you could do   
```bash
wget https://genome-idx.s3.amazonaws.com/bt/GRCh38_noalt_as.zip
unzip GRCh38_noalt_as.zip
```    

## Usage
```bash
knead_for_speed.sh -i manifest.tsv -o Kneaded/ -a fastp -c -c GRCh38_noalt_as/GRCh38_noalt_as -t 10

Options: 
-i Manifest File (headerless TSV file with sample name and absolute paths to demultiplexed forward and reverse reads) [default: manifest.tsv]  
-o Output Directory [default: Kneaded/]  
-a Trimming Algorithm [default: trimmomatic]  
	Currently Implemented: trimmomatic,fastp  
-f Host DNA Filtering Method [default: bt2_strict]  
    bt2_strict : Will remove read pairs if Bowtie2 aligns EITHER read host genome  
	bt2_lenient : Will remove read pairs if Bowtie2 aligns BOTH reads to host genome  
	srascrubber : Use ncbi::sra-human-scrubber instead of Bowtie2 - currently only works with human samples  
-c Host DNA Database (If Using Bowtie2)  
    Can specify full or relative path. Name of database must be included  
    eg. If using the example above, you would use `-c GRCh38_noalt_as/GRCh38_noalt_as`
-t Threads/CPUs To Use [default: 10]  
```

## Note on Defaults 
By default, `knead_for_speed` will use `trimmomatic` for read QC and `bowtie2_strict` for host removal.  
I don't use trimmomatic so will always specify `-a fastp` but I left it as the default as this script was designed to mimic the behaviour of `kneaddata` (without the `trf` step) while accepting compressed FASTQ files.  
I also don't understand how to add the option to change the trimmomatic settings by passing a commmand line argument, it always seems to chop off the last few characters.  
Fixing this is top of my todo list for this repo, but low down on my overall todo list.  
The options that trimmomatic currently uses are `MINLEN:60 ILLUMINACLIP:NexteraPE-PE.fa:2:30:10 SLIDINGWINDOW:4:20 MINLEN:50`  

## Note on Host Removal
`sra-human-scrubber` is designed to avoid uploading human genetic data to public repositories so is very conservative by nature, masking or removing anything that looks remotely like a human read.  The `bt2_strict` option in this wrapper is realtively strict, but can miss some host reads due to imprefect host reference genomes - so downstream microbiome profiling should always be perfromed with a marker gene-based appraoch (like [MetaPhlAn](https://github.com/biobakery/MetaPhlAn)) or using a database that contains also contains the host genome (like Kraken2's [pluspf](https://benlangmead.github.io/aws-indexes/k2))

# TO-DO  
Fix Trimmomatic options  
Make mouse database for sra-human-scrubber
