Lab 5: Trimming
--

---

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
    make
    PATH=$PATH:$(pwd)/bin


---

> Download data. For this lab, we'll be using only 1 sequencing file.

-


	cd /mnt
	wget https://s3.amazonaws.com/gen711/Pero360B.2.fastq.gz


---

> Do 3 different trimming levels between 2 and 40. This one is trimming at a Phred score of 30 (BAD!!!) When you run your commands, you'll need to change the numbers in `LEADING:30` `TRAILING:30` `SLIDINGWINDOW:4:30` and `Pero360B.trim.Phred30.fastq` to whatever trimming level you want to use.


    mkdir /mnt/trimming
    cd /mnt/trimming
    
    #paste the below lines together as 1 command
    
    java -Xmx10g -jar $HOME/Trimmomatic-0.32/trimmomatic-0.32.jar SE \
    -threads 4 \
    ../Pero360B.2.fastq.gz \
    Pero360B.trim.Phred30.fastq \
    ILLUMINACLIP:$HOME/Trimmomatic-0.32/adapters/TruSeq3-PE.fa:2:30:10 \
    SLIDINGWINDOW:4:30 \
    LEADING:30 \
    TRAILING:30 \
    MINLEN:25



---
> After Trimmomatic is done, Run FastQC. You'll have to change the numbers to match the levels you trimmed at.


    cd /mnt
    fastqc -t 4 Pero360B.2.fastq.gz
    fastqc -t 4 trimming/Pero360B.trim.Phred2.fastq
    fastqc -t 4 trimming/Pero360B.trim.Phred15.fastq
    fastqc -t 4 trimming/Pero360B.trim.Phred30.fastq


---
> Run Jellyfish.


	mkdir /mnt/jelly
	cd /mnt/jelly

	# You'll have to run these commands 4 separate times -
	# once for each different trimmed dataset, and once for the raw dataset.
	# Change the names of the input and output files..

	jellyfish count -m 25 -s 200M -t 4 -C -o trim30.jf ../trimming/Pero360B.trim.Phred30.fastq
	jellyfish histo trim30.jf -o trim30.histo


---
> Open up a new terminal window using the buttons command-t


	scp -i ~/Downloads/????.pem ubuntu@ec2-??-???-???-??.compute-1.amazonaws.com:/mnt/*zip ~/Downloads/
	scp -i ~/Downloads/????.pem ubuntu@ec2-??-???-???-??.compute-1.amazonaws.com:/mnt/jelly/*histo ~/Downloads/


> Now, on your MAC, find the files you just downloaded - for the zip files - double click and that should unzip them.. Click on the `html` file, which will open up your browser. Look at the results. Try to figure out what each plot means.

-

> Now look at the `.histo` file, which is a kmer distribution. I want you to plot the distribution using R and RStudio.

-

> OPEN RSTUDIO


    #Import all 3 histogram datasets: this is the code for importing 1 of them..
    
    trim2 <- read.table("~/Downloads/trim2.histo", quote="\"")
    
    #Plot: Make sure and change the names to match what you import.
    #What does this plot show you?? 
    
    barplot(c(trim2$V2[1],trim15$V2[1],trim30$V2[1]),
        names=c('Phred2', 'Phred15', 'Phred30'),
        main='Number of unique kmers')
    
    # plot differences between non-unique kmers
    
    plot(trim2$V2[2:30] - trim30$V2[2:30], type='l',
        xlim=c(2,20), xaxs="i", yaxs="i", frame.plot=F,
        ylim=c(0,2000000), col='red', xlab='kmer frequency',
        lwd=4, ylab='count',
        main='Diff in 25mer counts of freq 2 to 20 \n Phred2 vs. Phred30')




> Look at the FastQC plots across the different trimming levels. Anything surprising?

> What do the analyses of kmer counts tell you?