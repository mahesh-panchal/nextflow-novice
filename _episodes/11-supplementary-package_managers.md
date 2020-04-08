---
title: "Supplementary - Nextflow and Package Managers"
teaching: 0
exercises: 0
questions:
- "Key question (FIXME)"
objectives:
- "First learning objective. (FIXME)"
keypoints:
- "First key point. Brief Answer to questions. (FIXME)"
---

# Package Managers

By default, Nextflow expects all the commands or interpreters
to be available via the shell `PATH` environment variable.
However, many tools may have complex dependencies which can
cause conflicts with other tools in the same environment.
Nextflow supports several systems of package managers that
isolate tools and their dependencies to separated environments
preventing the majority of conflicts.

## Module system

The module system is a package manager that loads tools via
the `module load <package>` command. This is supported in
nextflow through the `module` directive.

```groovy
process blast {

	module 'ncbi-blast/19.0.5'

	"""
	blast -version
	"""
}
```

This directive can also be set in a config file:
```
process {
	withName: blast {
		module = 'ncbi-blast/19.0.5:gnu-parallel/3.5'
	}
}
```

Multiple packages can be loaded at the same time by separating the
package names with a colon (`:`).

Note that the module system is often managed by the computer
administrators which may limit what is available to the user.

## Conda

## Docker

## Singularity


{% include links.md %}
