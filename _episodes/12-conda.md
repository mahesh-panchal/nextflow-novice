---
title: "Supplementary: Package Managers - Conda"
teaching: 0
exercises: 0
questions:
- "Conda manages software on my computer. How can I use it?"
objectives:
- "Demonstrate how to use Nextflow with Conda."
keypoints:
- "The `conda` directive can be used to load a conda environment."
---

# Package Managers

By default, Nextflow expects all the commands or interpreters
to be available via the shell `PATH` environment variable.
However, many tools may have complex dependencies which can
cause conflicts with other tools in the same environment.
Nextflow supports several systems of package managers that
isolate tools and their dependencies into separated environments
preventing the majority of software conflicts.

Ideally, package management should be handled in the configuration
file rather than in the nextflow script. This allows users
to tailor software execution to their computing environment.

## Conda

[Conda](https://docs.conda.io/en/latest/) is package,
dependency and environment manager. Of particular interest is
the [Bioconda](https://bioconda.github.io/) channel which specialises
in bioinformatic software. Support for Conda is provided via the
`conda` directive in the `process` scope.

A user will often create an environment for themselves to use tools.
e.g.
~~~
# Create environment via commands
$ conda create -c bioconda -n blast_env blast=2.9.0

# Or create environment via yaml files
$ cat environment.yml
name: blast_env
channels:
  - conda-forge
  - bioconda
  - defaults

dependencies:
  - blast=2.9.0
$ conda env create -f environment.yml
~~~
{: .language-bash}

Both methods of environment creation are supported in Nextflow.
This will create the environment in the `conda.cacheDir` directory
(the default location is the `conda` folder in the working directory).
~~~
process {
    withName: 'blastn' {
        conda 'blast=2.9.0'
    }
    withName: 'blaster' {
        conda '/path/to/environment.yml'
    }
}
~~~
{: .source}

The use of existing environments is also supported by providing
the full path to the environment.
~~~
process {
    withName: 'blastn' {
        conda '/path/to/existing/conda/env'
    }
}
~~~
{: .source}

{% include links.md %}
