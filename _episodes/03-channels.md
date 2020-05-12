---
title: "Channels"
teaching: 0
exercises: 0
questions:
- "How do I read data into the workflow?"
- "How do I pass data around the workflow?"
objectives:
- "Learn how to read data into the workflow."
- "Learn how data is passed between processes."
keypoints:
- "Channels pass data into and out of processes."
- "There are two types of channel, queue and value channels."
- "Channels can be manipulated using channel operators."
---

A Channel is a data structure designed to efficiently pass
data from one process to another. The primary property of
channels is they are asynchronous. As soon as a process
completes, the results are put in the next channel for
processing, without waiting for other processes. This allows
a following process to start sooner.

Nextflow utilises two types of Channel, queue and value channels.
Queue type channels consume data (first in, first out) as it's passed
from one process to another. This means a queue type channel can
only be used for input to one process. A value type channel however
can be reused and so can be used as input to multiple processes.

## Reading data into a workflow

The primary method of reading data into a Nextflow workflow
is to use Channel factories.

There are several channel factories available:

- `Channel.of`:
    A queue type channel which emits any values passed as parameters to it.
    ~~~
    Channel.of(1, 2, 3, 4, 'X', 'Y')
        .view()
    ~~~
    {: .language-groovy}
    ~~~
    1
    2
    3
    4
    X
    Y
    ~~~
    {: .output}
- `Channel.value`:
    A value type channel which emits a single value.
    ~~~
    val_ch = Channel.value('Reuse_me')
    ~~~
    {: .language-groovy}
- `Channel.fromList`:
    A queue type channel which emits all elements of the `List` passed.
    ~~~
    Channel.fromList(['a','b','c','d'])
        .view()
    ~~~
    {: .language-groovy}
    ~~~
    a
    b
    c
    d
    ~~~
    {: .output}
- `Channel.fromPath`:
    A queue type channel which emits each file that matches the
    path provided. Shell wildcard characters can be used to provide
    a glob pattern to match.
    ~~~
    file_ch = Channel.fromPath('path/to/my/*.files')
    ~~~
    {: .language-groovy}
- `Channel.fromFilePairs`:
    A queue type channel which emits a group of files that match
    the glob pattern provided, along with the common prefix of
    the file group.
    ~~~
    filepair_ch = Channel.fromFilePairs('path/to/my/*_{1,2}.fastq.gz')
    filepair_ch.view()
    ~~~
    {: .language-groovy}
    ~~~
    [SRR2356,[path/to/my/SRR2356_1.fastq.gz,path/to/my/SRR2356_2.fastq.gz]]
    [SRR2357,[path/to/my/SRR2357_1.fastq.gz,path/to/my/SRR2357_2.fastq.gz]]
    [SRR2358,[path/to/my/SRR2358_1.fastq.gz,path/to/my/SRR2358_2.fastq.gz]]
    ~~~
    {: .output}    
- `Channel.watchPath`:
    A queue type channel which emits any filenames matching the glob
    pattern provided. By default, only files created while watching
    the path are emitted. Modification or deletion events can also
    be used to trigger emissions.
- `Channel.empty`:
    An empty channel which emits nothing.

Information on these and other channel factories can be found in the
[Nextflow Channel factory documentation](https://www.nextflow.io/docs/latest/channel.html#channel-factory)

## Channel operators

Channels pass data from one process to another using the `into` and
`from` keywords, which put data in, and take data out of the channels.
The `into` keyword creates a queue type channel which is named by
the creating process and is available from the global scope to another
process.

~~~
process make_file {

    output:
    file "myfile.txt" into file_ch

    script:
    """
    echo "Hello" > myfile.txt
    """
}

process add_world {

    input:
    file 'myfile.txt' from file_ch

    script:
    """
    echo "World" >> myfile.txt
    """
 }
~~~
{: .language-groovy}

Channel operators allow you to manipulate data within channels.
Some common examples are:

- `collect` (value type channel): Gather all data in the channel
    as a single output. A common use case, is when one process
    is used to summarise the output from the previous processes.
    ~~~
    process collect_logs {

        input:
        file logfiles from logs_ch.collect()

        script:
        """
        process_log_files.py
        """
     }
    ~~~
    {: .language-groovy}

- `mix` (queue type channel): Combine data from other channels into
    this one. A common use case is when one wants to process
    data from different stages (e.g., pre- and post-filtering).
    ~~~
    process bam_stats {

        input:
        file bam_file from raw_bam_ch.mix(filtered_bam_ch)

        script:
        """
        samtools flagstat $bam_file > ${bam_file.baseName}.stats
        """
     }
    ~~~
    {: .language-groovy}
- `join` (queue type channel): Join together data based
    on a key. A common use case is to process related files
    that were previously processed separately. Reading
    input from two queue type channels simultaneously does
    not guarantee correcting pairing of data due to their
    asynchronicity, and therefore must be combined using a
    common property before processing.
    ~~~
    process annotate_blastp {

        input:
        tuple val(sample), file(fasta_file) from blastp_fasta_files

        output:
        tuple val(sample), file("*.tsv") into blastp_annotation_files

        script:
        """
        blastp ... > ${sample}_blastp-annotation.tsv
        """
    }

    process annotate_interproscan {

        input:
        tuple val(sample), file(fasta_file) from interproscan_fasta_files

        output:
        tuple val(sample), file("*.tsv") into interproscan_annotation_files

        script:
        """
        interproscan ... > ${sample}_interproscan-annotation.tsv
        """
    }

    process merge_annotations {

        input:
        tuple val(sample), file(blast_annotation), file(interproscan_annotation) from blastp_annotation_files.join(interproscan_annotation_files)

        output:
        file("${sample}_merged-annotations.gff")

        script:
        """
        merge_annotations.py $blast_annotation $interproscan_annotation > ${sample}_merged-annotations.gff
        """
     }
    ~~~
    {: .language-groovy}

{% include links.md %}
