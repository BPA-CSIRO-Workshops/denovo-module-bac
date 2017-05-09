Key Learning Outcomes
=====================

After completing this module the trainee should be able to:

-   Perform a simple genome assembly for a small organism using `Velvet`

-   Visualise the assembly graph using `Bandage`

-   Be aware of the effects and trade-offs of the parameter `K` on the
    genome assembly

-   Understand that genome assembly produces a graph structure

Resources You’ll be Using
=========================

Tools Used
----------

Velvet
:   \
    <https://www.ebi.ac.uk/~zerbino/velvet/>

Bandage
:   \
    <https://rrwick.github.io/Bandage/>

Data Source and Useful Links
============================

Data used in this tutorial is from ENA accession SRR2054105
:   \
    <http://www.ebi.ac.uk/ena/data/view/SRR2054105>

Spreadsheet used in the group activity
:   \
    <https://docs.google.com/spreadsheets/d/1iFbCCihawpY1LClsAB-OJ66lyeW7EsdJyaMo1HCetF8/edit?usp=sharing>

Introduction
============

In this tutorial we will take raw sequencing reads and *de novo*
assemble them into contigs. We will also explore the internal assembly
graph structure to aid our understanding of why assemblies are
incomplete.

Before the assembly
===================

Sequence data
-------------

We have sequenced the genome of a bacterium using paired-end chemistry
on an Illumina HiSeq 2000 instrument at the University of Oxford and
publicly available as SRR2054105 the Europen Nucleotide Archive (ENA).

Let’s have a look at the datasets by doing:

     cd /home/trainee/bacterial
     ls

You should see the following two files:

R1.fastq.gz

R2.fastq.gz

How many reads are in this data set?\

2100778 pairs

What is the yield in basepairs?\

210077800 bp = 210 Mbp

Assuming an average bacterial genome size of 4 Mbp, what depth of
coverage do we have?\

210 / 4 = 53x

Trim or clip reads
------------------

There are two distinct reasons one may wish to trim or clip the raw
reads.

1.  Low quality bases typically occur toward the end of Illumina reads.
    The lower the quality score, the higher the chance that the base is
    incorrect. This introduces false k-mers into the assembly process. A
    good assembler should handle these gracefully.

2.  Sequencing adaptors are artificial sequence that can occur at the
    end of reads that came from fragments of DNA that were shorter than
    desired. They were not in the original genome and their presence in
    lots of reads will totally confuse the assembler.

Normally one would use `fastqc` or similar to determine whether quality
trimming is warranted and for the presence of sequencing adaptors. The
data set in this exercise is 99% free of adaptors and is good quality
overall so we will skip those steps.

How do you know which adaptor sequence to trim?\

You either ask the sequencing provider or use a tool that tries all
known adapters.

How are are quality values in `FASTQ` files calculated? Can we trust
them?\

The are calculated using formulas calibrated to the physical
measurements the sequencing instrument takes. In the case of Illumina
sequencing, these measurements will include pixel light intensities from
the images the camera/scanner takes of the flow cell.

Allocate yourself to a `K` value on the spreadsheet
---------------------------------------------------

An important parameter to most genome assemblers is `K` which is the
k-mer size used to construct the *de Bruijn* graph (pronounced *de
Brown*) .

Please go to this shared online
spreadsheet<https://docs.google.com/spreadsheets/d/1iFbCCihawpY1LClsAB-OJ66lyeW7EsdJyaMo1HCetF8/edit?usp=sharing>
and choose **a K value** and put your name next to it.

Assemble the reads using Velvet
===============================

The `Velvet` software performs the assembly in two steps. The first
`velveth` step hashes the reads. The second `velvetg` step builds,
cleans and traverses the graph.

Choose an output folder `DIR` and use the `K` value you were allocated
on the spreadsheet and assemble the reads.

    velveth DIR K -shortPaired -fastq.gz -separate R1.fastq.gz R2.fastq.gz

Description of the arguments used in the command:

DIR
:   = Directory name you chose to write the output to

K
:   = K-mer value; it must be an odd number

-shortPaired
:   = The read types are short and paired-end

-fastq.gz
:   = The format of the input files

-separate
:   = R1 and R2 reads are in separate files

input file names
:   = input FASTQ file names

Run the velvetg step now:

    /usr/bin/time -f "%e" velvetg DIR -exp_cov auto -cov_cutoff auto

Description of the arguments used in the command:

time
:   = to capture the `velvetg` command run time

DIR
:   = Directory name you chose to write the velveth output to

-exp\_cov
:   = expected coverage is set to `auto`. To be determined by `velvet`

-cov\_cutoff
:   = coverage cutoff is set to `auto`. To be determined by `velvet`

Add your results to the spreadsheet
-----------------------------------

[H]

ll

**Column** & **Where to find the value**\
`K-mer size` & You chose this earlier and used it in the velveth
command\
`How long it took to run` & Look at the final output line for the user
time in seconds: NN.N\
`Average K-mer coverage ` & Look for Estimated Coverage = NN.N\
`Number of contigs` & Look for text Final graph has NNN nodes\
`N50 contig size` & Look for n50 of NNNNN\
`Largest contig size` & Look for max NNNNN\
`Total number of basepairs (sum of contig lengths)` & Look for total
NNNNNNN\

[tab:results]

Effect of K on assembly
-----------------------

Examine the spreadsheet and look for patterns in the tabular data and
corresponding bar/line charts.

How does K affect each of the statistics? Which value of K do you think
is doing the best job? Why?

Output Files
------------

[H]

ll

**Filename** & **Description**\
`contigs.fa` & this is a FASTA file with your contigs\
`Log` & has most of the metrics in it that we recorded\
`stats.txt` & TSV file of length and coverage of individual contigs\
`Sequences` & A copy of the input FASTQ sequences in FASTA format\
`Pregraph Roadmaps Graph2` & Interim assembly graph structure\
`LastGraph` & Final graph structure\

[tab:velvet~o~ut]

Let’s examine the `stats.txt` file and look at the `short1_cov` column
which is the k-mer coverage of each contig:

    cut -f6 dir/stats.txt | less

Press `SPACE` and `b` to scroll up and down, and press `q` to quote.

What do you notice about the k-mer coverages?\

They all appear to have a similar value, but there are some that do not
fit the pattern.

What do the outliers correspond to?\

Repetitive elements of the genome including gene duplications. Also
replicons with differing copy numbers, like bacterial plasmids.

    grep NN dir/contigs.fa

Why is there N letters in the assembly?\

Paired-end reads come from each end of the same original fragment of
DNA. The unsequenced gap in the middle is variable, and unknown. The
assembler sometimes struggles to resolve these gaps. So it knows two
sections of the genome are connected, but it doesn’t quite know what is
between them. So it pads them with its best guess of how many bases
there are, and uses N to denote them as unknown.

Visualising the assembly graph using Bandage
============================================

The final graph that `Velvet` uses is stored in the file `LastGraph`.
The `Bandage` software allows us to view and explore the assembly graph.

1.  Start `Bandage`.

2.  Go to `File -> Load Graph` and load the `LastGraph` from your
    assembly in `DIR`. This may take a little while so be patient.

3.  Maximize your window to fill up the whole screen.

4.  Click `Draw graph` on the left hand panel.

5.  Change `Random colours` to `Colour by read depth` on the left hand
    panel.

Now get up out of your chair and walks around and look at all the
different graphs the other participants obtained with different values
of `K`.

How does `K` affect the graph?\

Small K produces higher k-mer coverage but also more connectivity as
smaller K is more ambiguous.

What would the graph look like in an ideal situtation?\

If the genome had M replicons, we would like to see only M sub-graphs.
Each sub-graph would be linear or circular, depending on the form of the
replicon in the original organism.

Why didn’t anyone achieve perfection?\

Short reads are unable to disambiguate repeats longer than the read
length (or read span). Most genomes have repeats beyond 500 bp.

Here is another example
<https://github.com/rrwick/Bandage/wiki/Effect-of-kmer-size> from the
Bandage web site on how K can affect assembly graphs.

Explore the graph
-----------------

`Bandage` is designed to be an interactive tool. It allows you to see
relationships between parts of your genome that are lost when you only
look at the FASTA file of contigs.

-   Zoom in using the `Zoom` up/down button in the left hand panel

-   Pan around by holding down the Option/Windows key and dragging on
    the background

-   Move nodes out of the way by selecting and dragging

Feel free to play around a bit and ask questions.

Features of the assembly graph
------------------------------

The graph is quite tangled. The long stretches correspond to contigs.
The intersections correspond to shared k-mers in the graph, which occur
due to repeated sequences within the genome.

Select an node (rectangle) in the graph. It’s length is reported in the
right hand panel as `Length: NNN`.

On the menu choose `Output -> Web BLAST selected node`. Your browser
should open the NCBI BLAST Website
<http://blast.ncbi.nlm.nih.gov/Blast.cgi?PROGRAM=blastn&PAGE_TYPE=BlastSearch&LINK_LOC=blasthome>.

Click the `BLAST` button at the bottom, and wait for the result.

Did your node match anything in the Genbank database?\

You will need to examine the BLAST report on the trainee browser window.

Can you determine what species of bacteria was sequenced?\

The top hit of the small segment of DNA the trainee chose may not
necessarily reflect the true species of bacteria. But if multiple
segments produce consistent top hits to the same species you would have
some confidence.

Conclusion
==========

You should now:

-   know how to use Velvet to assemble a simpel genome from Illumina
    sequences

-   understand the role of the k-mer length K in the assembly process

-   be able to relate the graph structure to the final contigs

-   realize the limitations of short read sequences with respect to
    genome complexity

Thank you.
