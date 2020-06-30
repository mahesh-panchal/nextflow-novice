---
title: "Software Package Managers"
teaching: 0
exercises: 0
questions:
- "How is software managed by nextflow?"
objectives:
- "Learn how to leverage package managers to handle software requirements"
keypoints:
- "Commands are expected to be available from the shell PATH."
- "Environment modules can provide centrally managed software environments."
- "Conda can provide user managed software environments"
- "Container platforms can provide self-contained software environments."
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

> ## Reproducibility
>
> Software results can vary depending depending on the execution
> environment. In order to ensure reproducibility and cross-plaform
> compatibility across many infrastructures, the use of container
> technology is recommended. In the absence of a suitable container
> technology, conda is recommended to manage software installation
> and dependancies. Although modules and self installed tools are
> supported, they do not enable ease of use across platforms,
> creating a potentially high barrier to using the workflow.
{: .callout}

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

[Docker](https://www.docker.com/) is container platform that provides a
standardised packaging format known as container images. A container
image is a unit of software that packages up code and all its
dependencies so the application runs the same regardless
of the underlying infrastructure. The
[Docker Engine](https://www.docker.com/products/container-runtime)
needs to be installed to run Docker container images on your
computer infrastructure.

Typically, images are run as containers in which your commands
can be executed.
~~~
# Start a container based on the image `docker run <image>`
# Run a command `fastqc --version`
# Remove the container on completion `--rm`
$ docker run --rm quay.io/biocontainers/fastqc:0.11.9--0 fastqc --version
FastQC v0.11.9
~~~
{: .language-bash}

Container images are built according to recipes prescribed in a
`Dockerfile`. A base image is used as starting point, which could
be an operating system image, or a pre-defined image with other tools
preinstalled (e.g. miniconda as seen below). Additional instructions
then add commands to run,
set environment variables, entry points, and other metadata.

An example `Dockerfile`:
~~~
# Select the miniconda image as the base
# https://hub.docker.com/r/continuumio/miniconda3/dockerfile
FROM continuumio/miniconda3:4.8.2

# Select the shell to use.
SHELL ["/bin/bash", "-c"]

# Add metadata to the container using labels.
LABEL description="Spade (Search for Patterned DNA Elements) container" \
      author="Mahesh Binzer-Panchal" \
      version="1.0.0"

# APT (Advanced Packaging Tool) is package manager for certain linux distributions
# It can be used to update and install software dependencies
RUN apt-get update --fix-missing && \
    apt-get install -y procps ghostscript

# Conda is another package manager that simplifies package installation
# This is a useful option for installing bioinformatic packages
RUN conda update -n base conda && \
    conda install -c conda-forge -c bioconda \
	python=3.6 mafft=7.455 blast=2.9.0 openssl=1.1.1e && \
    conda clean --all -f -y

# Some tools are not available via package managers.
# These must then be manually installed.
WORKDIR /opt
RUN git clone --depth 1 https://github.com/yachielab/SPADE && \
    cd SPADE && chmod u+x *.py && \
    pip install matplotlib==2.2.3 && \
    pip install seaborn==0.8.1 && \
    pip install weblogo==3.6.0 && \
    pip install biopython==1.76

# Environment variables can be set to provide settings for new tools.
ENV PATH="/opt/SPADE:${PATH}"

# A default command can be provided when a container is started (entry point).
CMD [ "SPADE.py" ]
~~~
{: .source}

When a container image is built, it is stored in a registry, either
locally or online. Nextflow is able to retrieve these container images
when provided with a path to the image and version (tag)
(`'docker-repository/image-name:tag'`) given by the
`container` directive in the `process` scope. Images should preferably
be stored in an online repository to enable access for others.

> ## Docker Registries
>
> Here are some useful Docker registries:
> - Docker Hub: [https://hub.docker.com/](https://hub.docker.com/)
> - Red Hat Quay: [https://quay.io/](https://quay.io/)
> - Biocontainers (Bioconda images): [https://biocontainers.pro/#/](https://biocontainers.pro/#/)
>
> Github also supports hosting Docker images using Github packages.
{: .callout}

An additional `docker` scope is provided by Nextflow
which allows you to supply extra parameters to Docker. In order
to use a container image with Docker, it must be enabled.   
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

> ## Default user
>
> When running a container using Docker, the default docker user is used
> to run the process. This is the same user
> with which the image was built (default: `root`). For life science
> projects it is rare that tools need to be run with superuser
> privileges, and often one wants to run tools using the current
> user. It is helpful to add `runOptions` to the docker configuration
> scope like so:
>
> ~~~
> docker {
>     enabled = true
>     // Uses `id` to get the current user id and group id of the user
>     runOptions='-u "$( id -u ):$( id -g )"'
> }
> ~~~
> {: .language-groovy}
>
> Docker images run using other container platforms use the settings
> prescribed by the other container platform. For example, Singularity
> will use the current user as the default user.  
{: .callout}

> ## References
>
> - Nextflow Docker integration: [https://www.nextflow.io/docs/latest/docker.html](https://www.nextflow.io/docs/latest/docker.html)
> - Nextflow Docker Configuration: [https://www.nextflow.io/docs/latest/config.html#scope-docker](https://www.nextflow.io/docs/latest/config.html#scope-docker)
> - Docker Documentation: [https://docs.docker.com/](https://docs.docker.com/)
> - Docker Best Practices: [https://docs.docker.com/develop/dev-best-practices/](https://docs.docker.com/develop/dev-best-practices/)
> - Dockerfile reference: [https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/)
{: .callout}

## Singularity


{% include links.md %}
