---
title: "Workflow Configuration"
teaching: 0
exercises: 0
questions:
- "How do I configure Nextflow for my infrastructure?"
objectives:
- "Learn how to separate workflow logic from execution."
- "Learn how to create workflow profiles."
- "Learn how configuration nesting works."
keypoints:
- "First key point. Brief Answer to questions. (FIXME)"
---

## Configuration

Configuration files are a collection of name and value pairs that
tell Nextflow how to behave. They provide a way to separate workflow
logic from the execution environment, allowing workflow portability
and a cleaner workflow script.

When a Nextflow script is launched, it looks for configuration
settings in three places: `nextflow.config` in the current directory,
`nextflow.config` in the script directory (if not the same as the
current directory), and finally `config` in `$HOME/.nextflow`.
Additional configuration can be provided with the `-c <config>` parameter.
When two or more of these configuration settings exist, they are merged
where `-c <config>` overrides settings in `$PWD/nextflow.config`, which
overrides `<script_dir>/nextflow.config`, which overrides
`$HOME/.nextflow/config`. Lastly, command line configuration overrides
configuration provided from files.

## Configuration scopes

Configuration can be organised into scopes using the curly bracket
notation or the dot notation. Configuration can also be included
from another file, which helps to include repeated configuration
settings in different scopes. Relative paths are resolved against
the actual path of the including config file, which helps packaging
configuration.

~~~
executor {
    name = 'local'
    // maximum cpus (applies to 'local' executor only)
    cpus = 4
    memory = 32.GB
}
// How many cpus a process should request.
process.cpus = 1

// Include configuration from foo
includeConfig 'path/foo.config'
~~~
{: .language-groovy}

Certain scopes are reserved to have special meaning.

### Scope *params*

The `params` scope allows one to define parameters accessible to
the workflow script.

Variables defined in the `params` scope are accessible from anywhere
in the script, but it is better practice to provide them via an input
declaration after having done appropriate checks on the input.
~~~
process echo {

    echo true

    script:
    """
    echo ${params.str}
    """
}
~~~
{: .language-groovy}

An example providing a default value in a configuration file.

~~~
params.str = 'Hello world'
~~~
{: .language-groovy}

Overriding the default value with a command line parameter.
~~~
nextflow run script.nf --str "Hello $USER"
~~~

### Scope *env*

The `env` scope defines variables to be exported in the execution
environment.

~~~
env {
    PATH = "/my/new/tool:$PATH"
    TOOL_LIB = "/my/new/tool/libs"
}
~~~
{: .language-groovy}


### Scope *process*

The `process` scope defines any property described in the
[documentation](https://www.nextflow.io/docs/latest/process.html#process-directives).

~~~
process {
    cpus = 1
    time = '1h'
    scratch = true
}
~~~
{: .language-groovy}

Some of the process configuration properties can accessed within a process
from the Task Configuration variable `task`.
<!-- FIXME: Which ones - where in the docs -->
~~~
process fastqc {

    input:
    tuple val(sample),path(reads) from read_fqc_ch

    script:
    """
    fastqc --threads ${task.cpus} $reads
    """
}
~~~
{: .language-groovy}

## Configuration profiles




{% include links.md %}
