Lab 2: BLAST
--

During this lab, we will acquaint ourselves with the the software package BLAST. Your objectives are:

-

1. Familiarize yourself with the software, how to execute it, how to visualize results.

2. Regarding your dataset, tell me how some of these genes are related to their homologous copies.

-

---

> Step 1: Launch and AMI. For this exercise, we will use a c3.xlarge instance.


	ssh -i ~/Downloads/your.pem ubuntu@ec2-???-???-???-???.compute-1.amazonaws.com

-

> The machine you are using is Linux Ubuntu: Ubuntu is an operating system you can use (I do) on your laptop or desktop. One of the nice things about this OS is the ability to update the software, easily.  The command `sudo apt-get update` checks a server for updates to existing software.

-


	sudo apt-get update

-

> The upgrade command actually installs any of the required updates.


	sudo apt-get -y upgrade


> OK, what are these commands?  `sudo` is the command that tells the computer that we have admin privileges. Try running the commands without the sudo -- it will complain that you don't have admin privileges or something like that. *Careful here, using sudo means that you can do something really bad to your own computer -- like delete everything*, so use with caution. It's not a big worry when using AWS, as this is a virtual machine- fixing your worst mistake is as easy as just terminating the instance and restarting.

-

> So now that we have updates the software, lets see how to add new software. Same basic command, but instead of the `update` or `upgrade` command, we're using `install`. EASY!!

-


	sudo apt-get -y install tmux git curl gcc make g++ python-dev unzip \
        default-jre

-

> ok, for this lab we are going to use BLAST, which is available as a package entitled `ncbi-blast+`

-


	sudo apt-get -y install ???

-

> to get a feel for the different options, type `blastp -help`. Which type of blast does this correspond to? Look at the help info for blastp and tblastx

-

> Let's go root


	sudo bash

---

Install mafft and RAxML
--

> Let's install mafft so that we can do an alignment (<a href="http://mafft.cbrc.jp/alignment/software/">http://mafft.cbrc.jp/alignment/software/</a>)


    cd $HOME
    wget http://mafft.cbrc.jp/alignment/software/mafft-7.164-without-extensions-src.tgz
    tar -zxf mafft-7.164-without-extensions-src.tgz
    cd mafft-7.164-without-extensions/core
    sudo make && sudo make install
    PATH=$PATH:/home/ubuntu/mafft-7.164-without-extensions/core

-

> Now lets install RAxML so that we can make a phylogeny. ()


    cd $HOME
    git clone https://github.com/stamatak/standard-RAxML.git
    cd standard-RAxML/
    make -f Makefile.PTHREADS.gcc
    PATH=$PATH:/home/ubuntu/standard-RAxML

-

> remember, for blasting we need both some data (a query) and a database. Lets start with the data 1st. You will have one of the 5 different datasets. Do you remember how to use the `wget` and `gzip` commands from last week?

-


    cd /mnt
    dataset1= https://www.dropbox.com/s/srfk4o2bh1qmq6l/dataset1.fa.gz
    dataset2= https://www.dropbox.com/s/977n0ibznzuor22/dataset2.fa.gz
    dataset3= https://www.dropbox.com/s/8s2h7sm6xtoky6q/dataset3.fa.gz
    dataset4= https://www.dropbox.com/s/qth3mjrianb48a6/dataset4.fa.gz
    dataset5= https://www.dropbox.com/s/quexoxfh6ttmudo/dataset5.fa.gz

-

> Now let's download the database. For this exercise we will use Swissprot: `ftp://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/complete/uniprot_sprot.fasta.gz`

> unzip this file using `gzip -d`

---

Make blast database and blast
--

-

> make a blast database


	makeblastdb -in uniprot_sprot.fasta -out uniprot -dbtype prot

> Now we are ready to blast.


	head -n20 dataset1.fa > test.fa
	blastp -evalue 1e-10 -num_threads 4 -db uniprot -query test.fa -outfmt 6

> You will see the results in a table with 12 columns. Use `blastp -help` to see what the results mean.

> Test out some of the blast options. Try changing the word size `-word_size`, scoring matrix, evalue, cost to open or extend a gap. See how these changes affect the results.

> After you've done this, you should make a file containing the query and the hits.


	grep -A4 -w AAseq_1 dataset1.fa 

    #use the dataset you have, and substitute your contig for AAseq_1
    #increase -A4 until the whole contigs is displayed.
    #copy and paste it into nano.
    #do the same for the database matches. 
    
    grep -A4 'sp|Q6GZX4|001R_FRG3G' uniprot_sprot.fasta

---

mafft
--

> Align the proteins using mafft


	mafft --reorder --bl 80 --auto for.align > for.tree

---

RAxML
--

> Make a phylogeny


	raxmlHPC-PTHREADS -help
	raxmlHPC-PTHREADS -f a -m PROTCATBLOSUM62 -T 4 -x 34 -N 100 -n tree -s for.tree -p 35

> Copy phylogeny and view online.


	more RAxML_bipartitionsBranchLabels.tree

	#copy this info.

> Visualize tree on website: <a href="http://iubio.bio.indiana.edu/treeapp/treeprint-form.html">http://iubio.bio.indiana.edu/treeapp/treeprint-form.html</a>