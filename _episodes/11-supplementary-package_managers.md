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
isolate tools and their dependencies into separated environments
preventing the majority of conflicts.

Ideally, package management should be handled in the configuration
file rather than in the nextflow script. This allows users
to tailor execution to their computing environment.

## Environment Modules

[Environment Modules](http://modules.sourceforge.net/)
is a package manager that loads tools via
the `module load <package>` command. Modules are supported in
Nextflow via the `module` directive in the `process` scope.

```groovy
process blast {

	module 'ncbi-blast/2.9.0'

	"""
	blast -version
	"""
}
```

The `module` directive can also be assigned in a config file:
```
process {

	// available to all processes
    module = 'cluster-utils/1.2.3'

	// Override the module directive above for a specific process
	withName: blast {
		module = 'ncbi-blast/2.9.0:gnu-parallel/3.5'
	}
}
```

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
```bash
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
```

Both methods are supported in Nextflow:
```groovy
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
```

This will create the environment in the `conda.cacheDir` directory (the default location is the `conda` folder in the working directory).

The use of existing environments is also supported by providing
the full path to the environment.
```groovy
process blastp {

	conda '/path/to/existing/conda/env'

	"""
	blastp -version
	"""
}
```

The `conda` directive can also be used in the config file.
```
process {

	// available to all processes
    conda = 'cluster-utils/1.2.3'

	// Override the conda directive above for a specific process.
	withName: blastn {
		conda = 'blast=2.9.0 gnu-parallel=3.5'
	}
}
```


## Docker

## Singularity


{% include links.md %}
