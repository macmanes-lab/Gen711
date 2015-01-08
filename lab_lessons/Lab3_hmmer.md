Lab3 : HMMER

During this lab, we will acquaint ourselves with the the software package HMMER. Your objectives are:

-

1. Familiarize yourself with the software, how to execute it, how to visualize results.

2. Regarding your dataset. Characterize a few conserved domains.

The HMMER manual <a href="ftp://selab.janelia.org/pub/software/hmmer3/3.1b1/Userguide.pdf">ftp://selab.janelia.org/pub/software/hmmer3/3.1b1/Userguide.pdf</a>

The HMMER webpage: <a href="http://hmmer.janelia.org/">http://hmmer.janelia.org/</a>

---

> Step 1: Launch and AMI. For this exercise, we will use a c3.xlarge instance. Remember to change the permission of your key code `chmod 400 ~/Downloads/your.pem` (change your.pem to whatever you named it)


	ssh -i ~/Downloads/your.pem ubuntu@ec2-???-???-???-???.compute-1.amazonaws.com


---


	sudo bash
	apt-get update
	apt-get -y upgrade
	apt-get -y install tmux git curl gcc make g++ python-dev unzip default-jre


-

> Ok, for this lab we are going to use HMMER

-


	cd $HOME
	wget http://selab.janelia.org/software/hmmer3/3.1b1/hmmer-3.1b1-linux-intel-x86_64.tar.gz
	tar -zxf hmmer-3.1b1-linux-intel-x86_64.tar.gz
	cd hmmer-3.1b1-linux-intel-x86_64/
	./configure
	make && make all install
	make check


---

-

> You will download one of the 5 different datasets (use the same dataset). Do you remember how to use the `wget` and `gzip` commands from last week? Also, download Swissprot and Pfam-A

-


	cd /mnt

	#download your dataset

	dataset1= https://www.dropbox.com/s/srfk4o2bh1qmq6l/dataset1.fa.gz
	dataset2= https://www.dropbox.com/s/977n0ibznzuor22/dataset2.fa.gz
	dataset3= https://www.dropbox.com/s/8s2h7sm6xtoky6q/dataset3.fa.gz
	dataset4= https://www.dropbox.com/s/qth3mjrianb48a6/dataset4.fa.gz
	dataset5= https://www.dropbox.com/s/quexoxfh6ttmudo/dataset5.fa.gz

	#download the SwissProt database

	wget ftp://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/complete/uniprot_sprot.fasta.gz

	#download the Pfam-A database

	wget ftp://ftp.ebi.ac.uk/pub/databases/Pfam/current_release/Pfam-A.hmm.gz


> we are going to run HMMER to identify conserved protein domains. This will take a little while, and we'll use `tmux` to allow us to do this in the background, and continue to work on other things.


    gzip -d *gz
    tmux new -s pfam
    hmmpress Pfam-A.hmm #this is analogous to 'makeblastdb'
    hmmscan -E 1e-3 --domtblout dataset.pfam --cpu 4 Pfam-A.hmm dataset1.fa
    ctl-b d
    top -c #see that hmmscan is running..


> the neat thing about HMMER is that it can be used as a replacement for blastP or PSI-blast.


    #blastp-like HBB-HUMAN is a Hemoglobin B protein sequence. 
    
    phmmer --domtblout hbb.phmmer -E 1e-5 \
    /home/ubuntu/hmmer-3.1b1-linux-intel-x86_64/tutorial/HBB_HUMAN \
    uniprot_sprot.fasta
    
    #PSI-blast-like
    
    jackhmmer --domtblout hbb.jackhammer -E 1e-5 \
    /home/ubuntu/hmmer-3.1b1-linux-intel-x86_64/tutorial/HBB_HUMAN \
    uniprot_sprot.fasta
    
    #you can look at the results using `more hmm.phmmer` or `more hmm.jackhmmer`. Try blasting a few of the results using the BLAST web interface.
    

> Now let's look at the Pfam results. This analyses may still be running, but we can look at it while it's still in progress.


    more dataset.pfam
    #There are a bunch of columns in this table - what do they mean?
    
    #Try to extract all the hits to a specific domain. Google a few domains (column 1) to see if any seem interesting. 
    
    #for instance, find all occurrences of ABC_tran
    grep ABC_tran dataset.pfam
    
    #use grep to count the number of matches. Copy this number down.
    
    grep -c ABC_tran dataset.pfam
    
    #Find all the contigs that have a ABC_tran domain. 
    
    grep ABC_tran dataset.pfam | awk '{print $4}' | sort | uniq
    

> Just for fun, check on the Pfam search to see what it is doing... 


    tmux attach -t pfam
    ctl-b d

