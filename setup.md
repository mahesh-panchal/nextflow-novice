---
title: Setup
---

# Overview

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
an editor available to you. There are several options available to you:
* Command-line:
  * nano: A basic file editor, no syntax highlighting
  * vim: A powerful file editor, potentially difficult to use, syntax highlighting
available through vim plug-ins.
* Graphical
  * [atom](https://atom.io/): Easy to use, supports git, syntax highlighting available
though atom package manager.

## Nextflow

In order to run nextflow workflows, nextflow must be installed on your system.

You can use either:
- Option A: Direct installation
- Option B: Installation via the Conda package manager

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

The conda environment can then be activated using:
```
conda activate nextflow-env
```

## Git

## Container software

Docker and Singularity are two popular container technologies supported by Nextflow.

### Installing Docker

### Installing Singularity


{% include links.md %}
