Lab 7: Transcriptome assembly
---

--

During this lab, we will acquaint ourselves with de novo transcriptome assembly using Trinity. You will:

1. Install software and download data

2. Error correct, quality and adapter trim data sets.

3. Apply digital normalization to the dataset.

4. Trinity assembly

5. Because the above steps will take a few hours, I am providing you with 2 datasets: one is the 10 million read dataset you used last week. The other is that same 10M read dataset that I have error corrected, quality/adapter trimmed, normalized, and subsampled to 0.5 million reads (I did this so that the assembly could be done in a reasonable amount of time). Especially for the people who are going to do <em>de novo</em> transcriptome projects, and students who will use something like this in their own research, that it is probably worth going through the whole pipeline at some point.

-

The JellyFish manual: <a href="ftp://ftp.genome.umd.edu/pub/jellyfish/JellyfishUserGuide.pdf">ftp://ftp.genome.umd.edu/pub/jellyfish/JellyfishUserGuide.pdf</a>

The Khmer manual: <a href="http://khmer.readthedocs.org/en/v1.1/">http://khmer.readthedocs.org/en/v1.1/</a>

Trinity reference material: <a href="http://trinityrnaseq.sourceforge.net/">http://trinityrnaseq.sourceforge.net/</a>

---

> Step 1: Launch and AMI. For this exercise, we will use a <span style="color: #ff0000;"><strong>m3.2xlarge</strong></span> (note different instance type). Remember to change the permission of your key code `chmod 400 ~/Downloads/????.pem` (change ????.pem to whatever you named it)


	ssh -i ~/Downloads/?????.pem ubuntu@ec2-???-???-???-???.compute-1.amazonaws.com


---

> Update Software


	sudo bash
	apt-get update


---

> Install updates


	apt-get -y upgrade


---

> Install other software


	apt-get -y install subversion tmux git curl bowtie libncurses5-dev samtools gcc make g++ python-dev unzip dh-autoreconf default-jre python-pip zlib1g-dev


---

> Install Lighter, software for error correction.


    cd $HOME
    git clone https://github.com/mourisl/Lighter.git
    make -j8
    PATH=$PATH:$(pwd)/scripts


---

> Install Trimmomatic


    cd $HOME
    wget http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/Trimmomatic-0.32.zip
    unzip Trimmomatic-0.32.zip
    cd Trimmomatic-0.32
    chmod +x trimmomatic-0.32.jar


---

> Install Trinity


    cd $HOME
    svn checkout svn://svn.code.sf.net/p/trinityrnaseq/code/trunk trinityrnaseq-code
    cd trinityrnaseq-code
    make -j8
    PATH=$PATH:$(pwd)


---

> Install Khmer


    cd $HOME
    pip install screed pysam
    git clone https://github.com/ged-lab/khmer.git
    cd khmer
    make -j8
    make install
    PATH=$PATH:$(pwd)/scripts


---
> Download data. For this lab, these data are to be used by people wanting to do the whole pipeline. Most people will want the other dataset I link to below here..


	cd /mnt
	wget https://s3.amazonaws.com/gen711/raw.10M.SRR797058_1.fastq.gz
	wget https://s3.amazonaws.com/gen711/raw.10M.SRR797058_2.fastq.gz


> Alternatively, you can download the pre-corrected, trimmed, normalized datasets. Sadly, I had to subsample this dataset severely (to 500,000 reads) so that we could assemble it in a lab period...


	cd /mnt
	wget https://www.dropbox.com/s/eo3wrx6lvngq3ja/ec.P2.C25.left.fq.gz
	wget https://www.dropbox.com/s/eycchg3m2my2ag2/ec.P2.C25.right.fq.gz


---

> Error correct (do this step if you are working with the raw data only). Note you will have to uncompress the data if you are doing these steps. I chose the software 'lighter' because if is 1. probably good and 2. it is fast! It is written by Ben Langmead, the author of several of the powerpoint lectures I posted last week.


    mkdir /mnt/ec
    cd /mnt/ec
    lighter -r /mnt/raw.10M.SRR797058_1.fastq -r /mnt/raw.10M.SRR797058_2.fastq -t 8 -k 25 100000000 .1


---

> Trim (do this step if you are working with the raw data only)


    mkdir /mnt/trimming
    cd /mnt/trimming
    
    #paste the below lines together as 1 command
    
    java -Xmx10g -jar $HOME/Trimmomatic-0.32/trimmomatic-0.32.jar PE \
    -threads 8 -baseout ec.P2trim.fastQ \
    /mnt/ec/raw.10M.SRR797058_1.cor.fq \
    /mnt/ec/raw.10M.SRR797058_2.cor.fq \
    ILLUMINACLIP:$HOME/Trimmomatic-0.32/adapters/TruSeq3-PE.fa:2:30:10 \
    SLIDINGWINDOW:4:2 \
    LEADING:2 \
    TRAILING:2 \
    MINLEN:25


---

> Run Khmer (do this step if you are working with the raw data only)


    mkdir /mnt/khmer
    cd /mnt/khmer
    interleave-reads.py /mnt/trimming/ec.P2trim.fastQ_1P /mnt/trimming/ec.P2trim.fastQ_2P -o interleaved.fq
    normalize-by-median.py -p -x 15e8 -k 25 -C 25 --out khmer_normalized.fq interleaved.fq
    split-paired-reads.py khmer_normalized.fq


---

> Run Trinity - everybody do this. If you are running with the raw data, you'll have to change the names of the input files. Note that I am using `--min_kmer_cov 2` in the command below. This is only so that you can get through the assembly in a short amount of time. DO NOT USE THIS OPTION IN 'REAL LIFE' AS IT WILL MAKE YOUR ASSEMBLY WORSE!!! This should take ~30 minutes, so use this time to talk to your group members, or whatever else..


    mkdir /mnt/trinity
    cd /mnt/trinity
    Trinity --seqType fq --JM 20G --min_kmer_cov 2 \
    --left /mnt/ec.P2.C25.left.fq \
    --right /mnt/ec.P2.C25.right.fq \
    --CPU 8 --output ec.P2trim.C25 --group_pairs_distance 999 --inchworm_cpu 8


---

> Generate Length Based stats from your assembly. What do these mean?


	$HOME/trinityrnaseq-code/util/TrinityStats.pl ec.P2trim.C25/Trinity.fasta



> lets looks for coding sequences. Before we can do this, we need to install a Perl module using the cpan command.


	cpan URI::Escape
	$HOME/trinityrnaseq-code/trinity-plugins/TransDecoder_r20140704/TransDecoder --CPU 8 -t ec.P2trim.C25/Trinity.fasta


> This will take a few minutes. Once done, you will have a file of amino acid sequences, and coding sequences. Look at how many coding sequences you found, and how many were complete (have a start and stop codon) vs. fragmented in one way or another. What do these numbers mean?? What would you hope these numbers look like. What does `grep -c` do?


    $HOME/trinityrnaseq-code/util/TrinityStats.pl Trinity.fasta.transdecoder.pep
    grep -c complete Trinity.fasta.transdecoder.pep
    grep -c internal Trinity.fasta.transdecoder.pep
    grep -c 5prime Trinity.fasta.transdecoder.pep
    grep -c 3prime Trinity.fasta.transdecoder.pep
    
