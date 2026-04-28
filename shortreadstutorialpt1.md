## Tutorial for the Analysis of Shotgun Metagenomics (Short-Reads) Data for Environmental Microbiology
##### I wrote this tutorial for my fellow lab members (Sylvan lab and Smith lab) to follow along as we transition into analyzing more metagenomes in our work, and I hope that this will help them! :)  

##### The tutorial is made of several sections and after the assembly, you can follow the path of binning, or working with contigs without binning, or both.

## Part 1: Quality control on raw reads  
### Tools
For quality reports and visualization:
[fastqc](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)
[multiqc](https://github.com/MultiQC/MultiQC)
For read trimming and filtering:
[Fastp](https://github.com/opengene/fastp?tab=readme-ov-file#all-options)
or [BBDuk](https://bbmap.org/tools/bbduk)

After downloading and unzipping your reads, assess the quality of the reads with FastQC. I installed fastqc via conda/mamba. To run fastqc on several samples, just list your files, separated by a space. I set number of threads to 6 here. You can also use the -o flag to set your output directory.

<code> fastqc read_1_S1_R1_001.fastq.gz read_1_S1_R2_001.fastq.gz read_2_S1_R1_001.fastq.gz read_2_S1_R2_001.fastq.gz -t 6  

#or have it read all your files with the fastq.gz extension  

fastqc ~/*fastq.gz -o ~/output/directory/ -t 6  

#for further options such as adding a contaminant file, or using a different format for your input files, you can use use the help flag.  

fastqc --help
</code>

After your fastqc reports are generated, you can use Multiqc to create a combined, single, interactive report of all your fastqc reports. Be sure to have all your fastqc files in the same directory.

<code>multiqc .</code>

Use your FastQC report to guide your filtering and trimming parameters in Fastp. Here's a [resource](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/3%20Analysis%20Modules/) by  to help assess the FastQC reports. 

This is an example of my parameters in Fastp, but I would not blindly copy these. They might not apply exactly to your different samples. I trimmed 15 bp from the left (trim_front), and also had it quality-trimmed from the right (cut_right), based on how my reads looked in "Per base sequence content" in FastQC.  I also had overrepresented sequences and adapter content detected. You won't be able to address all the issues that FastQC flags (meaning sometimes you have to proceed even with warnings post-QC), but you can try your best. 

I also pair it with fastqc at the end to do a post-trimming QC
```fastp -i /data/raw/S01_R1.fastq.gz -I /data/raw/S01_R2.fastq.gz -o S01_trim_R1.fastq.gz -O S01_trim_R2.fastq.gz --detect_adapter_for_pe --overrepresentation_analysis --dedup --trim_front1 15 --trim_front2 15 --trim_poly_g --max_len1 150 --max_len2 150 --correction --cut_window_size 5 --cut_mean_quality 20 --cut_right --length_required 100 --thread 16 --html S01_trim.fastp.html --json S01_trim.fastp.json  && fastqc S01_trim_R1.fastq.gz S01_trim_R2.fastq.gz -t 16```

Alternatively, you could use BBDuk for trimming and filtering. Fastp worked wgreat for my basalt samples but it had issues with polyG tail removal in my current work with soil, so if you are running into the same issue, you can try BBDuk. I decompressed the fastq.gz files beforehand.

```bbduk.sh in1=S01_R1.fastq in2=S01_R2.fastq out1="bbduk/S01_trim_R1.fastq.gz" out2="bbduk/S01_trim_R2.fastq.gz" ftl=20 ftr2=5 trimpolyg=10 minlen=70 maq=15 threads=10```

Once your post-QC reads are good to go, you can proceed to assembly!