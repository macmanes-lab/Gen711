Lab 6: khmer
--

---

During this lab, we will acquaint ourselves with digital normalization. You will:

1. Install software and download data

2. Quality and adapter trim data sets.

3. Apply digital normalization to the dataset.

4. Count and compare kmers and kmer distributions in the normalized and un-normalized dataset.

5. Plot in RStudio.

-

The JellyFish manual: <a href="ftp://ftp.genome.umd.edu/pub/jellyfish/JellyfishUserGuide.pdf">ftp://ftp.genome.umd.edu/pub/jellyfish/JellyfishUserGuide.pdf</a>

The Khmer manual: <a href="http://khmer.readthedocs.org/en/v1.1/">http://khmer.readthedocs.org/en/v1.1/</a>

---

> Step 1: Launch and AMI. For this exercise, we will use a c3.xlarge instance. Remember to change the permission of your key code `chmod 400 ~/Downloads/????.pem` (change ????.pem to whatever you named it)


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


	apt-get -y install tmux git curl gcc make g++ python-dev unzip default-jre python-pip zlib1g-dev


---

> Install Trimmomatic


    cd $HOME
    wget http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/Trimmomatic-0.32.zip
    unzip Trimmomatic-0.32.zip
    cd Trimmomatic-0.32
    chmod +x trimmomatic-0.32.jar


---

> Install Jellyfish


    cd $HOME
    wget ftp://ftp.genome.umd.edu/pub/jellyfish/jellyfish-2.1.3.tar.gz
    tar -zxf jellyfish-2.1.3.tar.gz
    cd jellyfish-2.1.3/
    ./configure
    make -j4
    PATH=$PATH:$(pwd)/bin


---

> Install Khmer


    cd $HOME
    pip install screed pysam
    git clone https://github.com/ged-lab/khmer.git
    cd khmer
    make -j4
    make install
    PATH=$PATH:$(pwd)/scripts


---
> Download data. For this lab, we'll be using a smaller dataset that consists of 10million paired end reads.

-


	cd /mnt
	wget https://s3.amazonaws.com/gen711/raw.10M.SRR797058_1.fastq.gz
	wget https://s3.amazonaws.com/gen711/raw.10M.SRR797058_2.fastq.gz


---

> Trim low quality bases and adapters from dataset. These files will form the basis of all out subsequent analyses.

-


    mkdir /mnt/trimming
    cd /mnt/trimming
    
    #paste the below lines together as 1 command
    
    java -Xmx10g -jar $HOME/Trimmomatic-0.32/trimmomatic-0.32.jar PE \
    -threads 4 -baseout P2.trimmed.fastQ \
    /mnt/raw.10M.SRR797058_1.fastq.gz \
    /mnt/raw.10M.SRR797058_2.fastq.gz \
    ILLUMINACLIP:$HOME/Trimmomatic-0.32/adapters/TruSeq3-PE.fa:2:30:10 \
    SLIDINGWINDOW:4:2 \
    LEADING:2 \
    TRAILING:2 \
    MINLEN:25


---
> Run Jellyfish on the un-normalized dataset.


    mkdir /mnt/jelly
    cd /mnt/jelly
    
    jellyfish count -m 25 -F2 -s 700M -t 4 -C -o trimmed.jf /mnt/trimming/P2.trimmed.fastQ_1P /mnt/trimming/P2.trimmed.fastQ_2P
    jellyfish histo trimmed.jf -o trimmed.histo


---

> Run Khmer


    mkdir /mnt/khmer
    cd /mnt/khmer
    interleave-reads.py /mnt/trimming/P2.trimmed.fastQ_1P /mnt/trimming/P2.trimmed.fastQ_2P -o interleaved.fq
    normalize-by-median.py -p -x 15e8 -k 25 -C 50 --out khmer_normalized.fq interleaved.fq


---

> Run Khmer on the normalized dataset.


    cd /mnt/jelly
    
    jellyfish count -m 25 -s 700M -t 4 -C -o khmer.jf /mnt/khmer/khmer_normalized.fq
    jellyfish histo khmer.jf -o khmer.histo


> Open up a new terminal window using the buttons command-t


	scp -i ~/Downloads/????.pem ubuntu@ec2-??-???-???-??.compute-1.amazonaws.com:/mnt/jelly/*histo ~/Downloads/


> Now, on your MAC, find the files you just downloaded - for the zip files - double click and that should unzip them.. Click on the `html` file, which will open up your browser. Look at the results. Try to figure out what each plot means.

-

> Now look at the `.histo` file, which is a kmer distribution. I want you to plot the distribution using R and RStudio.

-

> OPEN RSTUDIO


    #Import all 2 histogram datasets: this is the code for importing 1 of them..
    
    khmer <- read.table("~/Downloads/khmer.histo", quote="\"")
    trim <- read.table("~/Downloads/trimmed.histo", quote="\"")
    
    #What does this plot show you?? 
    
    barplot(c(trim$V2[1],khmer$V2[1]),
        names=c('Non-normalized', 'C50 Normalized'),
        main='Number of unique kmers')
    
    # plot differences between non-unique kmers
    
    plot(khmer$V2[10:300] - trim$V2[10:300], type='l',
        xlim=c(10,300), xaxs="i", yaxs="i", frame.plot=F,
        ylim=c(-10000,60000), col='red', xlab='kmer frequency',
        lwd=4, ylab='count',
        main='Diff in 25mer counts of \n normalized vs. un-normalized datasets')
    abline(h=0)



---

<a href="http://genomebio.org/Gen711/wp-content/uploads/2014/10/khmer_norm1.jpeg"><img class="aligncenter size-large wp-image-240" src="http://genomebio.org/Gen711/wp-content/uploads/2014/10/khmer_norm1-1024x700.jpeg" alt="khmer_norm" width="1024" height="700" /></a>

-

-

> What do the analyses of kmer counts tell you?