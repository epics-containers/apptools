# apptools
Scripts for deploying applications to a shared filesystem using module and apptainer

## Introduction

These scripts provide a mechanism for adding command line utilities to the path. This is DLS specific at present but could be generalized.

Any application that has been published in a container registry is supported. The container is pulled and converted to an Apptainer sif file. The sif file is then executed by Apptainer. The sif file is kept as a cache for future executions.

The intention is that the sif files will be in shared location to minimize repeating pulls from container registries. This shared location could be /scratch/30day_tmp or an NFS mount. I have tested using /dls_sw and it is a little slower but not quite as slow as loading our python projects from /dls_sw.

## How to use

In future this will be incorporated into the global modules. I don't have write permission to /dls_sw/apps/Modules which is where this would go in the long run. In the mean time I have made the scripts create a private module as a demo.

Do these steps:

```
cd -somewhere in your scratch drive-
git clone git@github.com:epics-containers/apptools
./apptools/module

module load apptools
```

Now you can run the one thing we have installed as a demo (first time will be a delay while the sif gets made)

```
dls-pmac-control --help
```

## How to add a new tool

- Get your project published in a container somehow.
- For python just use the copier project and choose the container option
  - https://github.com/DiamondLightSource/python-copier-template
- add a file in ./apptools/apptools with the name of your project
- chmod +x your project file
- add the following contents into the file (example is dls-pmac-control)
- you can pass args to apptainer via ARGS
- you can pass args to the command at the end of the command line
- use "${@}" to pass extra args at runtime invocation


### example for dls-pmac-control
```
#!/bin/bash
export ARGS="-e" # remove environment so Qt does not try to contact DBus
apptools-launch ghcr.io/diamondlightsource dls-pmac-control 3.2.0b1 ENTRYPOINT_COMMAND "${@}"
```

### example for ubuntu bash shell
```
#!/bin/bash

# bash args: don't run the host's bashrc
apptools-launch docker.io/library ubuntu latest bash --noprofile  --norc "${@}"

```