---
layout: reference
---

## Reference

## [Introduction to workflows]({{page.root}}/01-workflow-intro)

* A workflow is a series of tasks to process data.
* A workflow framework is a set of tools that allows you to
compose tasks, connect them together, and process data.
* There are many workflow frameworks available:
  * Shell scripts
  * Nextflow
  * SnakeMake
  * Galaxy
  * ...
* Advantages of Nextflow
  * Reproducible (when combined with container and version control technology).
  * Portable (Ideally can be run anywhere - locally, grid/hpc, or cloud).
  * Simple syntax but exensible.
  * Functional logic can be separated from execution engine.
  * Reentrancy.
  * Open source.
  * Rapidly increasing community.

## [Nextflow syntax]({{page.root}}/02-nextflow-syntax)

* There are two fundamental data structures:
  * Channels
  * Processes
* A Channel is a dataflow object, of which there are two types:
  * Value channels - elements are reusable.
  * Queue channels - elements are consumed (single use).
* A Process is an encapsulated task - a set of instructions that must be
executed together.

## Glossary


{% include links.md %}
