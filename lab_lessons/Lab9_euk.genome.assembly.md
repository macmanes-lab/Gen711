Lab 9: Genome assembly
--

---

During this lab, we will acquaint ourselves with Genome Assembly using SPAdes. We will assembly the genome of Plasmodium falciparum. The data are taken from this paper: http://www.nature.com/ncomms/2014/140909/ncomms5754/full/ncomms5754.html?WT.ec_id=JA-NCOMMS-20140919.
As it stands right now, I think that you will do all the preprocessing steps this week, then the assembly next. Once you have done all the steps, `gzip` compress the files and download them to your USB drive, or the MAC HD. I can provide you with these files next week if issues arise. 

1. Install software and download data

2. Error correct, quality and adapter trim data sets.

3. (next week) Assemble

-

The SPAdes manuscript: http://www.ncbi.nlm.nih.gov/pubmed/22506599
The SPAdes manual: http://spades.bioinf.spbau.ru/release3.1.1/manual.html
SPAdes website: http://bioinf.spbau.ru/spades

> Step 1: Launch and AMI. For this exercise, we will use a <span style="color: #ff0000;"><strong>c3.8xlarge</strong></span> (note different instance type). Remember to change the permission of your key code `chmod 400 ~/Downloads/????.pem` (change ????.pem to whatever you named it)


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
    cd Lighter
    make -j8
    PATH=$PATH:$(pwd)


---

> Install Trimmomatic


    cd $HOME
    wget http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/Trimmomatic-0.32.zip
    unzip Trimmomatic-0.32.zip
    cd Trimmomatic-0.32
    chmod +x trimmomatic-0.32.jar
    

---

> Install SRAtoolkit


    cd $HOME
    wget http://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/2.4.2/sratoolkit.2.4.2-ubuntu64.tar.gz
    tar -zxf sratoolkit.2.4.2-ubuntu64.tar.gz
    PATH=$PATH:/home/ubuntu/sratoolkit.2.4.2-ubuntu64/bin


---

> Install SPAdes


    wget http://spades.bioinf.spbau.ru/release3.1.1/SPAdes-3.1.1-Linux.tar.gz
    tar -zxf SPAdes-3.1.1-Linux.tar.gz
    cd SPAdes-3.1.1-Linux
    PATH=$PATH:$(pwd)/bin


---

> Download 3.5kb MP library


	cd /mnt
	wget ftp://ftp-trace.ncbi.nlm.nih.gov/sra/sra-instant/reads/ByRun/sra/ERR/ERR022/ERR022558/ERR022558.sra 


> Download 10kb MP library


	cd /mnt
	wget ftp://ftp-trace.ncbi.nlm.nih.gov/sra/sra-instant/reads/ByRun/sra/ERR/ERR022/ERR022557/ERR022557.sra 


> Download PE library #1


	cd /mnt
	wget ftp://ftp-trace.ncbi.nlm.nih.gov/sra/sra-instant/reads/ByRun/sra/ERR/ERR019/ERR019273/ERR019273.sra


---

> Download PE library #2


	cd /mnt
	wget ftp://ftp-trace.ncbi.nlm.nih.gov/sra/sra-instant/reads/ByRun/sra/ERR/ERR019/ERR019275/ERR019275.sra


---

> Extract fastQ from sra format.


    cd /mnt
    
    #this is a basic for loop. Copy is all as 1 line. 
    
    for i in `ls *sra`; do
        fastq-dump --split-files --split-spot $i;
        rm $i;
    done


---

> Error Correct Data


    mkdir /mnt/ec
    cd /mnt/ec
    lighter -r /mnt/ERR019273_1.fastq -r /mnt/ERR019273_2.fastq -t 32 -k 21 45000000 .1
    lighter -r /mnt/ERR022557_1.fastq -r /mnt/ERR022557_2.fastq -t 32 -k 21 45000000 .1
    lighter -r /mnt/ERR022558_1.fastq -r /mnt/ERR022558_2.fastq -t 32 -k 21 45000000 .1
    lighter -r /mnt/ERR019275_1.fastq -r /mnt/ERR019275_2.fastq -t 32 -k 21 45000000 .1
    
    #remove the raw files. 
    
    rm *fastq &


> trim the data:


    mkdir /mnt/trim
    cd /mnt/trim
    #paste the below lines together as 1 command
    
    java -Xmx10g -jar $HOME/Trimmomatic-0.32/trimmomatic-0.32.jar PE \
    -threads 32 -baseout PE_lib1.fq \
    /mnt/ec/ERR019273_1.cor.fq \
    /mnt/ec/ERR019273_2.cor.fq \
    ILLUMINACLIP:$HOME/Trimmomatic-0.32/adapters/TruSeq2-PE.fa:2:30:10 \
    SLIDINGWINDOW:4:2 \
    LEADING:2 \
    TRAILING:2 \
    MINLEN:25
    
    #paste the below lines together as 1 command
    
    java -Xmx10g -jar $HOME/Trimmomatic-0.32/trimmomatic-0.32.jar PE \
    -threads 32 -baseout PE_lib2.fq \
    /mnt/ec/ERR019275_1.cor.fq \
    /mnt/ec/ERR019275_2.cor.fq \
    ILLUMINACLIP:$HOME/Trimmomatic-0.32/adapters/TruSeq2-PE.fa:2:30:10 \
    SLIDINGWINDOW:4:2 \
    LEADING:2 \
    TRAILING:2 \
    MINLEN:25
    
    #paste the below lines together as 1 command
    
    java -Xmx10g -jar $HOME/Trimmomatic-0.32/trimmomatic-0.32.jar PE \
    -threads 32 -baseout MP10000.fq \
    /mnt/ec/ERR022557_1.cor.fq \
    /mnt/ec/ERR022557_2.cor.fq \
    ILLUMINACLIP:$HOME/Trimmomatic-0.32/adapters/TruSeq2-PE.fa:2:30:10 \
    SLIDINGWINDOW:4:2 \
    LEADING:2 \
    TRAILING:2 \
    MINLEN:25
    
    #paste the below lines together as 1 command
    
    java -Xmx10g -jar $HOME/Trimmomatic-0.32/trimmomatic-0.32.jar PE \
    -threads 32 -baseout MP3500.fq \
    /mnt/ec/ERR022558_1.cor.fq \
    /mnt/ec/ERR022558_2.cor.fq \
    ILLUMINACLIP:$HOME/Trimmomatic-0.32/adapters/TruSeq2-PE.fa:2:30:10 \
    SLIDINGWINDOW:4:2 \
    LEADING:2 \
    TRAILING:2 \
    MINLEN:25
    
    


---


    mkdir /mnt/
    
    #remove the corrected files. 
    
    rm *fq



> Assembly. This you may want to do next week. Alternatively, you can put it in a tmux window and let it run. You'd have to login later however to download the assembled genome.  


    mkdir spades
    cd spades
    
    spades.py -t 32 -m 60 \
    --pe1-1 /mnt/trim/PE_lib1_1P.fq \
    --pe1-2 /mnt/trim/PE_lib1_2P.fq \
    --pe2-1 /mnt/trim/PE_lib2_1P.fq \
    --pe2-2 /mnt/trim/PE_lib2_2P.fq \
    --mp1-1 /mnt/trim/MP3500_1P.fq \
    --mp1-2 /mnt/trim/MP3500_2P.fq \
    --mp2-1 /mnt/trim/MP10000_1P.fq \
    --mp2-2 /mnt/trim/MP10000_2P.fq \
    -o Pfal --only-assembler

