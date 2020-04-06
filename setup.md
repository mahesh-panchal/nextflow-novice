---
title: Setup
---

# Overview

Nextflow can be used on any POSIX compatible system (Linux, OS X, etc).

## The terminal

Much of the lesson content relies on executing various command-line instructions
and so a bash terminal window needs to be available to you.

On Unix and Mac OS X, this is already available.

On Windows, assuming you have 64-bit Windows 10, go to
`Control Panel > Programs > Turn Windows Features On Or Off` and enable
"Windows Subsystem for Linux" and click "OK". Click "Restart now" to reboot
your computer. Open the Microsoft Store from the Start Menu, and search for
"Linux". Then click on the "Get the apps" under the "Linux on Windows?" banner.
Ubuntu is the recommended flavour of Linux environment to install.

## A file editor

Scripting is another important aspect of this lesson and therefore you will need
an editor available to you. There are several options available to you, so here are
are a few suggestions:
* Command-line:
  * nano: A basic file editor, no syntax highlighting.
  * vim: A powerful file editor, potentially difficult to use, syntax highlighting
available through vim plug-ins.
* Graphical
  * [atom](https://atom.io/): Easy to use, supports git, syntax highlighting available
though atom package manager.

## Nextflow

In order to run Nextflow workflows, Nextflow must be installed on your system.

Here are two options for installation:
- Option A: Direct installation.
- Option B: Installation via the Conda package manager.

### Option A: Direct installation

Open a terminal and execute the following command:
```bash
curl -s https://get.nextflow.io | bash
```

### Option B: Installation via Conda

Open the terminal and create an environment for Nextflow.
```bash
conda create -n nextflow-env nextflow
```

The conda environment can then be activated and deactivated using:
```bash
conda activate nextflow-env
conda deactivate
```

## Graphviz (optional)

Graphviz is a package of open-source tools for drawing graphs specified in DOT language scripts. This is only used for graphing the workflow task execution, and is not a necessity for the operation of Nextflow.

# Other services (workshop dependent)

These services are only needed for select supplementary material.

## Git

## Docker

* Docker engine 1.10.x (or later)

## Singularity

* Singularity 2.5.x (or later, optional)

## Conda

* Conda 4.5 (or later, optional)

## Job Scheduler

## AWS Batch services

## Google services


{% include links.md %}
