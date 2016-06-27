layout: true
class: inverse, middle

---
class: title
# Planemo
## ~~A Galaxy Tool SDK~~
## A Scientific Workflow SDK
John Chilton, Aysam Guerler, and the Galaxy Team

???

Oh look, I get to follow Björn again. That guy ruins the curve - when
it comes to pretty much any attempt to contribute to Galaxy.

---
class: large

### A Galaxy Philosophy

* The most important Galaxy user is the *bench scientist* using the GUI, they come first!
  * .smaller[No one wants to inconvenience a bioinformatician or developer, but we will if absolutely required for the biologist.]
* Galaxy workflows will *never require an SDK*.
  * .smaller[Planemo, workflow formats, etc... are conveniences for people who prefer developer processes.]

???

I'm going to get in trouble from a couple different sides with this talk, and I will state some strong, personal opinions. So I'm going to start with something that amounts to either a justification or a reassurance.

I'm pretty confident this amounts to a Galaxy team perspective - regardless
of how your define that team.

READ SLIDES

The first part of this talk will cover workflow enhancements from an end
user perspective in some ways though.

---

### GUI Enhancements - Workflow Editor Form

![Workflow Editor](images/gx_new_workflow_editor.png)

???

The editor form now uses the same backbone driven MVC components as the
new tool form presented last year.

---

### GUI Enhancements - Workflow Run Form

TODO: UPDATE IMAGE

![Workflow Run Form](images/gx_new_workflow_run.png)

???

The run workflow form has likewise been overhauled and will be merged soon. This
should allow more dynamic option control when running workflows.

---

### GUI Enhancements - Labels

TODO: UPDATE IMAGE

![Workflow Run Form](images/gx_new_workflow_run.png)

???

When reasoning about workflows and connections between steps, persistent and unique 
labels for steps and outputs are important. These are useful in the API and too a 
lesser extent in the GUI today.

A major theme of this presentation is going to be that workflows are programs, they
are a coding artifact. I'm not sure anyone would disagree with me on that - but I 
think the implications may be counter-intuitive at times.

---

### GUI Enhancements - Nested Workflows

TODO: UPDATE IMAGE

![Nested Workflows](images/gx_new_workflow_run.png)

???

Workflows are programs, languages describing programs should provide abstractions for
composition. Nesting workflows was one of the most requested feature requests of 
Galaxy and it now supports this.

---

### What About Planemo?

???

Enough screenshots right - when I present people expect to see long command-lines!

---

### Planemo's Success

.pull-left[

It is *the way* to develop Galaxy tools in 2016! Why?

- Artifact-centric - not Galaxy-centric or registry-centric.
- Works with existing developer tools - CLI, Git(hub), CI (Travis).
- Very flexible, easily configurable.
- Well documented with focus on *usage examples*.

It is about **developer workflow**.
]

.pull-right[

![Nemo](images/nemo.gif)

]

???

In 2014, Greg von Kuster presented a tool development workflow that involved 
publishing things to a local tool shed and running tests from there and viewing
the results through the web interface. I call this registry or shed centric tool
development (development activities "boot strap the tool shed, upload to the tool 
shed, run tests against the tool shed, view results in the tool shed, export capsule
from the tool shed". Prior to that tool development was Galaxy-centric - "download 
Galaxy, update the Galaxy tool conf, update the Galaxy test data, run the Galaxy 
tests." Planemo is tool centric - lint the tool, test the tool, serve the tool.

---

class: bottom
background-image: url(images/organic_mower_wat.jpg)
background: #FFFFFF
background-repeat: no-repeat

### So Planemo?

.photo-credit[Photo Credit:

*Peter Smith (@skwiot)*]

---

class: large

### Workflows are Programs

When I write programs...

* ... I write *tests* (and write them first)!
* ... I commit them to *Github*!
* ... use a text editor - *my* text editor!

???

The hippest, artisinal, free range text editor of my choice. It is a go 
lang rewrite of a node rewrite of an Erlang editor from 1987 - it is super hot right
now but I'm sure you've never heard of.

---
class: large

### Workflow Operations with Planemo


Serve them:

```
$ planemo serve <workflow.ga>
```

Brings up Galaxy interface with the workflow loaded, install shed 
tools as needed.

???

TODO: Install tool shed tools in .ga files, works for format 2 workflows.

---
class: large

### Load up local tools!

Serve them with tools...

```
$ planemo s --extra_tools <tool_dir>
            <workflow.ga>
```

Supply as many tools as you want.

---
class: large

### Workflows are harder to serve than tools

* Longer to setup input data
* Takes time to install shed tools
* `sqlite` database locks
* Cluster access is more important

---
class: large

### Planemo Profiles

Profile - a *persistent*, named Galaxy configuration available for serving, running
and testing across workflow invocations.

```
$ planemo profile_create <name>
$ planemo serve --profile <name> <workflow.ga>
```

---
class: large

### Planemo Profiles and Postgres

```
$ planemo profile_create --postgres <name>
$ planemo serve --profile <name> <workflow.ga>
```

Automatically *provision a postgres database* and configure this profile to use it. All
future `serve`, `run`, and `test` invocations using this profile will use this 
database.

???

Makes some reasonable guesses, but Postgres connection settings can be configured with
additional CLI options or in `~/.planemo.yml`.

---
class: large

### Planemo Profiles and Clusters

```
$ planemo profile_create --job_conf <job_conf.xml>
                         <name>
```

Setup a job configuration object for this profile and use it with all future
`serve`, `run`, and `test` invocations using this profile.

???

TODO: Implement slurm.

---
class: large

### Planemo Docker Profiles

Or just...

.slightly-smaller[```
$ planemo profile_create --engine_type docker_galaxy
                         <name>
```]

Leverage the `docker-galaxy-stable` project spearheaded by the 
Björn Grüning to `serve` and `run` using Galaxy in a Docker container.

This container comes pre-configured with slurm and Postgres as available locally -
but offers lots of additionals goodies such as Condor, ProFTP, supervisor, and uwsgi.

https://github.com/bgruening/docker-galaxy-stable

???

Leverage the omnipresent `docker-galaxy-stable` community project spearheaded by the 
omnipresent Björn Grüning to `serve` and `run` using Galaxy in a Docker container.


---
class: large

### Galaxy Workflow Format

Doesn't Galaxy have a JSON workflow format?

```json
"tool_state": "{\"__page__\": 0, \"__rerun_remap_job_id__\": null,
\"input1\": \"null\", \"chromInfo\": \"\\\"/home/john/workspace/galaxy-central/tool-data/shared/ucsc/chrom/?.len\\\"\",
\"queries\": \"[{\\\"input2\\\": null, \\\"__index__\\\": 0}]\"}",
```

- Neither human writable, nor human readable.
- JSON doesn't allow comments.
  - One shouldn't have to describe a configuration file in JSON,
    let alone write a program in it.

---
class: smaller

### Format 2 Workflows - Example

```yaml
class: GalaxyWorkflow
name: "Test Workflow"
tools:
  - name: text_processing
    owner: bgruening
inputs:
  - id: input1
outputs:
  - id: wf_output_1
    source: sort#outfile
steps:
  - id: sed
    tool_id: tp_sed_tool
    state:
      infile:
        $link: input1
    code: "s/ World//g"
  - id: sort
    tool_id: tp_sort_header_tool
    state:
      infile: sed#output
      style: h  #Human readable
```

---
class: large

### Another Aside - gxformat2

```
pip install gxformat2
```

`gxformat2` is a Python library for the conversion of "Format 2" workflows.

- Started as a way to build test workflows for Galaxy testing framework.
- All steps can be labeled, connections described by ID.
  - Pypi @ https://pypi.python.org/pypi/gxformat2/
  - Github @ https://github.com/jmchilton/gxformat2

---
class: large

### Another Aside - Ephemeris

```
pip install emphemeris
```

`emphemeris` is an opinionated Python library and scripts for bootstrapping Galaxy tools, index data, and worklows.

- Scripts from `ansible-galaxy-tools` by Marius van den Beek, Enis Afgane, Björn Grüning, and others.
- Pypi @ https://pypi.python.org/pypi/emphemeris/
- Github @ https://github.com/galaxyproject/emphemeris

---
class: large

### Format 2 Workflows - Composition Example

TODO: Show a subworkflow example.

---

### Format 2 Workflows - Implicit Connections Example

TODO: Show a data manager example.

---
class: large

### Testing Workflows

```
$ planemo test [--profile <name>] <workflow>
```

* Same great HTML and other formatting options as tools.
* Produce sharable test result link with `planemo share_test`.
* Test either Galaxy native or Format 2 workflows.

---
class: large
  
### Generalized Test Format

Test any aritfact (Galaxy Tool, Galaxy Workflow, CWL Tool, CWL 
Workflow) - using the same YAML-based scheme and format.

If workflow is `my_workflow.ga`, place an aritfact named
`my_workflow-test.yml` in the same directory.

```
planemo test my_workflow.ga
```

Will detect this aritfact and run the tests in this YAML file.

---

### Run Workflows

---

class: large, bottom
layout: false
background-image: url(images/CWL-Logo-HD.png)
background-repeat: no-repeat
background-size: contain

### .

---

layout: true
class: inverse, middle

---
class: large

### Common Workflow Language

* http://www.commonwl.org/
* Group of (and specifications by) engineers
  * Formed at 2014 BOSC Codefest.
  * After 4 draft iterations, 1.0 will be released next week.
* Like Galaxy, the rare open infrastructure that crosses the Atlantic.
  * Adopted by various Elixer efforts, Seven Bridge Genomics,
    and will be implemented in Taverna.
  * Will be supported on all NIH Cancer Cloud Pilots and endorsed by the 
    GA4GH. Reference implementation `cwltool` developed at Curoverse.
* Poster P06 at 3:15 by Michael Crusoe

---
class: large

### CWL & Galaxy

*Experimental* tool support today using planemo.

```
$ planemo serve --cwl <tool.cwl>
```

When `serve`, `test`, `run` encounter CWL tools they will use a Galaxy fork.

Work in progress at https://github.com/common-workflow-language/galaxy.

---

class: large

### Planemo Engine Type `cwltool`

With `--engine_type=cwltool` (set default in `~/.planemo.yml`), one can
`run` and `test` both CWL tools and workflows.

---

### CWL and `tool_init` (1 / 2)

```shell
$ planemo tool_init --cwl \
                    --id 'seqtk_seq' \
                    --name 'Convert to FASTA (seqtk)' \
                    --example_command \
                        'seqtk seq -a 2.fastq > 2.fasta' \
                    --example_input 2.fastq \
                    --example_output 2.fasta \
                    --container 'dukegcb/seqtk' \
                    --test_case \
                    --help_from_command 'seqtk seq'
```

---

class: smaller

### CWL and `tool_init`  (2 / 2)

.pull-left[
```yaml
#!/usr/bin/env cwl-runner
cwlVersion: 'cwl:draft-3'
class: CommandLineTool
id: "seqtk_seq"
label: "Convert to FASTA (seqtk)"
requirements:
  - class: DockerRequirement
    dockerPull: dukegcb/seqtk
inputs:
  - id: input1
    type: File
    description: TODO
    inputBinding:
      position: 1
      prefix: "-a"
outputs:
  - id: output1
    type: File
    outputBinding:
      glob: out
baseCommand: ["seqtk", "seq"]
arguments: []
stdout: out
description: |
  
  Usage:   seqtk seq [options] <in.fq>|<in.fa>
  ...
```
]

.large[.pull-right[
Generates:
- `seqtk_seq.cwl`
- `seqtk_seq-tests.yml`
- `test-data/2.fasta`
- `test-data/2.fastq`
]]

---
class: large


### CWL and `lint`

```
$ planemo l seqtk_seq.cwl
Linting tool /opt/tools/seqtk_seq.cwl
Applying linter general... CHECK
.. CHECK: Tool defines a version [0.0.1].
.. CHECK: Tool defines a name [Convert to FASTA (seqtk)].
.. CHECK: Tool defines an id [seqtk_seq_v3].
Applying linter cwl_validation... CHECK
.. INFO: CWL appears to be valid.
Applying linter docker_image... CHECK
.. INFO: Tool will run in Docker image [dukegcb/seqtk].
Applying linter new_draft... CHECK
.. INFO: Modern CWL version [cwl:draft-3]
```

---
class: large

### CWL and `test`

```
$ planemo t --engine_type=cwltool \
            seqtk_seq.cwl
```

Tool test output HTML is produced in the file `tool_test_output.html`.