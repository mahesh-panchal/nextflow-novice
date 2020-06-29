---
title: "Channels"
teaching: 30
exercises: 20
questions:
- "How do I read data into the workflow?"
- "How do I pass data around the workflow?"
objectives:
- "Learn how to read data into the workflow."
- "Learn how data is passed between processes."
- "Learn how to use channel operations to wrangle data into the required input form."
keypoints:
- "Channels pass data into and out of processes."
- "There are two types of channel, queue and value channels."
- "Channels can be manipulated using channel operators."
---

## What is a Channel?

A Channel is a data structure designed to efficiently pass
data from one process to another. The primary property of
channels is they are asynchronous. As soon as a process
completes, the results are put in the next channel for
processing, without waiting for other processes. This allows
a following process to start sooner.

Nextflow utilises two types of Channel, queue and value channels.
Queue type channels consume data in a first in, first out manner
to create process input declarations. The data in value type channels
however can be reused when constructing process input declarations.
Since a process must perform operations on input channels to make
an input declaration and spawn a task, each process needs it's own input
channel. This is achieved using the `into` operator on a channel to create
two or more channels that can be used by different processes.

~~~
Channel.of(1,2,3,4).into { sq_ch; db_ch }

process square {

    input:
    val x from sq_ch

    shell:
    """
    echo $x*$x | bc -l
    """
}

process double {

    input:
    val x from sq_ch

    shell:
    """
    echo 2*$x | bc -l
    """
}
~~~
{: .language-groovy}

## Reading data into a workflow

The primary method of reading data into a Nextflow workflow
is to use Channel factories; methods that produce channels.

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
process WriteHello {

    output:
    file "myfile.txt" into file_ch

    script:
    """
    <commands>
    """
}

process AddWorld {

    input:
    file 'myfile.txt' from file_ch

    script:
    """
    <commands>
    """
 }
~~~
{: .language-groovy}

Channel operators allow you to manipulate data within channels.
Some common examples are:

- `view` (queue type channel): Prints channel data values to
    the console standard output. This is mostly useful for
    debugging your own workflows, in particular when you see
    an unclear error message related to file input.
    ~~~
    process WrapText {

        input:
        file text_file from text_file_ch.view()

        script:
        """
        <commands>
        """

     }
    ~~~
    {: .language-groovy}
- `into` (value type channel): Emits the data from one channel into
    one or more other named channels. This is commonly used to
    provide the same input to multiple processes.
    ~~~
    Channel.fromFilePairs('/path/to/reads_{1,2}.fastq.gz')
        .into { fastqc_read_ch; fastp_read_ch }

    process FastP {

        input:
        tuple val(sample), file(reads) from fastp_read_ch

        output:
        tuple val(sample), file("${sample}_trimmed_{1,2}.fastq.gz") into (bwa_ch, screen_ch)

        script:
        """
        <commands>
        """

    }

    process FastQC {

        input:
        tuple val(sample), file(reads) from fastqc_read_ch

        script:
        """
        <commands>
        """

    }
    ~~~
    {: .language-groovy}    
- `collect` (value type channel): Gather all data in the channel
    as a single output. A common use case, is when one process
    is used to summarise the output from the previous processes.
    ~~~
    process CollectLogs {

        input:
        file logfiles from logs_ch.collect()

        script:
        """
        <commands>
        """

    }
    ~~~
    {: .language-groovy}
- `mix` (queue type channel): Combine data from other channels into
    this one. A common use case is when one wants to process
    data from different stages (e.g., pre- and post-filtering)
    with the same analysis process.
    ~~~
    process BAMStats {

        input:
        file bam_file from raw_bam_ch.mix(filtered_bam_ch)

        script:
        """
        <commands>
        """

    }
    ~~~
    {: .language-groovy}
- `join` (queue type channel): Join together data based
    on a common attribute. A common use case is to process related files
    that were previously processed separately. Reading
    input from two queue type channels simultaneously does
    not guarantee correcting pairing of data due to their
    asynchronicity, and therefore must be combined using a
    common property before processing.
    ~~~
    process AnnotateBlastp {

        input:
        tuple val(sample), file(fasta_file) from blastp_fasta_files

        output:
        tuple val(sample), file("${sample}_blastp-annotation.tsv") into blastp_annotation_files

        script:
        """
        <commands>
        """

    }

    process AnnotateInterproscan {

        input:
        tuple val(sample), file(fasta_file) from interproscan_fasta_files

        output:
        tuple val(sample), file("${sample}_interproscan-annotation.tsv") into interproscan_annotation_files

        script:
        """
        <commands>
        """

    }

    process MergeAnnotations {

        input:
        tuple val(sample), file(blast_annotation), file(interproscan_annotation) from blastp_annotation_files.join(interproscan_annotation_files)

        script:
        """
        <commands>
        """

    }
    ~~~
    {: .language-groovy}

Many more channel operators are described in the [Nextflow Channel
Operator Documentation](https://www.nextflow.io/docs/latest/operator.html).

## Multiple input channels

It is important to understand how multiple input channels are processed.
When two or more channels are declared as process inputs, the process
waits until it receives an input value from all the channels
declared as input.

### Two or more queue type channels.

~~~
process foo {

    echo true

    input:
    val x from Channel.of(1,2)
    val y from Channel.of('a','b','c')

    script:
    """
    echo $x and $y
    """
}
~~~
{: .language-groovy}

In this case the process `foo` will only run two times since there
are only two inputs in the first channel. Channel values are
consumed, and so there is nothing left to pair with `'c'`, which
is discarded.

In the example above, it should be noted that while the process will
execute on the pairings `1 and a`, and `2 and b`, that for more complex
workflows, the queues are asynchronous meaning there's no guarantee
of having the pairing `1 and a`, and `2 and b`. The emission of
`'c'` may happen first resulting in a `1 and c` pairing. If certain
files must be processed together, use one of the queue combining
operators such as `join` or `groupBy` to generate the correct pairing
before being passed as input.

### Value channels with queue channels.

~~~
process foo {

    echo true

    input:
    val x from Channel.value(1)
    val y from Channel.of('a','b','c')

    script:
    """
    echo $x and $y
    """
}
~~~
{: .language-groovy}

In this example, the process `foo` runs three times as values
from value type channels can be reused. This produces
the pairings `1 and a`, `1 and b`, and `1 and c`. This is useful
when using one process to generate an index that is then used
for processing multiple input files in another process. The
indexing process outputs are collected into a value channel,
e.g., using `toList` or `collect`, that can be reused for
multiple down stream processes.

> ## Exercises
>
> - mix operator
> - join operator
> - each -> cross
{: .challenge}

{% include links.md %}
