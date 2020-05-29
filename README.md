# Sample applciation for build/deploy pipeline

This is just a simple express.js application along with a devfile.yaml that provides links to :
- build information - Dockerfile to convert this source into a container image
- deployment information - k8s manifest to deploy the containerized application

## Building with tekton

Add your Docker credentials to `tekton/my-docker-creds.secret.yaml` so that you can push/pull images to your own repository, then setup the pipeline/tasks with:
```bash
kubectl apply -k tekton/
```

If you've setup dynamic PV provisioning in the cluster, it should then be as easy as modifying and applying `tekton/example.pipeline-run.yaml`.
