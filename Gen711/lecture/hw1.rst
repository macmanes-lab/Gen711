==========
HOMEWORK 1
==========

**This assignment is due in lab 18Sept**: It is worth 20 points. 

The purpose of this homework is to get you more familiar with the tasks I taught you in Lab1, specifically BLAST, alignment, tree making. You may work in groups, but you each must submit your own report. Feel free to come see me for help. 

1. Follow the instructions exactly in lab 1, all the way down until you are making a file ``query.fa``. **This command!!**

::

  grep -A1 'ENSPTRP00000032491\|ENSPTRP00000032494' dataset1.pep > query.pep

Instead of using ``ENSPTRP00000032491`` and ``ENSPTRP00000032494``, you are going to use...

- if your last name begins with A-E: ENSPTRP00000000215 and ENSPTRP00000028630 
- if your last name begins with F-K: ENSPTRP00000000198 and ENSPTRP00000044401
- if your last name begins with L-Q: ENSPTRP00000009781 and ENSPTRP00000000220
- if your last name begins with R-Z: ENSPTRP00000007897 and ENSPTRP00000005951

**Can you see** how you would change the original command (``grep -A1 'ENSPTRP00000032491\|ENSPTRP00000032494' dataset1.pep > query.pep``) to search for these new sequences?
 
Once you've done that, you can use BLAST just like you did in lab 1:

::

  blastp -evalue 8e-8 -num_threads 8 -db uniprot -query query.pep -max_target_seqs 3 -outfmt "6 qseqid pident evalue stitle"

**Remember in lab**, that this search gave us several lines of results?? We then searched for these genes using this code:

::

  grep --no-group-separator -A1 'HXA2_HUMAN\|HXA2_BOVIN\|HXA2_PAPAN\|HXA3_HUMAN\|HXA3_MOUSE\|HXA3_BOVIN\|HXA9_HUMAN' uniprot.pep > results.pep

You will do the same type of ``grep`` search, but the names of the genes will be different.. i.e., ``HXA2_HUMAN\|HXA2_BOVIN\|HXA2_PAPAN`` will be different names because you have blasted for different sequences.

In the end, you will have Newick tree code and an alignment. Send me an email with these 2 things. 

============================
Terminate your instance
============================
