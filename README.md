Resources and Dockerfiles for reproducing Red Hat case #04011614. It basically
contains two nearly identical Pipelines that only differ in task *buildah*.

**pipeline-with-clustertask** references ClusterTask *buildah*.

**pipeline-with-task** references Task *buildah* from namespace
*openshift-pipeline* by means of a cluster resolver.

[This blog
article](https://www.redhat.com/en/blog/migration-from-clustertasks-to-tekton-resolvers-in-openshift-pipelines)
suggests that replacing the usage of ClusterTasks is just a matter of replacing
the corresponding code by code referencing a Task by means of a cluster
resolver. However, at least Task *buildah* shipped with recent versions of
OpenShift Pipelines is not a 100% drop-in replacement for the corresponding
ClusterTask *buildah*.


## Prerequisites

* Red Hat OpenShift Cluster v4.16
* Red Hat OpenShift Pipelines v1.16.0 installed. (Any version that ships with
  both ClusterTasks and Tasks should do.)

## Setup

* Create (or choose) a namespace to work with.

* Fetch the code from `https://github.com/makuhama/case04011614.git`. (Probably
  you have done so already).

* Create all required resources:

```
oc apply -f target-imagestream.yaml
oc apply -f pipeline_with_clustertask/pipeline_with_clustertask_pipeline.yaml
oc apply -f pipeline_with_task/pipeline_with_task_pipeline.yaml
```

* Adapt the two YAML files:

```
pipeline_with_clustertask/pipelinerun_with_clustertask_pipelinerun.yaml
pipeline_with_task/pipelinerun_with_task_pipelinerun.yaml
```

by changing the namespace in line 18.

## Good case

Run Pipeline *pipeline_with_clustertask* with

```
oc create -f pipeline_with_clustertask/pipelinerun_with_clustertask_pipelinerun.yaml
```

This PipelineRun succeeds

## Bad case

Run Pipeline *pipeline_with_task* with

```
oc create -f pipeline_with_task/pipelinerun_with_task_pipelinerun.yaml
```

This PipelineRun fails.

## Findings

ClusterTask *buildah* (now deprecated as all ClusterTasks) sticks to the
semantics of the `buildah` tool where the Dockerfile (Containerfile) given by
option `--file / -f` is first search for relative to the current directory and
then relative to the context (i.e. the directory containing all the input for
the build). (See
[documentation](https://github.com/containers/buildah/blob/main/docs/buildah-build.1.md).)

This second option is not available with Task *buildah* as shipped with
OpenShift Pipelines.
