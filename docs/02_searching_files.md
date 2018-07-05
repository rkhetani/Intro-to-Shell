---
title: "The Shell: Searching and Redirection"
author: "Sheldon  McKay, Bob Freeman, Mary Piper"
date: "Wednesday, August 23, 2017"
---

Approximate time: 60 minutes

## Learning objectives

- Learn how to search for characters or patterns in a text file using the `grep` command
- Learn how to write to file and append to file using output redirection
- Explore how to use the pipe (`|`) character to chain together commands

## Searching files

We went over how to search within a file using `less`. We can also
search within files without even opening them, using `grep`. Grep is a command-line
utility for searching plain-text data sets for lines matching a pattern or regular expression (regex).
Let's give it a try!

We are going to practice searching with `grep` using our FASTQ files, which contain the sequencing reads (nucleotide sequences) output from a sequencing facility. Each sequencing read in a FASTQ file is associated with four lines of output, with the first line (header line) always starting with an `@` symbol. A whole fastq record for a single read should appear similar to the following:

	@HWI-ST330:304:H045HADXX:1:1101:1111:61397
	CACTTGTAAGGGCAGGCCCCCTTCACCCTCCCGCTCCTGGGGGANNNNNNNNNNANNNCGAGGCCCTGGGGTAGAGGGNNNNNNNNNNNNNNGATCTTGG
	+
	@?@DDDDDDHHH?GH:?FCBGGB@C?DBEGIIIIAEF;FCGGI#########################################################

Suppose we want to see how many reads in our file `Mov10_oe_1.subset.fq` are "bad", with 10 consecutive Ns (`NNNNNNNNNN`).

```bash
$ cd ~/unix_lesson/raw_fastq

$ grep NNNNNNNNNN Mov10_oe_1.subset.fq
```

We get back a lot of lines.  What if we want to see the whole fastq record for each of these reads? 

We can use the `-B` and `-A` arguments for grep to return the matched line plus one before (`-B1`) and two lines after (`-A2`). Since each record is four lines and the second line is the sequence, this should return the whole record.

```bash
$ grep -B 1 -A 2 NNNNNNNNNN Mov10_oe_1.subset.fq
```

```
@HWI-ST330:304:H045HADXX:1:1101:1111:61397
CACTTGTAAGGGCAGGCCCCCTTCACCCTCCCGCTCCTGGGGGANNNNNNNNNNANNNCGAGGCCCTGGGGTAGAGGGNNNNNNNNNNNNNNGATCTTGG
+
@?@DDDDDDHHH?GH:?FCBGGB@C?DBEGIIIIAEF;FCGGI#########################################################
--
@HWI-ST330:304:H045HADXX:1:1101:1106:89824
CACAAATCGGCTCAGGAGGCTTGTAGAAAAGCTCAGCTTGACANNNNNNNNNNNNNNNNNGNGNACGAAACNNNNGNNNNNNNNNNNNNNNNNNNGTTGG
+
?@@DDDDDB1@?:E?;3A:1?9?E9?<?DGCDGBBDBF@;8DF#########################################################
```

***

**Exercises**

1. Search for the sequence CTCAATGA in `Mov10_oe_1.subset.fq`.
In addition to finding the sequence, have your search also return
the name of the sequence.

2. Search for that sequence in all Mov10 replicate fastq files.

***

## Redirection

We're excited we have all these sequences that we care about that we
just got from the FASTQ files. That is a really important motif
that is going to help us answer our important question. But all those
sequences just went whizzing by with grep. How can we capture them?

We can do that with something called "redirection". The idea is that
we're redirecting the output from the terminal (all the stuff that went
whizzing by) to something else. In this case, we want to print it
to a file, so that we can look at it later.

**The redirection command for writing something to file is `>`.**

Let's try it out and put all the sequences that contain 'NNNNNNNNNN'
from all the files into another file called `bad_reads.txt`.

```bash
$ grep -B 1 -A 2 NNNNNNNNNN Mov10_oe_1.subset.fq > bad_reads.txt
```

The prompt should sit there a little bit, and then it should look like nothing
happened. But you should have a new file called `bad_reads.txt`. 

```bash
$ ls -l
```

Take a look at the file and see if it contains what you think it should. *NOTE: If we already had a file named `bad_reads.txt` in our directory, it would have overwritten it without any warning.*
 
**The redirection command for appending something to an existing file is `>>`.**

If we use `>>`, it will append to rather than overwrite a file.  This can be useful for saving more than one search, for example.
    
```bash
$ grep -B 1 -A 2 NNNNNNNNNN Mov10_oe_2.subset.fq >> bad_reads.txt

$ ls -l
```

Since our `bad_reads.txt` file isn't a raw_fastq file, we should move it to a different location within our directory. We decide to move it to the `other` folder using the command `mv`. 

```bash
$ mv bad_reads.txt ../other/
```

There's one more useful redirection command that we're going to show, and that's called the pipe command. 

**The redirection command for using the output of a command as input for a different command is `|`.**

It's probably not a key on your keyboard you use very much. What `|` does is take the output that went scrolling by on the terminal and runs it through another command. When it was all whizzing by before, we wished we could just slow it down and look at it, like we can with `less`. Well it turns out that we can! We pipe the `grep` command to `less` or to `head` to just see the first few lines.

```bash
$ grep -B 1 -A 2 NNNNNNNNNN Mov10_oe_1.subset.fq | less
```

Now we can use the arrows to scroll up and down and use `q` to get out.

We can also do count the number of lines using the `wc` command. `wc` stands for *word count*. It counts the number of lines, words or characters. So, we can use it to count the number of lines we're getting back from our `grep` command using the `-l` argument. And that will magically tell us how many bad sequences we have in the file.

```bash
$ grep NNNNNNNNNN Mov10_oe_1.subset.fq | wc -l
```

This command when used without any arguments would tell us the number of lines, words and characters in the file; the `-l` flag specifies that we only want the number of lines. Try it out without the `-l` to see the full output.

Redirecting is not super intuitive, but it's really powerful for stringing together these different commands, so you can do whatever you need to do.

The philosophy behind these commands is that none of them really do anything all that impressive. BUT when you start chaining them together, you can do some really powerful things really efficiently. **To be able to use the shell effectively, becoming proficient with the pipe and redirection operators:  `|`, `>`, `>>` is essential.**

## Practice with searching and redirection (piping)

Finally, let's use the new tools in our kit and a few new ones to examine our gene annotation file, **chr1-hg19_genes.gtf**, which we will be using later to find the genomic coordinates of all known exons on chromosome 1.

```bash
$ cd ~/unix_lesson/reference_data/
```

Let's explore our `chr1-hg19_genes.gtf` file a bit. What information does it contain?

```bash
$ less chr1-hg19_genes.gtf
```

	chr1    unknown exon    14362   14829   .       -       .       gene_id "WASH7P"; gene_name "WASH7P"; transcript_id "NR_024540"; tss_id "TSS7245";
	chr1    unknown exon    14970   15038   .       -       .       gene_id "WASH7P"; gene_name "WASH7P"; transcript_id "NR_024540"; tss_id "TSS7245";
	chr1    unknown exon    15796   15947   .       -       .       gene_id "WASH7P"; gene_name "WASH7P"; transcript_id "NR_024540"; tss_id "TSS7245";
	chr1    unknown exon    16607   16765   .       -       .       gene_id "WASH7P"; gene_name "WASH7P"; transcript_id "NR_024540"; tss_id "TSS7245";
	chr1    unknown exon    16858   17055   .       -       .       gene_id "WASH7P"; gene_name "WASH7P"; transcript_id "NR_024540"; tss_id "TSS7245";


> The GTF file is a tab-delimited gene annotation file often used in NGS analyses. For more information on this file format, check out the [Ensembl site](http://useast.ensembl.org/info/website/upload/gff.html). 

The columns in the **GTF file contain the genomic coordinates of gene features (exon, start_codon, stop_codon, CDS) and the gene_names, transcript_ids and protein_ids (p_id) associated with these features**. Note that sometimes an exon can be associated with multiple different transcripts or gene isoforms. For example, 

```bash
$ grep PLEKHN1 chr1-hg19_genes.gtf | head -n 5
```

This search returns two different transcripts of the same gene, NM_001160184 and NM_032129, that contain the same exon.

Now that we know what type of information is inside of our gtf file, let's explore our commands to answer a simple question about our data: **how many unique exons are present on chromosome 1 using `chr1-hg19_genes.gtf`?**

To determine the number of unique exons on chromosome 1, we are going to perform a series of steps:
	
	1. Extract only the genomic coordinates of exon features
	2. Subset the dataset to only include the feature type and genomic location information
	3. Remove duplicate exons
	4. Count the total number of exons
	
#### Extracting exon features

We only want the exons (not CDS or start_codon features), so let's use `grep` to only keep the exon lines and pipe it to head to see what we get:

```bash
$ grep exon chr1-hg19_genes.gtf | head
```

#### Subsetting dataset to only keep genomic coordinates

We will define the uniqueness of an exon by its genomic coordinates. Therefore, we only need the genomic location (chr, start, stop, and strand) information to find the total number of unique exons. The columns corresponding to this information are 1, 4, 5, and 7. 

'cut' is a program that will extract columns from files.  It is a very good command to know.  Let's first try out the 'cut' command on a just the exonic lines to make sure we have the command correct by using multiple piped commands and looking at the first 10 lines:

```bash
$ grep exon chr1-hg19_genes.gtf | cut -f1,4,5,7  | head
```
   
`-f1,4,5,7` means to cut these fields (columns) from the dataset.  

	chr1	14362	14829	-
	chr1	14970	15038	-
	chr1	15796	15947	-
	chr1	16607	16765	-
	chr1	16858	17055	-

The `cut` command assumes our data columns are separated by tabs (i.e. tab-delimited). The `chr1-hg19_genes.gtf` is a tab-delimited file, so the default `cut` command works for us. However, data can be separated by other types of delimiters. Another common delimiter is the comma, which separates data in comma-separated value (csv) files. If your data is not tab delimited, there is a `cut` command argument (`-d`) to specify the delimiter.

Our output looks good, so let's keep going...

#### Removing duplicate exons

Now, we need to remove those exons that show up multiple times for different transcripts. For this, we can use a new tool, `sort`, to remove exons that show up more than once. We can use the `sort` command with the `-u` option to return only unique lines.

```bash
$ grep exon chr1-hg19_genes.gtf | cut -f1,4,5,7 | sort -u | head 
```

Do you see a change in how the sorting has changed? By default the `sort` command will sort and what you can't see here is that it has removed the duplicates. How do we check this?

#### Counting the total number of exons

First, let's check how many lines we would have without using `sort -u` by piping the output to `wc -l`.

```bash
grep exon chr1-hg19_genes.gtf | cut -f1,4,5,7 | wc -l
```

Now, to count how many unique exons are on chromosome 1, we will add back the `sort -u` and pipe the output to `wc -l`

```bash
$ grep exon chr1-hg19_genes.gtf | cut -f1,4,5,7 | sort -u | wc -l
```

What we did in one command up here, we could have done it in multiple steps by saving the output of each command to a file, but that would not be as efficient if all we needed was a number to work with. The intermediate files are not useful and they occupy precious space on the computer and add clutter to the file system. 

**Commands, options, and keystrokes covered in this lesson**

```bash
grep
> (output redirection)
>> (output redirection, append)
| (output redirection, pipe)
wc
cut
sort
```

---
*This lesson has been developed by members of the teaching team at the [Harvard Chan Bioinformatics Core (HBC)](http://bioinformatics.sph.harvard.edu/). These are open access materials distributed under the terms of the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0), which permits unrestricted use, distribution, and reproduction in any medium, provided the original author and source are credited.*

* *The materials used in this lesson were derived from work that is Copyright © Data Carpentry (http://datacarpentry.org/). 
All Data Carpentry instructional material is made available under the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0).*

* *Adapted from the lesson by Tracy Teal. Contributors: Paul Wilson, Milad Fatenejad, Sasha Wood, and Radhika Khetani for Software Carpentry (http://software-carpentry.org/)*

