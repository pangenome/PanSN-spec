# PanSN-spec: Pangenome Sequence Naming

*Harmonizing biosample information in FASTA, GFA, VCF, BED, and GFF/GTF formats.*

## what?

We specify a backwards-compatible hack to simplify the identification of samples and haplotypes in pangenomes.

It lets us name the sequences in a pangenome and retain that information in any common data format used to describe sequences and variation.

## the pattern

In a pangenome (or a collection of many genomes), we suggest the following sequence naming scheme for sequences:

```
[sample_name][delim][haplotype_id][delim][contig_or_scaffold_name]
```

where

```
sample_name := string
delim := character
haplotype_id := number
contig_or_scaffold_name := string
```

For instance, `HG002#1#ctg3320` names `ctg3320` on the first haplotype (or phase group) of `HG002`, while `HG002#2#ctg9876` is contig `ctg9876` on the other.

We suggest using `#` as a delimiter, but any character not used in your contig or sample identifiers will work.
Tools supporting PanSN should allow the user to change the delimiter.

The prefixing should provide a unique hierachy of sample names and haplotype identifiers for the entire pangenome under analysis.

## deeper motivation

When working with pangenomes it becomes necessary to maintain simple metadata about the origins of each sequence.
The most critical such identification links each sequence to an individual genome or a specific maplotype (e.g. maternal or paternal) in that organism.

In standards like the Sequence Alignment/Map format (SAM) this information is added to each alignment record, and depends on metadata carried in the header of the file.
Each read is part of a read group, and each read group is part of a biosample.
This scheme is flexible and precise, but it involves a hidden hierarchy of relationships that can be hard for users to parse, and it necessitates a header to contain the auxiliary information.

A typical practice when working with pangenomes is to split the pangenome into separate files, with the name of each providing the identity of the specific genome copy represented therein.
This matches per-sample or per-haplotype assemblies generated for each single sample.
But, if we then make alignments, genome graphs, or other assemblies out of these genomes, we will lose sample contig-metadata.

## benefits of the name hack

If we build a pangenome graph from a FASTA file, using a lossless building tool like [`pggb`](https://github.com/pangenome/pggb), our output will be a GFA file whose paths (`P`-lines) encode individual records in the FASTA.
We then can complete numerous downstream operations with this file, such as projecting it into a VCF file with [`vg deconstruct`](https://github.com/vgteam/vg).
But, in order to feed sample information through these three data types, and others that refer to them (such as annotations in `BED` and `GFF`), we have only one option.

We have to stash the information in the sequence name.
This field is found in FASTA, and also common among all bioinformatic formats that refer to FASTA.

By formalizing the way that we communicate sample identification and phase, we can automatically parse this data when making variant calls in `vg` or other tools.
Any alternative arrangement is incompatible with FASTA, due to the lack of standard annotations there.

In many contexts, want to aggregate information by sample or phase/haplotype.
This is trivial to do when the prefix of the path names in our pangenome graph, or in the pangenome FASTA, contains this information.

By using PanSN, we simplify human interpretation.
For instance, it's easy to see which sample a given alignment against a pangenome FASTA.

## how?

You could implement PanSN using an awk script, or perhaps with a custom tool like [`fastix`](https://github.com/ekg/fastix).
Methods that support this convention can then operate on your pangenome sequence names to group data by sample and haplotype.

## opinionated limitations

We suggest not stashing generic data in your sequence names.
PanSN is not for generic metadata.
It is meant to provide a complete, hierarchical decomposition of your data, ideally into biosamples with a single layer of internal subdivisions.

If that doesn't match your needs, this approach is not for you.
Additional metadata is helpful for numerous analyses, but not typically core to the identity of the sequence.
If you need to add further annotations to sequences, use a flat table, a database, or a semantic system to do so.

## who?

Erik Garrison <egarris5@uthsc.edu>
