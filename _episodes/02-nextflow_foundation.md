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

Nextflow is a Domain Specific Language (DSL) designed to ease
the writing of computational pipelines. It is an extension of the
computer language Groovy, which is a superset of the computer
language Java. As such, Nextflow can execute any piece of Groovy
code or use any library for the JVM platform.

This section describes how to write and run a nextflow workflow.

## Writing a workflow.

Here are some fundamentals of Groovy scripting.

- Variables are assigned to using `=` and can have any value.
    ```groovy
	var = 1
	var = -3.1499392
	var = false
	var = "Hello world"
	var = new java.util.Date()
	```
- Lists are defined using the square bracket `[]` notation.
	```groovy
	emptylist = []                   // an empty list
	// Lists can contain duplicates, and the values can be of any type.
	mixedList = [1, 1, 'string', true, null, 5 as byte]
	mixedList.add("new value")       // adds "new value" to the end of mixedList
	println mixedList.size()         // prints the size
	println mixedList[0]             // prints 1
	```
- Maps ( also known as associative arrays ) are defined using the `[:]` literal. They associate a unique string with a value, and are commonly referred to as key-value pairs.
    ```groovy
	emptyMap = [:]                   // an empty map
	mymap = [ name : "Steve", age: 43, likes: ['walks','cooking','coding']]
    println mymap['name']            // access values using quoted key.
	```

A nextflow script has two main components, processes and channels.

Example (`example.nf`):
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

A process is a task that executes a user script on a single input set. The
script can be written in any computer language, although the default
is bash.


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
