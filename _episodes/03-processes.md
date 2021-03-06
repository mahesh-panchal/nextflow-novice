---
title: "Processes"
teaching: 30
exercises: 20
questions:
- "How do I include a software tool into a Nextflow workflow?"
- "How do I pass parameters to the tool?"
- "How do I access tool results?"
objectives:
- "Understand the process data structure"
- "Detail how and when a process runs from Nextflow"
- "Construct and run a complete process definition"
keypoints:
- "A process has five parts: directives, input, output, when, and the script."
- "Each process task runs from it's own directory."
- "A process only executes when there is a complete process input declaration."
- "Selected process output can be saved to a folder using `publishDir`."
- "Directives control process execution behaviour."
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

After running the example script, a `work` folder is created where
process tasks are cached.

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
~~~
{: .language-bash}
~~~
#!/bin/bash -ue
echo "string-b"
~~~
{: .output}

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

- `val`: A named value of any type to be used as input.
- `env`: A named environment variable to be used as input.
- `file`: A file, which is soft-linked into the staging folder so it can be
  accessed properly in the execution context.
* `path`: A backwards-compatible drop-in replacement for `file` with additional
  functionality introduced in Nextflow 19.10.0, and is thus preferred over
  `file`.
- `stdin`: Takes input from stdin.
- `tuple`: Declares input as a group of inputs using the above qualifiers.
- `each`: Executes a task for each input in the collection.

If the `<input name>` matches a name of a channel, it will take input
directly from that. In general `<input name>` is treated as variable
accessible in the process scope. `<input name>` can also be a
string describing a file glob (pathname pattern expansion)
containing the characters `?` or `*`, which
can be useful to control how an input file is named.

~~~
process stage {

    input:
    path 'dir/*' from log_ch.collect()
    path csv_files, stageAs: 'data??.csv' from csv_ch.collect()

    script:
    """
    ls *.csv
    process_files.py $csv_files
    ls dir
    """
}
~~~
{: .language-groovy}

A table of how file glob inputs are interpreted:

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

- `val`: A named value of any type to be taken as output.
- `env`: A named environment variable to be taken as output.
- `file`: A filename or glob (pathname pattern expansion) to be taken as
  output.
- `path`: A filename or glob (pathname pattern expansion) to be taken as output
  (preferred over `file`, see the "input" section above).
- `stdout`: Reads output from stdout.
- `tuple`: Declares output as a group of outputs using the above qualifiers.

The `<output name>` can be a literal value, a file glob pattern, a
variable in the process scope, or an input variable. A process task
will always emit just one value into the output channel. When a file
glob pattern is used, a List of files is emitted rather than
a single file object. Input files are by default not included
in the list of matched files.

~~~
process outfiles {

    output:
    path "somefile.txt" into onefile_ch
    path "lots*.file" into manyfiles_ch

    script:
    """
    touch somefile.txt lots{1..4}.file
    """
}

onefile_ch.view()
manyfiles_ch.view()
~~~
{: .language-groovy}

~~~
N E X T F L O W  ~  version 20.01.0
Launching `view_outputfiles.nf` [stoic_blackwell] - revision: 07b06e56ad
executor >  local (1)
[78/ea9da5] process > outfiles [100%] 1 of 1 ✔
<pwd>/<work_cachedir>/somefile.txt
[<pwd>/<work_cachedir>/lots1.file, <pwd>/<work_cachedir>/lots2.file, <pwd>/<work_cachedir>/lots3.file, <pwd>/<work_cachedir>/lots4.file]
~~~
{: .output}
where the current directory has been replaced with `<pwd>`, and the
work directory has been replaced with `<work_cachedir>`.

## Conditional processing

A process will only execute when it receives a complete input declaration,
i.e. has a data value for each declared input. We can also limit a
process execution further to when an input has a certain property, by
using the `when` process block. This takes any expression that
returns a true or false value. If we want to execute a process
based on another process having no output, we can create an input
when the channel is empty, and use the `when` block to test for that
specific input.

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
taken if you want to evaluate a variable in Nextflow context (by
the nextflow interpreter) or in Bash context (by the bash interpreter).

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

The script block can be thought of as a function that returns
the multi-line string as the result. As such, a script block
allows the inclusion of additional code statements including
control structures such as if statements to dictate which
code snippet should be executed.

~~~
process my_task {

    input:
    tuple val(sample), path(reads) from my_read_ch

    script:
    if ( sample ~ /control/ ) {
        options = "Options for control"
        """
        echo "Do this with control $options"
        """
    } else {
        options = "Options for cases"
        """
        echo "Do that with cases -I -X $options"
        """
    }
}
~~~
{: .language-groovy}

Scripts do not need to be included as multi-line strings
into a Nextflow workflow process. They
can be written in a separate file that can be used as a stand-alone
script or as `template` script. Stand-alone scripts should be stored
in the `bin` folder within the same directory as the workflow, and
can be executed as though they were a normal script available from the
`PATH` environment variable. A template script should be stored
in the `templates` folder within the same directory as the workflow.
Template scripts interpret variables starting with `$` in Nextflow
context, and must be called using the `template` function. Variables
can either be assigned to from the input declaration or from the
script block before the template is called.

~~~
process task_A {

    script:
    """
    # Run my_script.pl from bin/
    my_script.pl
    """
}

process task_B {

    input:
    val var from var_ch

    script:
    template vars.sh
}
~~~
{: .language-groovy}

where `vars.sh` looks like:

~~~
#! /usr/bin/env bash

echo "This is my $var"
~~~
{: .language-bash}

## Directives

Directives describe the execution behaviour of the process, from the
compute resources to reserve, to where it should put the results.
There are lots of directives described in the [documentation](https://www.nextflow.io/docs/latest/process.html#directives), which allow
you to customise the running of your workflow to your infrastructure.
The directives should appear before the input, output, when, and
script blocks, or alternatively can be provided via configuration files
(recommended for certain directives).

Here is a table of some useful directives.

|-----------+-------------|
| Directive | Description |
|-----------+-------------|
| `publishDir` | Defines where files from the output block should be written. Multiple `publishDir` directives can be provided to a process, along with the `pattern` parameter to write output to separate paths. |
| `label` | Provides the process with a label that can be used for configuring a group of processes. |
| ----------+-------------|
| `executor` | Defines the system on which the processes are executed. This could be local execution, submission to a job scheduler, or to other services. |
| `cpus` | Assigns the number of cores to reserve. This can be accessed within the script block using `${task.cpus}`. |
| `time` | Assigns the maximum length of time the task should run for. |
| `memory` | Assigns the amount of memory to reserve. |
| `queue` | Defines the queue to use for certain grid based executors |
| `clusterOptions` | Defines additional options necessary for your grid bases executors.
| `scratch` | Defines whether a process should be run on a temporary folder local to the execution node (Highly recommended when using a grid executor!). Only output files are copied back across to the working dir (Saving valuable disk space, but losing potentially helpful files from unsuccessful tasks). |
| ----------+-------------|
| `module` | Defines which software modules to use. |
| `conda` | Defines which conda software packages to use. |
| `container` | Defines which container images of software to use. Use `containerOptions` to provide process specific additional configuration. |
| ----------+-------------|
| `errorStrategy` | Describes how Nextflow should behave when a process terminates with an error. |
|-----------+-------------|

Directives can also be dynamically defined using a closure.

~~~
process foo {

    memory { 2.GB * task.attempt }
    time { 1.hour * task.attempt }

    errorStrategy { task.exitStatus in 137..140 ? 'retry' : 'terminate' }
    maxRetries 3

    script:
    """
    <your job here>
    """

}
~~~
{: .language-groovy}

A special variable `task` is accessible to the process block, which
provides the runtime values of the task directives and more.

~~~
process qualimap {

    input:
    tuple val(sample), path(alignment) from aln_ch

    script:
    bamfile = alignment.find { it =~ /\.bam$/ }
    mem = task.memory ? task.memory.toGiga() : task.cpus * 6
    """
    export JAVA_OPTS="-Xms32m -Xmx${mem}G -Djava.io.tmpdir=${task.scratch}"
    qualimap bamqc -bam $bamfile -outdir ${bamfile.baseName} -outformat 'HTML' -nt ${task.cpus}
    """
}
~~~
{: .language-groovy}

The variables available through the task configuration are partially
documented in the second table in [Tracing Documentation pages](https://www.nextflow.io/docs/latest/tracing.html#trace-report).

> ## Exercises
>
> - Order the blocks
> - Stage a parameter file
> - Capture output from one process and stage in another.
>     - Stage a folder
>     - Stage some files
> - When does a process task execute.
>     - Incomplete header
{: .challenge}

{% include links.md %}
