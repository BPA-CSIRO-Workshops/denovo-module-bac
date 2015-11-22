# HD3 – Your first genome assembly

Torsten Seemann, Paul Berkman, Philippe Moncuquet

## Introduction

In this tutorial we will take raw sequencing reads and _de novo_ assemble them
into contigs. We will also explore the internal assembly graph structure to
aid our understand where, how and why assemblies are incomplete.

## Learning objectives

* To be able to perform a simple genome assembly for a small organism
* To be aware of the effects and trade-offs of the parameter K on the genome assembly
* To understand that genome assembly produces a graph structure

## Instructions

### Sequence data

We have sequenced the genome of an bacterium using paired-end chemistry on an Illumina HiSeq 2000 instrument at the University of Oxford and publicly available as [SRR2054105](http://www.ebi.ac.uk/ena/data/view/SRR2054105):
```
R1.fastq.gz
R2.fastq.gz
```
* _How many reads are in this data set?_
* _What is the yield in basepairs?_
* _Assuming an average bacterial genome size of 4 Mbp, what depth of coverage do we have?_

### Trim or clip reads

There are two distinct reasons one may wish to trim or clip the raw reads.

1. *Low quality bases* typically occur toward the end of Illumina reads. The lower the quality score, the higher the chance that the base is incorrect. This introduces false k-mers into the assembly process. A good assembler should handle these gracefully.

2. *Seqeuncing adaptors* are artificial sequence that can occur at the end of reads that came from fragments of DNA that were shorter than desired. They were not in the original genome and their presence in lots of reads will totally confuse the assembler.

Normally one would use `fastqc` or similar to determine whether quality trimming is warranted and for the presence of sequencing adaptors. The data set in this exercise is 99% free of adaptors and is good quality overall so we will skip those steps.

* _How do you know which adaptor sequence to trim?_
* _How are are quality values in FASTQ files calculated? Can we trust them?_

### Allocate yourself to a `K` value on the spreadsheet

An important parameter to most genome assemblers is `K` which is the k-mer size used to construct the de Bruijn graph (pronounced _de Brown_) .

Please go to this
[shared online spreadsheet](https://docs.google.com/spreadsheets/d/1iFbCCihawpY1LClsAB-OJ66lyeW7EsdJyaMo1HCetF8/edit?usp=sharing)
and **choose a K value** and put your name next to it.

### Assemble the reads using _Velvet_

The Velvet software performs the assembly in two steps. The first `velveth` step hashes the reads. The second `velvetg` step builds, cleans and traverses the graph.

Choose an output folder `DIR` and use the `K` value you were allocated on the spreadsheet and assemble the reads.

```
velveth DIR K –shortPaired –fastq.gz –separate R1.fastq.gz R2.fastq.gz
```

```
/usr/bin/time –f “%e” velvetg DIR -exp_cov auto –cov_cutoff auto
```

### Add your results to the spreadsheet

Using the dir/velvet.out and dir/Log files determine the values for the following assembly statistics and add them to the [spreadsheet](https://docs.google.com/spreadsheets/d/1iFbCCihawpY1LClsAB-OJ66lyeW7EsdJyaMo1HCetF8/edit?usp=sharing) from earlier :

Column | Where to find the value
-------|------------------------
K-mer size | You chose this earlier and used it in the `velveth` command
How long it took to run |	Look at the final output line for the user time in seconds: `NN.N`
Average K-mer coverage |	Look for `Estimated Coverage = NN.N`
Number of contigs |	Look for text `Final graph has NNN nodes`
“N50” contig size |	Look for `n50 of NNNNN`
Largest contig size | Look for `max NNNNN`
Total number of basepairs (sum of contig lengths) | Look for `total NNNNNNN`

### Effect of K on assembly

Examine the [spreadsheet](https://docs.google.com/spreadsheets/d/1iFbCCihawpY1LClsAB-OJ66lyeW7EsdJyaMo1HCetF8/edit?usp=sharing)
and look for patterns in the tabular data and corresponding bar/line charts.

* _How does K affect each of the statistics?_
* _Which value of K do you think is doing the best job? Why?_


### Output files

Filename | Description
---------|------------
`contigs.fa` |this is a FASTA file with your contigs
`Log` | has most of the metrics in it that we recorded
`stats.txt` |	TSV file of length and coverage of individual contigs
`Sequences` |	A copy of the input FASTQ sequences in FASTA format
`Pregraph` `Roadmaps` `Graph2` | Interim assembly graph structure
`LastGraph` |	Final graph structure

Let's examine the `stats.txt` file and look atht eh `short1_cov` column which is the k-mer coverage of each contig:

```
cut –f6 dir/stats.txt | less
```

* _What do you notice about the k-mer coverages?_
* _What do the outliers correspond to?_

```
grep NN dir/contigs.fa
```
* _Why is there N letters in the assembly?_

### Visualising the assembly graph using _Bandage_

The final graph that Velvet uses is stored in the file `LastGraph`. The Bandage software allows us to view and explore the assembly graph.

1. Start `Bandage`.
2. Go to `File -> Load Graph` and load the `LastGraph` from your assembly in `DIR`. This may take a little while so be patient.
3. Maximize your window to fill up the whole screen.
4. Click `Draw graph` on the left hand panel.
5. Change `Random colours` to `Colour by read depth` on the left hand panel.

Now get up out of your chair and walks around and look at all the different graphs the other participants obtained with different values of `K`.

* _How does `K` affect the graph?_
* _What would the graph look like in an ideal situtation?_
* _Why didn't anyone achieve perfection?_

Here is [another example](https://github.com/rrwick/Bandage/wiki/Effect-of-kmer-size) from the Bandage web site.

### Explore the graph

`Bandage` is designed to be an interactive tool. It allows you to see relationships between parts of your genome that are lost when you only look at the FASTA file of contigs.

* Zoom in using the `Zoom` up/down button in the left hand panel
* Pan around by holding down the Option/Windows key and dragging on the background
* Move nodes out of the way by selecting and dragging

Feel free to play around a bit and ask questions.

### Features of the assembly graph

The graph is quite tangled. The long stretches correspond to contigs. The intersections correspond to shared k-mers in the graph, which occur due to repeated sequences within the genome.

1. Select an node (rectangle) in the graph. It's length is reported in the right hand panel as `Length: NNN`.
2. On the menu choose `Output -> Web BLAST selected node`. Your browser should open the
[NCBI BLAST Website](http://blast.ncbi.nlm.nih.gov/Blast.cgi?PROGRAM=blastn&PAGE_TYPE=BlastSearch&LINK_LOC=blasthome).
3. Click the `BLAST` button at the bottom, and wait for the result.

Questions:
* _Did your node match anything in the Genbank database?_
* _Can you determine what species of bacteria was sequenced?_

## Conclusion

You should now:
* know how to use Velvet to assemble a simpel genome from Illumina sequences
* understand the role of the k-mer length `K` in the assembly process
* be able to relate the graph structure to the final contigs
* realize the limitations of short read sequences with respect to genome complexity

Thank you.
