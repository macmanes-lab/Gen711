Lab4: Processing fastQ and fastA
--

During this lab, we will acquaint ourselves with the the software packages FastQC and JellyFish. Your objectives are:

-

1. Familiarize yourself with the software, how to execute it, how to visualize results.

2. Regarding your dataset. Characterize sequence quality.

The FastQC manual: <a href="http://www.bioinformatics.babraham.ac.uk/projects/download.html#fastqc">http://www.bioinformatics.babraham.ac.uk/projects/download.html#fastqc</a>

The JellyFish manual: <a href="ftp://ftp.genome.umd.edu/pub/jellyfish/JellyfishUserGuide.pdf">ftp://ftp.genome.umd.edu/pub/jellyfish/JellyfishUserGuide.pdf</a>

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

	apt-get -y install tmux git curl gcc make g++ python-dev unzip default-jre


-

> Ok, for this lab we are going to use FastQC. There is a version available on apt-get, but it is an old version and we want to make sure that we have the most updated version.. <span style="color: #ff0000;"><strong>Make sure you know what each of these commands does, rather than blindly copying and pasting.. </strong></span>

-

    cd $HOME
    wget http://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.2.zip
    unzip fastqc_v0.11.2.zip
    cd FastQC/
    chmod +x fastqc
    PATH=$PATH:$(pwd)


---

> Download data, and uncompress them.. What does the `-cd` flag mean WRT gzip??

-

    cd /mnt
    wget https://s3.amazonaws.com/gen711/Pero360B.1.fastq.gz
    wget https://s3.amazonaws.com/gen711/Pero360B.2.fastq.gz
    gzip -cd /mnt/Pero360B.1.fastq.gz > /mnt/Pero360B.1.fastq &
    gzip -cd /mnt/Pero360B.2.fastq.gz > /mnt/Pero360B.2.fastq &


---
> Install Fastool, a neat and fast tool used for fastQ -->  fastA

-

    cd $HOME
    git clone https://github.com/fstrozzi/Fastool.git
    cd Fastool/
    make
    PATH=$PATH:$(pwd)


---
> Use Fastool to convert from fastQ to fastA

-

    cd /mnt
    fastool --to-fasta Pero360B.1.fastq > Pero360B.1.fasta &
    fastool --to-fasta Pero360B.2.fastq > Pero360B.2.fasta &


---
> While Fastool is working, lets install JellyFish.. <span style="color: #ff0000;"><strong>Again, make sure you know what each of these commands does, rather than just copying and pasting.. </strong></span>

-

    cd $HOME
    wget ftp://ftp.genome.umd.edu/pub/jellyfish/jellyfish-2.1.3.tar.gz
    tar -zxf jellyfish-2.1.3.tar.gz
    cd jellyfish-2.1.3/
    ./configure
    make
    PATH=$PATH:$(pwd)/bin


---
> Run FastQC. Make sure to look at the manual to see what the different outputs mean.

    cd /mnt
    fastqc -t 4 Pero360B.1.fastq Pero360B.2.fastq


---
> Run Jellyfish. Make sure to look at the manual.

    cd /mnt
    mkdir jelly
    cd jelly
    jellyfish count -F2 -m 25 -s 200M -t 4 -C ../Pero360B.1.fasta ../Pero360B.2.fasta
    jellyfish histo mer_counts.jf > Pero360B.histo
    head -50 Pero360B.histo


---
> Open up a new terminal window using the buttons command-t

    scp -i ~/Downloads/????.pem ubuntu@ec2-??-???-???-??.compute-1.amazonaws.com:/mnt/*zip ~/Downloads/
    scp -i ~/Downloads/????.pem ubuntu@ec2-??-???-???-??.compute-1.amazonaws.com:/mnt/jelly/*histo ~/Downloads/


> Now, on your MAC, find the files you just downloaded - for the zip files - double click and that should unzip them.. Click on the `html` file, which will open up your browser. Look at the results. Try to figure out what each plot means.

-

> Now look at the `.histo` file, which is a kmer distribution. I want you to plot the distribution using R and RStudio.

-

> OPEN RSTUDIO


    #Import Data
    histo <- read.table("~/Downloads/Pero360B.histo", quote="\"")
    head(histo)
    
    #Plot
    plot(histo$V2 ~ histo$V1, type='h')
    
    #That one sucks, but what does it tell you about the kmer distribution?
    
    #Maybe this one is better?
    plot(histo$V2 ~ histo$V1, type='h', xlim=c(0,100))
    
    #Better. what is xlim? Maybe we can still improve? 
    
    plot(histo$V2 ~ histo$V1, type='h', xlim=c(0,500), ylim=c(0,1000000))
    
    #Final plot
    
    plot(histo$V2 ~ histo$V1, type='h', xlim=c(0,500), ylim=c(0,1000000),
            col='blue', frame.plot=F, xlab='25-mer frequency', ylab='Count',
            main='Kmer distribution in brain sample before quality trimming')



> Done?