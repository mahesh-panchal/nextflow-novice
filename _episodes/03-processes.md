---
title: "Processes"
teaching: 30
exercises: 20
questions:
- "How do I run an application using Nextflow?"
- "How do I pass parameters to the application?"
- "How do I access application results?"
objectives:
- "First learning objective. (FIXME)"
keypoints:
- "First key point. Brief Answer to questions. (FIXME)"
---

## Processes

Processes are the Nextflow data structure used to run user scripts.

## The process data structure

A process has five parts:

- Directives: Defines configuration properties
- Input: Defines the input
- Output: Defines the output
- When: Defines when the process should execute (default: true = always when input is available)
- Script: Defines the script to run.

~~~
process < name > {

   [ directives ]

   input:
    < process inputs >

   output:
    < process outputs >

   when:
    < condition >

   [script|shell|exec]:
   < user script to be executed >

}
~~~
{: .source}

The user script is the only required part of the process that needs to be defined.

## Isolated run folder

Each instance of a process is staged in it's own folder.

Example script:
~~~
x = Channel.of( 'a', 'b', 'c')

process echo_string {

    input:
    val x

    script:
    """
    echo "string-$x"
    """

}
~~~
{: .language-groovy}

~~~
$ ls -a -R work/
~~~
{: .language-bash}
~~~
work/:
.  ..  01  40  b6

work/01:
.  ..  8f8eea1b232f6a6218c0b4998fcac2

work/01/8f8eea1b232f6a6218c0b4998fcac2:
.  ..  .command.begin  .command.err  .command.log  .command.out  .command.run  .command.sh  .exitcode

work/40:
.  ..  e920cb5e8d1a63b338f075444ba38c

work/40/e920cb5e8d1a63b338f075444ba38c:
.  ..  .command.begin  .command.err  .command.log  .command.out  .command.run  .command.sh  .exitcode

work/b6:
.  ..  c9801458f3e4924a57eac3e9154105

work/b6/c9801458f3e4924a57eac3e9154105:
.  ..  .command.begin  .command.err  .command.log  .command.out  .command.run  .command.sh  .exitcode
~~~
{: .output}

The hidden file `.command.sh` contains the interpolated script to
execute.

~~~
cat work/b6/c9801458f3e4924a57eac3e9154105/.command.sh
#!/bin/bash -ue
echo "string-b"
~~~
{: .language-bash}

The hidden file `.command.run` contains code that provides the run-time
environment in which the script is run. Running this file, can be used to
help debug processes during the workflow development stage.

## Process input

Since tasks are isolated from each other, input is passed using
channels.

~~~
input:
  <input qualifier> <input name> [from <source channel>] [attributes]
~~~
{: .source}

The input qualifier declares the type of data received.

- val: A named value of any type to be used as input.
- env: A named environment variable to be used as input.
- file: A file, which is soft-linked into the staging folder so it can be accessed properly in the execution context.
- path: A file, which is soft-linked into the staging folder so it can be accessed properly in the execution context (use this).
- stdin: Takes input from stdin.
- tuple: Declares input as a group of inputs using the above qualifiers.
- each: Executes a task for each input in the collection.

If the `<input name>` matches a name of a channel, it will take input
directly from that. In general `<input name>` is treated as variable
accessible in the process scope. `<input name>` can also be a
string describing a file glob (pathname pattern expansion)
containing the characters `?` or `*`, which
can be useful to control how an input file is named. This is
the same as using the `stageAs` attribute, e.g. `path x, stageAs: 'data.txt' from input_ch`.

|--------------+------------------------+--------------------------------------------------|  
| Pattern      | Input file cardinality | Staged as                                        |
|:-------------|:-----------------------|:-------------------------------------------------|
| `*`          | 1 or many              | Original filename                                |
|--------------+------------------------+--------------------------------------------------|  
| `file?.ext`  | 1                      | `file1.ext`                                      |
| `file?.ext`  | many                   | `file1.ext`, `file2.ext`, `file3.ext`            |
|--------------+------------------------+--------------------------------------------------|  
| `file??.ext` | 1                      | `file01.ext`                                     |
| `file??.ext` | many                   | `file01.ext`, `file02.ext`, `file03.ext`         |
|--------------+------------------------+--------------------------------------------------|  
| `file*.ext`  | 1                      | `file.ext`                                       |
| `file*.ext`  | many                   | `file1.ext`,`file2.ext`,`file3.ext`              |
|--------------+------------------------+--------------------------------------------------|  
| `dir/*`      | 1 or many              | Original filename in a directory `dir`           |
| `dir??/*`    | many                   | Each file staged in its own dir `dir01`, `dir02` |            |
|--------------+------------------------+--------------------------------------------------|  

## Process output

Output should generally be coded to write to the current folder
where it remains isolated. Using the `output` declaration, one can
specify which files should be passed on to further processes,
and/or published as results using a directive.

~~~
output:
  <output qualifier> <output name> [into <target channel>[,<target channel2,...]] [attributes]
~~~
{: .source}

The output qualifier declares the type of data received.

- val: A named value of any type to be taken as output.
- env: A named environment variable to be taken as output.
- file: A filename or glob (pathname pattern expansion) to
be taken as output.
- path: A filename or glob (pathname pattern expansion) to
be taken as output (use this).
- stdout: Reads output from stdout.
- tuple: Declares output as a group of outputs using the above qualifiers.

The `<output name>` can be a literal value, a file glob pattern, a variable in the process scope,
or an input variable. When a file glob pattern is used, a List of files is emitted rather than
a single file object. Input files are by default not included in the list of matched files.

## Conditional processing

A process will only execute when it receives a complete input declaration, i.e. has a data value for
each declared input. However, we can also choose to pass complete input declarations to processes
and execute a process only if an input has a certain property, or alternatively execute a process conditional
on another producing no output. e.g.

An example of checking a property of the input:
~~~
Channel.of('Xa','Xb').into {proc_a_ch; proc_b_ch}

process property_a {

        echo true

        input:
        val str from proc_a_ch

        when:
        str =~ /a/

        script:
        """
        echo Found A!
        """
}

process property_b {

        echo true

        input:
        val str from proc_b_ch

        when:
        str =~ /b/

        script:
        """
        echo Found B!
        """

}
~~~
{: .language-groovy}

An example of execution from no channel input:
~~~
Channel.empty().set { step_a_ch }

process step_a {

        echo true

        input:
        val str from step_a_ch

        output:
        path 'log.txt' into step_b_ch

        script:
        """
        echo Found A! | tee log.txt
        """
}

process step_b_if_not_a {

        echo true

        input:
        val str from step_b_ch.ifEmpty('EMPTY')

        when:
        str == 'EMPTY'

        script:
        """
        echo Did not find A. Executing B!
        """
}
~~~
{: .language-groovy}

## The script

Most commonly, the user script is written as a multi-line string. The
default script language is Bash, but can be any interpreted language, such
as Rscript, Python, Perl, and so on, as long as an interpreter directive
is provided at the start of the script (this is different from the process
directives described next). The only limitation to the commands in
the script is that they must be available on the target execution system.

Example script using R:

~~~
process my_R_script {

    input:
    path 'data.csv' from analysis_ch

    script:
    """
    #! /usr/bin/env Rscript

    data <- read.csv("data.csv")
    """
}
~~~
{: .language-groovy}

One caveat of using Bash as the scripting language is that both Bash
and Nextflow use the same syntax for variables, and so care must be
taken if you want to evaluate a variable in Nextflow context or Bash
context.

~~~
process foo {

    script:
    """
    # Escape the \$ sign to evaluate a variable in Bash context.
    echo "\$PATH is not the same as $PATH"
    """
}
~~~
{: .language-groovy}

One can also use the `shell` block definition with single quote
multi-line strings to do the same thing.

~~~
process foo {

    shell:
    '''
    # Nextflow variables are now referenced with !{var}.
    echo "$PATH is not the same as !{PATH}"
    '''
}
~~~
{: .language-groovy}


{% include links.md %}
