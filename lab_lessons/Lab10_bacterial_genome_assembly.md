Lab 10: Bacterial Genome Assembly
--

---

During this lab, we will acquaint ourselves with Genome Assembly using SPAdes. We will assembly the genome of <em>E. coli</em>. The data are taken from here: <a href="https://github.com/lexnederbragt/INF-BIOx121_fall2014_de_novo_assembly/blob/master/Sources.md" target="_blank">https://github.com/lexnederbragt/INF-BIOx121_fall2014_de_novo_assembly/blob/master/Sources.md</a>.

<del>1. Install software and download data</del>

<del>2. Error correct, quality and adapter trim data sets.</del>

3. Assemble

-

The SPAdes manuscript: <a href="http://www.ncbi.nlm.nih.gov/pubmed/22506599" target="_blank">http://www.ncbi.nlm.nih.gov/pubmed/22506599</a>
The SPAdes manual: <a href="http://spades.bioinf.spbau.ru/release3.1.1/manual.html" target="_blank">http://spades.bioinf.spbau.ru/release3.1.1/manual.html</a>
SPAdes website: <a href="http://bioinf.spbau.ru/spades" target="_blank">http://bioinf.spbau.ru/spades</a>
ABySS webpage: <a href="https://github.com/bcgsc/abyss" target="_blank">https://github.com/bcgsc/abyss</a>

-

> Step 1: Launch and AMI. For this exercise, we will use a <span style="color: #ff0000;"><strong>c3.2xlarge</strong></span> (note different instance type). Remember to change the permission of your key code `chmod 400 ~/Downloads/????.pem` (change ????.pem to whatever you named it)


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


	apt-get -y install subversion tmux git curl libncurses5-dev gcc make g++ python-dev unzip dh-autoreconf zlib1g-dev libboost1.55-dev sparsehash openmpi*



---

> Install SPAdes


    cd $HOME
    wget http://spades.bioinf.spbau.ru/release3.1.1/SPAdes-3.1.1-Linux.tar.gz
    tar -zxf SPAdes-3.1.1-Linux.tar.gz
    cd SPAdes-3.1.1-Linux
    PATH=$PATH:$(pwd)/bin


---

> Install ABySS


    cd $HOME
    git clone https://github.com/bcgsc/abyss.git
    cd abyss
    ./autogen.sh
    ./configure --enable-maxk=128 --prefix=/usr/local/ --with-mpi=/usr/lib/openmpi/
    make -j4
    make all install


-

> Install a script for assembly evaluation.


    git clone https://github.com/lexnederbragt/sequencetools.git
    cd sequencetools/
    PATH=$PATH:$(pwd)


> Download and unpack the data


	cd /mnt
	wget https://s3.amazonaws.com/gen711/ecoli_data.tar.gz
	tar -zxf ecoli_data.tar.gz


> Assembly. Try this with different data combos (with mate pair data, without, with minION data and without, etc). Remember to name your assemblies something different using the `-o` flag. Spades has a built-in error correction tool (remove `--only-assembler`). Does 'double error correction seem to make a difference?'.


    mkdir /mnt/spades
    cd /mnt/spades
    
    spades.py -t 8 -m 15 --only-assembler --mp1-rf -k 127 \
    --pe1-1 /mnt/ecoli_pe.1.fq \
    --pe1-2 /mnt/ecoli_pe.2.fq \
    --mp1-1 /mnt/nextera.1.fq \
    --mp1-2 /mnt/nextera.2.fq \
    --pacbio /mnt/minion.data.fasta \
    -o Ecoli_all_data


---

> Evaluate Assemblies


    abyss-fac Ecoli_all_data/scaffolds.fasta
    
    #take a closer look.
    
    assemblathon_stats.pl Ecoli_all_data/scaffolds.fasta




> Assembling with ABySS (optional)


    mkdir /mnt/abyss
    cd /mnt/abyss
    
    abyss-pe np=8 k=127 name=ecoli lib='pe1' mp='mp1' long='minion' \
    pe1='/mnt/ecoli_pe.1.fq /mnt/ecoli_pe.2.fq' \
    mp1='/mnt/nextera.1.fq /mnt/nextera.2.fq' \
    minion='/mnt/minion.data.fasta' mp1_l=30


