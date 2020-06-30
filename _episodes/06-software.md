---
title: "Software Package Managers"
teaching: 0
exercises: 0
questions:
- "Key question (FIXME)"
objectives:
- "First learning objective. (FIXME)"
keypoints:
- "First key point. Brief Answer to questions. (FIXME)"
---

## Software package managers

The basic premise of Nextflow is to chain lots of different tools
together. By default, Nextflow expects all the commands or interpreters
to be available via the shell `PATH` environment variable.
However, many tools may have complex dependencies which can
cause conflicts with other tools in the same environment.
Nextflow supports several systems of package managers that
isolate tools and their dependencies into separated environments
preventing the majority of software conflicts.

Ideally, package management should be handled in the configuration
file rather than in the nextflow script. This allows users
to tailor software execution to their computing environment.

## Environment Modules

[Environment Modules](http://modules.sourceforge.net/)
is a package manager that loads tools via
the `module load <package>` command. Modules are supported in
Nextflow via the `module` directive in the `process` scope.

~~~
process blast {

    module 'ncbi-blast/2.9.0'

    """
    blast -version
    """
}
~~~
{: .source}

The `module` directive can also be assigned in a config file:
~~~
process {

    // available to all processes
    module = 'cluster-utils/1.2.3'

    // Override the module directive above for a specific process
    withName: blast {
        module = 'ncbi-blast/2.9.0:gnu-parallel/3.5'
    }
}
~~~
{: .source}

Multiple packages can be loaded at the same time by separating the
package names with a colon (`:`).

Note that environment modules are often centrally managed (e.g. by
cluster administrators) which may limit tools available to the user.

## Conda

[Conda](https://docs.conda.io/en/latest/) is another package, dependency and environment manager. Of particular interest is the
[Bioconda](https://bioconda.github.io/) channel which specialises
in bioinformatic software. Support for Conda is provided via the
`conda` directive in the `process` scope.

A user will often create an environment for themselves to use tools.
e.g.
~~~
# Create environment via commands
$ conda create -n blast_env blast=2.9.0

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

Both methods are supported in Nextflow:
~~~
process blastp {

    conda 'blast=2.9.0'

    """
    blastp -version
    """
}

process blastn {

    conda '/path/to/environment.yml'

    """
    blastn -version
    """
}
~~~
{: .source}

This will create the environment in the `conda.cacheDir` directory
(the default location is the `conda` folder in the working directory).

The use of existing environments is also supported by providing
the full path to the environment.
~~~
process blastp {

    conda '/path/to/existing/conda/env'

    """
    blastp -version
    """
}
~~~
{: .source}

The `conda` directive can also be used in the config file.
~~~
process {

    // available to all processes
    conda = 'gnu-parallel=3.5'

    // Override the conda directive above for a specific process.
    withName: blastn {
        conda = 'blast=2.9.0 gnu-parallel=3.5'
    }
}
~~~~
{: .source}

## Docker

[Docker](https://www.docker.com/) is platform that provides a
standardised packaging format known as container images. A container
image is a unit of software that packages up
code and all its dependencies so the application runs the same regardless
of the underlying infrastructure. The [Docker Engine](https://www.docker.com/products/container-runtime) needs to be installed to run
Docker container images on your computer infrastructure.

Container images are built according to recipes prescribed in a
`Dockerfile`. A base image is used a starting point, which could
be an operating system image, or an pre-defined image with other tools
preinstalled. Additional instructions then add commands to run,
set environment variables, entry points, and other metadata.

An example `Dockerfile`:
~~~
FROM continuumio/miniconda3:4.8.2

SHELL ["/bin/bash", "-c"]

LABEL description="Spade (Search for Patterned DNA Elements) container" \
      author="Mahesh Binzer-Panchal" \
      version="1.0.0"

RUN apt-get update --fix-missing && \
    apt-get install -y procps ghostscript

RUN conda update -n base conda && \
    conda install -c conda-forge -c bioconda \
	python=3.6 mafft=7.455 blast=2.9.0 openssl=1.1.1e && \
    conda clean --all -f -y

WORKDIR /opt
RUN git clone --depth 1 https://github.com/yachielab/SPADE && \
    cd SPADE && chmod u+x *.py && \
    pip install matplotlib==2.2.3 && \
    pip install seaborn==0.8.1 && \
    pip install weblogo==3.6.0 && \
    pip install biopython

ENV PATH="/opt/SPADE:${PATH}"

CMD [ "SPADE.py" ]
~~~
{: .source}

When a container image is built, it is stored in a repository, either
locally or online. Nextflow is able to retrieve container images for use
using the `docker pull` instruction at the location
(`'docker-repository/image-name:tag'`) provided by the
`container` directive in the `process` scope. Images should preferably
be stored in an online repository to enable access for others.

~~~
process blastp {

    container 'quay.io/biocontainers/blast:2.9.0--pl526h3066fca_4'

    """
    blastp -version
    """
}
~~~
{: .source}

As with other software package managers, containers can also
be defined in the config file. However, an additional setting
is needed to use the Docker engine.
~~~
docker {
    enabled = true
}
process {

    // available to all processes
    container = 'quay.io/biocontainers/blast:2.9.0--pl526h3066fca_1'

    // Override the container directive above for a specific process.
    withName: blastn {
        container = 'quay.io/biocontainers/blast:2.9.0--pl526h3066fca_4'
    }
}
~~~
{: .source}

## Singularity


{% include links.md %}
