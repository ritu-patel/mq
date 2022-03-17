# Build a custom MQ image

<img src="/readme-images/custom-image.png" width="55%" height="20%">

**Description**: Purpose of this repo is to build a custom MQ image every new release or every fix pack release. This repo contains dockerfile and security exit files. The base image in Dockerfile is using MQ version `9.2.4.0-r1`. Make sure you update the version every new release and run the pipeline to build and push the image to Nexus registry. Pipeline also runs smoke test and performance test.

- Suggested Access Groups for this repo: Admins only
- Smoke test: In this repo, there is a file named `release.yaml` which is used for smoke test. It deploys an ephemeral single instance QueueManager called "qm1-smoke". In `tekton/tasks/2-smoke-test.yaml`, you can see that it applies `release.yaml` and performs health check by putting/getting messages in a sample queue.
- Perfomance test: For performance test, it is running `cphtestp` on `qm1-smoke` queue manager created by smoke-test task. You will also have to grant [scc permissions](https://github.com/ibm-messaging/cphtestp/blob/master/openshift/openshift.md#grant-scc-permissions) for pods to be able to write to some temporary storage. More details on `cphtestp`: https://github.com/ibm-messaging/cphtestp/blob/master/openshift/openshift.md

Note: If you are not planning to use performance test in your pipeline then edit `tekton/tasks/2-smoke-test.yaml` and delete the `qm1-smoke` queue manager. You can simply just add `oc delete QueueManager qm1-smoke` at the very end of the health-check step (right before `workingDir`)

## Steps

### 1. Copy repo to your repository

Either manually download the folder and copy it to your bitbucket/github or git clone the repo and add another remote branch for your repository.

### 2. Explore and Edit files in `tekton` folder

- Notice that `tekton/tasks/4-scan.yaml` and `tekton/tasks/5-push.yaml` are empty. You can edit them based on the tools you use. For example, you can use sonarqube or prisma for image scanning and nexus registry for pushing the image. Currently the image is being pushed on OCP registry.

- Edit necessay parameters in your `tekton/pipeline/pipeline.yaml`
  Make sure when you provide the image name and tag in `pipeline.yaml` you use the same name/tag in `release.yaml` for our smoke test

### 3. Apply them on OpenShift

```
oc apply -f tekton/pipeline/
oc apply -f tekton/tasks/
```

Optionally: you can also add webhook for your repo

#### Next Steps

- Dynamic MQSC: https://github.com/ritu-patel/mq-dynamic-mqsc
- ArgoCD Setup: https://github.com/ritu-patel/mq-gitops
