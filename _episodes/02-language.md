---
title: "The Nextflow Language"
teaching: 40
exercises: 20
questions:
- "What language is nextflow written in?"
- "How do I write a nextflow script?"
- "How can I run a nextflow script?"
objectives:
- "Describe the Groovy syntax."
- "Write a Nextflow workflow."
- "Run a Nextflow workflow."
keypoints:
- "Nextflow is written in the Groovy computer language."
- "Channels and Processes are the fundamental data structures of Nextflow."
- "A workflow script is run using `nextflow run <script.nf>`."
---

Nextflow is a Domain Specific Language (DSL) designed to ease
the writing of computational pipelines. It is an extension of the
computer language Groovy, which is a superset of the computer
language Java. As such, Nextflow can execute any piece of Groovy
code or use any library for the JVM platform.

This section describes how to write and run a nextflow workflow.

## The Groovy language

> ## What's your scripting language experience?
>
> - How many of you use a scripting language on a regular basis?
>    - Bash
>    - Python
>    - Perl
>    - R
>    - Groovy
{: .discussion}

Before writing a workflow, here are some fundamental
concepts of the Groovy language.

- Single line comments (lines not meant to be executed) start
with `//`. Multi-line comments are nested between `/*` and `*/` tags.
    ~~~
    // This is a single line comment. Everything after the // is ignored.
    /*
        Comments can also
        span multiple
        lines.
    */
    ~~~
    {: .language-groovy}
- Variables are assigned using `=` and can have any value.
    ~~~
    myvar = 1                           // Integer
    myvar = -3.1499392                  // Floating point number
    myvar = false                       // Boolean
    myvar = "Hello world"               // String
    myvar = new java.util.Date()        // Object - Abstract data structure
    ~~~
    {: .language-groovy}
- Lists (also known as arrays) are defined using the square bracket `[]` notation.
    ~~~
    emptylist = []                      // an empty list
    // Lists can contain duplicates, and the values can be of any type.
    mixedList = [1, 1, 'string', true, null, 5 as byte]
    mixedList.add("new value")          // adds "new value" to the end of mixedList
    println mixedList.size()            // prints the size
    println mixedList[0]                // prints 1
    ~~~
    {: .language-groovy}
- Maps (also known as associative arrays) are defined using the `[:]` literal. They associate a unique string with a value, and are commonly referred to as key-value pairs.
    ~~~
    emptyMap = [:]                      // an empty map
    mymap = [ name : "Steve", age: 43, likes: ['walks','cooking','coding']]
    mymap.age = 44                      // values can be accessed using map property notation.
    println mymap['name']               // alternatively values can be accessed using quoted keys.
    ~~~
    {: .language-groovy}
- Groovy supports common control structures such as if/else tests,
switch/case tests, for loops, and while loops.
    ~~~
    // if / else test
    x = false
    if ( !x ) {
        return 1
    } else {
        return 0
    }

    // switch / case test
    switch (x) {
        case "found foo":
            result = "Got foo"
            // continues to following cases

        case Number:
            result = "Number"
            break          // stops at this case

        default:
            result = "default"
    }

    // for loop (other variants exist)
    String message = ''
    for (int i = 0; i < 5; i++) {
        message += 'Hi '
    }

    // while loop
    y = 5
    while ( y-- > 0 ){
        println "Are we there yet?"
    }
    println "We've arrived!"
    ~~~
    {: .language-groovy}

Groovy is very syntax-rich and supports many more operations. A full
description of Groovy semantics can be found in the [Groovy Documentation](https://groovy-lang.org/semantics.html).

> ## Can I do X in Groovy / Nextflow too?
>
> - Are there other data and control structures that you commonly
> use in your analyses?
> - What kind of things would you like to do in your computational
> pipeline?
{: .discussion}

## Writing a workflow

To ease the writing of computational pipelines Nextflow introduces two
high-level data structures; channels, and processes.

A channel is a data-flow object that passes data asynchronously from
one process to another. Channels provide methods for reading in data
from various sources. Nextflow was developed with a strong emphasis
on supporting bioinformatics, and as such includes methods for
supporting common file formats like fasta and fastq. Channels send
data in a first in, first out manner (FIFO), however data may arrive
at the next channel in a different order (asynchrony) due to
process execution time, or manipulation of channel values by
channel operators.

A process is a task that executes a user script. The user
script can be written in any interpreted language, although the default
is bash. Each task defined by a process is executed independently,
and in isolation, and so input must be communicated using channels.

Example (`example.nf`):
~~~
#! /usr/bin/env nextflow

number_ch = Channel.from(1,2,3,4)

process Sequence {

    input:
    val max from number_ch

    output:
    stdout into out_ch

    script:
    """
    echo {0..$max}
    """

}

out_ch.view()
~~~
{: .language-groovy}

> ## Write a nextflow script
>
> - Create and save a file `myscript.nf` with the directive
> to use the nextflow interpreter.
>
> > ## Solution
> >
> > The `myscript.nf` script should now look like this:
> >
> > ~~~
> > #! /usr/bin/env nextflow
> > ~~~
> > {: .language-groovy}
> {: .solution}
>
> - Add a channel `word_ch` to the script that sends the words, "This","is","my","nextflow","script" into the channel.
>
> > ## Solution
> >
> > ~~~
> > #! /usr/bin/env nextflow
> >
> > word_ch = Channel.from("This","is","my","nextflow","script")
> > ~~~
> > {: .language-groovy}
> {: .solution}
>
> - Use the `view` method to display the content of the `word_ch` channel.
>
> > ## Solution
> >
> > ~~~
> > #! /usr/bin/env nextflow
> >
> > word_ch = Channel.from("This","is","my","nextflow","script")
> > word_ch.view()
> > ~~~
> > {: .language-groovy}
> {: .solution}
>
> - Add a process `Display_Words` the to script. Take input
> from the `word_ch` channel.
> Use the shell command `echo` to print the each word.
>
> > ## Solution
> >
> > ~~~
> > #! /usr/bin/env nextflow
> >
> > word_ch = Channel.from("This","is","my","nextflow","script")
> > word_ch.view()
> >
> > process Display_Words {
> >
> >     input:
> >     val word from word_ch
> >
> >     script:
> >     """
> >     echo $word
> >     """
> >
> > }
> > ~~~
> > {: .language-groovy}  
> {: .solution}
{: .challenge}

## Running a workflow.

A Nextflow workflow is executed using the `nextflow run <script.nf>`
command. Each task is executed locally (on your computer) by default,
and expects all the commands in your process scripts to be
available on the command line. While local execution is suitable for
small scale data processing, Nextflow integrates support for several
third-party softwares enabling large scale data processing through
various package management tools, job schedulers, and distributed
compute infrastructure tools (See supplementary materials).

~~~
$ nextflow run example.nf
N E X T F L O W  ~  version 20.01.0
Launching `test.nf` [marvelous_ride] - revision: 614fc2b804
executor >  local (4)
[3e/7b764f] process > Sequence [100%] 4 of 4 ✔
0 1 2

0 1

0 1 2 3 4

0 1 2 3

~~~
{: .language-bash}

If given a path to a version control repository, Nextflow can
download and execute a workflow from it.

> ## Run your own workflows
>
> - Run your `myscript.nf` script
>
> > ## Solution
> >
> > ~~~
> > $ nextflow run myscript.nf
> > ~~~
> > {: .language-bash}
> {: .solution}
{: .challenge}

{% include links.md %}
