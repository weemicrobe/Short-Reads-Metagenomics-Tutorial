# Tutorial for the Analysis of Shotgun Metagenomics (Short-Reads) Data for Environmental Microbiology
I wrote this tutorial for my fellow lab members (Sylvan lab and Smith lab) to follow along as we transition into analyzing more metagenomes in our work.  I hope that this will help you! :)  

##### The tutorial is made of several sections and after the assembly, you can follow the path of binning, or working with contigs without binning, or both.

## Part 1: Quality control on raw reads  
### Tools
For quality reports and visualization:
[FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)
[MultiQC](https://github.com/MultiQC/MultiQC)  
For read trimming and filtering:
[Fastp](https://github.com/opengene/fastp?tab=readme-ov-file#all-options)
or [BBDuk](https://bbmap.org/tools/bbduk)

After downloading and unzipping your reads, assess the quality of the reads with FastQC. I installed fastqc via conda/mamba. To run fastqc on several samples, just list your files, separated by a space. I set number of threads to 6 here. You can also use the -o flag to set your output directory.

```
fastqc S01_R1.fastq.gz S01_R2.fastq.gz S02_R1.fastq.gz S02_R2.fastq.gz -t 6 
```

Or have it read all your files with the fastq.gz extension  

```
fastqc ~/*fastq.gz -o ~/output/directory/ -t 6
```

For further options such as adding a contaminant file, or using a different format for your input files, you can use the help flag.  

```fastqc --help```

After your FastQC reports are generated, you can use MultiQC to create a combined, single, interactive report of all your FastQC reports. Be sure to have all your FastQC files in the same directory.

<code>multiqc .</code>

Use your FastQC report to guide your filtering and trimming parameters in Fastp. Here's a [resource](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/3%20Analysis%20Modules/) to help assess the FastQC reports. 

This is an example of my parameters in Fastp, but I would not blindly copy these. They might not apply exactly to your different samples. I trimmed 15 bp from the left (trim_front), and also had it quality-trimmed from the right (cut_right), based on how my reads looked in "Per base sequence content" in FastQC.  I also had overrepresented sequences and adapter content detected. You might not be able to address all the issues that FastQC flags (meaning sometimes you have to proceed even with warnings post-QC), but you can try your best to address most of them. 

I also pair it with fastqc at the end to do a post-trimming QC  
```bash
fastp -i /data/raw/S01_R1.fastq.gz -I /data/raw/S01_R2.fastq.gz -o S01_trim_R1.fastq.gz -O S01_trim_R2.fastq.gz --detect_adapter_for_pe --overrepresentation_analysis --dedup --trim_front1 15 --trim_front2 15 --trim_poly_g --max_len1 150 --max_len2 150 --correction --cut_window_size 5 --cut_mean_quality 20 --cut_right --length_required 100 --thread 16 --html S01_trim.fastp.html --json S01_trim.fastp.json && fastqc S01_trim_R1.fastq.gz S01_trim_R2.fastq.gz -t 16
```

Alternatively, you could use BBDuk for trimming and filtering. Fastp worked great for my basalt samples, but it had issues with polyG tail removal in my current work with soil. If you are running into the same issue, you can try BBDuk. I decompressed the fastq.gz files beforehand.

```
bbduk.sh in1=S01_R1.fastq in2=S01_R2.fastq out1="bbduk/S01_trim_R1.fastq.gz" out2="bbduk/S01_trim_R2.fastq.gz" ftl=20 ftr2=5 trimpolyg=10 minlen=70 maq=15 threads=10
```

Once your post-QC reads are good to go, you can proceed to assembly!
