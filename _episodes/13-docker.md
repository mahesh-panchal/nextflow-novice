---
title: "Supplementary: Package Managers - Docker"
teaching: 0
exercises: 0
questions:
- "How can I use software from docker containers?"
objectives:
- "Demonstrate how to use Nextflow with Docker."
keypoints:
- "Docker must be enabled using `docker.enabled=true` in the configuration."
- "The `container` directive can be used to load a docker image."
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

## Docker

[Docker](https://www.docker.com/) is platform that provides a
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
$ docker run --rm --name fastqc quay.io/biocontainers/fastqc:0.11.9--0
~~~
{: .language-bash}

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
using the `docker pull` instruction at the path
(`'docker-repository/image-name:tag'`) provided by the
`container` directive in the `process` scope. Images should preferably
be stored in an online repository to enable access for others.

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

{% include links.md %}
