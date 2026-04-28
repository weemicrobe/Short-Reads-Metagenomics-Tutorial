# Tutorial for Short-Reads Shotgun Metagenomics Analysis - QC & Assembly

I wrote this tutorial for my fellow lab members (Sylvan lab and Smith lab) to follow along as we transition into analyzing more metagenomes in our work with environmental microbiology.  I hope that this will help you! :smile:   

###### *I've been writing code offline for a while and sharing them with my lab mates. I'm finally making them live on Github! This tutorial comprises several sections. After Part 2 on assembly, you can perform binning (tutorial to come), work with contigs without binning, or both.*  
###### *I always recommend [astrobiomike's](https://astrobiomike.github.io/genomics/) incredible site for beginners to bioinformatics and metagenomics! My endless thanks to Dr. Jordan Walker and Dr. Jessica Labonté for all their help in answering my many questions on metagenomics analysis!*

## Part 1: Quality control of raw reads  
### Tools
For quality reports and visualization:
[FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)
[MultiQC](https://github.com/MultiQC/MultiQC)  
For read trimming and filtering:
[Fastp](https://github.com/opengene/fastp?tab=readme-ov-file#all-options)
or [BBDuk](https://bbmap.org/tools/bbduk)

After downloading and unzipping your reads, assess the quality of the reads with FastQC. I installed these tools via Conda/Mamba. To run fastqc on several samples, just list your files, separated by a space. I set the number of threads to 6 here. You can also use the -o flag to set your output directory.

```
fastqc S01_R1.fastq.gz S01_R2.fastq.gz S02_R1.fastq.gz S02_R2.fastq.gz -t 6 
```

Or have it read all your files with the fastq.gz extension.  

```
fastqc ~/*fastq.gz -o ~/output/directory/ -t 6
```

For further options such as adding a contaminant file, or using a different format for your input files, you can use the help flag.  

```fastqc --help```

After your FastQC reports are generated, you can use MultiQC to create a combined, single, interactive report of all your FastQC reports. Be sure to have all your FastQC files in the same directory.

<code>multiqc .</code>

Use your FastQC report to guide your filtering and trimming parameters in Fastp. Here's a [resource](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/3%20Analysis%20Modules/) to help assess the FastQC reports. 

This is an example of my parameters in Fastp, but I would not blindly copy these. They might not apply exactly to your different samples. I trimmed 15 bp from the left (trim_front), and also had it quality-trimmed from the right (cut_right), based on how my reads looked in "Per base sequence content" in FastQC.  I also had overrepresented sequences and adapter content detected. You might not be able to address all the issues that FastQC flags (meaning sometimes you have to proceed even with warnings post-QC), but you can try your best to address most of them. 

I also paired it with FastQC at the end to do a post-trimming QC using the ```&&``` operator, though it is fine to do these steps separately if you have the tools in separate Conda/Mamba environments. 
```bash
fastp -i /data/raw/S01_R1.fastq.gz -I /data/raw/S01_R2.fastq.gz \
 -o S01_trim_R1.fastq.gz -O S01_trim_R2.fastq.gz \ 
 --detect_adapter_for_pe --overrepresentation_analysis --dedup --correction \
 --trim_front1 15 --trim_front2 15 --trim_poly_g --max_len1 150 --max_len2 150 \
 --cut_window_size 5 --cut_mean_quality 20 --cut_right --length_required 100 \
 --thread 16 --html S01_trim.fastp.html --json S01_trim.fastp.json &&
 fastqc S01_trim_R1.fastq.gz S01_trim_R2.fastq.gz -t 16
```

Alternatively, you could use [BBDuk](https://archive.jgi.doe.gov/data-and-tools/software-tools/bbtools/bb-tools-user-guide/installation-guide/) for trimming and filtering. Fastp worked great for my basalt samples, but it had issues with polyG tail removal in my current work with soil. If you are running into the same issue, you can try BBDuk. I decompressed the fastq.gz files beforehand here. Here is an example of how I used BBDuk. I wanted to trim 20 basepairs from the left, and 5 base pairs from the right in this example. 

```
bbduk.sh in1=S01_R1.fastq.gz in2=S01_R2.fastq.gz \
 out1=bbduk/S01_trim_R1.fastq.gz out2=bbduk/S01_trim_R2.fastq.gz \
 ftl=20 ftr2=5 trimpolyg=10 minlen=100 maq=20 threads=10
```

Once your post-QC reads are good to go, you can proceed to assembly!

## Part 2: Assembly of trimmed reads  
### Tool  
Short-read assembler: [MEGAHIT](https://github.com/voutcn/MEGAHIT)  

For those working on Grace (TAMU), MEGAHIT v1.2.9 should already be installed as a module.  If you are working on Discovery (USC), you will have to install MEGAHIT v1.2.9 with Conda. 

I work with microbially diverse and complex environmental samples, and co-assembly of samples, rather than individual sample assembly, have created better assemblies for these types samples so far. In my examples, I co-assemble multiple samples. 

#### MEGAHIT assembly presets:  
For my basalt samples, I added the `meta-sensitive` preset as I had very high diversity with low coverage, and the added k-mers from this preset did produce a better assembly with a higher N50.  
For my soil samples, I chose the `meta-large` preset as I was working with very large files of 34 soil samples!  
Remember to assemble on the _bigmem_ partition for Grace (TAMU), or _largemem_ partition for Discovery (USC). 

```
megahit -1 S01_trim_R1.fastq.gz,S02_trim_R1.fastq.gz,S03_trim_R1.fastq.gz,S04_trim_R1.fastq.gz,S05_trim_R1.fastq.gz,S06_trim_R1.fastq.gz,S07_trim_R1.fastq.gz,S08_trim_R1.fastq.gz \
-2 S01_trim_R2.fastq.gz,S02_trim_R2.fastq.gz,S03_trim_R2.fastq.gz,S04_trim_R2.fastq.gz,S05_trim_R2.fastq.gz,S06_trim_R2.fastq.gz,S07_trim_R2.fastq.gz,S08_trim_R2.fastq.gz \
-o eprcoassembly_metasens --preset meta-sensitive -t 32
```

You can also concatenate all your R1 files, and R2 files, and feed one of each file to MEGAHIT.
```
cat *trim_R1.fastq.gz > EPR_all8combined_trim_R1.fastq.gz
cat *trim_R2.fastq.gz > EPR_all8combined_trim_R2.fastq.gz

megahit -1 EPR_all8combined_trim_R1.fastq.gz -2 EPR_all8combined_trim_R2.fastq.gz -o eprcoassembly_metasens --preset meta-sensitive -t 32
```

For the assembly, you won't truly know how much memory you need until after the assembly is done. If you're not low on service units, I'd suggest requesting max memory and threads to be on the safe side. This might be overkill, but it's better than potentially wasting time/resources repeatedly assembling and failing.

To learn how much resources it took after the assembly and use this to better inform your next assembly:  
on Grace, use ```my job [jobID#]```  
on Discovery, use ```jobinfo [jobID#]```

If your assembly did not complete in the requested time, you can use the `--continue` flag on the next assembly, and MEGAHIT will pick up where it left off. Just be sure not to change any intermediate files generated by generated by MEGAHIT.

Use `tail` to view the last few lines of the log file generated by MEGAHIT.
```
cd eprcoassembly_metasens/
tail log

2023-07-27 18:18:14 - 7012829 contigs, total 3620058578 bp, min 200 bp, max 136273 bp, avg 516 bp, N50 492 bp
2023-07-27 18:18:17 - ALL DONE. Time elapsed: 11315.084732 seconds
```
The ideal N50 value differs by datasets, though I believe an N50 over 1,000 is generally great. Sometimes, that might be difficult to achieve if you're working with complex environments like basalt, or soil! I had gone through several (worse) assemblies before finally landing on this one, so I proceeded with this. 

I will plan to add a section here on using [SPAdes](https://github.com/ablab/spades) to co-assemble samples - stay tuned! :smile_cat:



