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

Certain [scopes](https://www.nextflow.io/docs/latest/config.html#config-scopes) are reserved to have special meaning. Only some are
described here.

### Scope *params*

The [`params` scope](https://www.nextflow.io/docs/latest/config.html#scope-params) allows one to define parameters accessible to
the workflow script.

~~~
params {
    str = "Hello"
    fasta = '/path/to/fasta'
}
~~~
{: .language-groovy}

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

Channel.fromPath(params.fasta, checkIfexists: true).set { fa_ch }

process index_fasta {

    input:
    path fasta from fa_ch

    script:
    """
    samtools faidx $fasta
    """
}
~~~
{: .language-groovy}

As described above, the configured value can be overriden with
a command line parameter, or using another configuration file provided
with `-c`.

~~~
nextflow run script.nf --str "Hello $USER"
~~~
{: .language-bash}

### Scope *env*

The [`env` scope](https://www.nextflow.io/docs/latest/config.html#scope-env) defines variables to be exported in the execution
environment.

~~~
env {
    PATH = "/my/new/tool:$PATH"
    TOOL_LIB = "/my/new/tool/libs"
}
~~~
{: .language-groovy}


### Scope *process*

The [`process` scope](https://www.nextflow.io/docs/latest/config.html#scope-process) defines any property described in the
[Process directives documentation](https://www.nextflow.io/docs/latest/process.html#process-directives).

~~~
process {
    cpus = 1
    time = '1h'
    scratch = true
}
~~~
{: .language-groovy}

The selectors `withName: <process_name>` and `withLabel: <label>`
can be used to provide configurations to specific processes.

~~~
process {
    withName: 'index_fasta' {
        cpus = 1
    }
    withLabel: 'bigMem' {
        memory = 256.GB
    }
}
~~~
{: .language-groovy}

[Selector expressions](https://www.nextflow.io/docs/latest/config.html#selector-expressions) can be used to group process
selections or negate them.

### Scope *manifest*

The [`manifest` scope](https://www.nextflow.io/docs/latest/config.html#scope-manifest) provides metadata for your workflow.

~~~
manifest {
    homePage = 'http://foo.com'
    description = 'Pipeline does this and that'
    mainScript = 'foo.nf'
    version = '1.0.0'
}
~~~
{: .language-groovy}

## Configuration profiles

[Configuration profiles](https://www.nextflow.io/docs/latest/config.html#config-profiles) define predefined configuration
settings to be used with the workflow, enabling workflow portability.
Profiles can include how to manage software, executor settings, or
computer or institute specific settings, and multiple profiles
can be used together to provide flexibility of use.

~~~
profiles {

	/*
	<profile_name> {
		<configuration scope1>
		<configuration scope2>
		...
	}
	*/

	// default profile
	standard {
	}

	hal_9000 {
		process {
			cpus = 1000
		}
	}

	laptop {
		process {
			cpus = 1
		}
	}
}
~~~
{: .language-groovy}

Profiles are used directly on the command line:
~~~
nextflow run -profile <profile1>[,<profile2>,...]` <nextflow_script>
~~~
{: .language-bash} 

## Executor configurations



{% include links.md %}
