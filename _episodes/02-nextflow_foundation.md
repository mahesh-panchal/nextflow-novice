---
title: "Nextflow foundation"
teaching: 30
exercises: 0
questions:
- "How to I write a nextflow script"
- "How can I run a nextflow script"
objectives:
- "Write a simple nextflow workflow"
- "Run a simple nextflow workflow"
keypoints:
- FIXME
---

# Nextflow foundation

This section describes how to write and run a nextflow workflow.

## Writing a workflow.

Nextflow workflows are written in the computer language Groovy.

A nextflow script has two main components, processes and channels.

Example (`example.nf`
```nextflow
#! /usr/bin/env nextflow

number_ch = Channel.from(1,2,3,4)

process echo {

    input:
    val number from number_ch

    output:
    stdout into out_ch

    script:
    """
    echo "Number: $number"
    """

}

out_ch.view()
```

A Channel is a structure that passes data asynchronously from one process to another.

A process is a task that executes on a single input set.

## Running a workflow.

```bash
$ nextflow run test.nf
N E X T F L O W  ~  version 20.01.0
Launching `example.nf` [naughty_ramanujan] - revision: d06de0a119
executor >  local (4)
[01/cf5b42] process > echo [100%] 4 of 4 ✔
Number: 2

Number: 1

Number: 3

Number: 4
```

{% include links.md %}
