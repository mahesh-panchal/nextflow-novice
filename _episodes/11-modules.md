---
title: "Supplementary: Package Managers - Environment modules"
teaching: 0
exercises: 0
questions:
- "Environment modules manage software on my computer. How can I use them?"
objectives:
- "Demonstrate how to use Nextflow with Environment modules."
keypoints:
- "The `module` directive can be used to load a module environment."
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

{% include links.md %}
