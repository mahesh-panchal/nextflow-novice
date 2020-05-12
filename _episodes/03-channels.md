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

Other Channel factories exist



{% include links.md %}
