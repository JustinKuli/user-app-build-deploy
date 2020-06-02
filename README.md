# Building with tekton

Add your Docker credentials to `tekton/my-docker-creds.secret.yaml` so that you can push/pull images to your own repository, then setup the pipeline and tasks with:
```bash
kubectl apply -k tekton/
```

To run the example pipeline, you can create a PersistentVolume and reference it in the example PipelineRun. An example claim is already provided. Your volume should be ReadWriteOnce and capable of handling 50 MB of storage requests

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example
spec:
  capacity:
    storage: 50 Mi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: slow
  nfs:
    path: /tmp
    server: 172.17.0.2
```

for example. Hint, use the OpenShift UI and the server address will be filled in for you.
        
Fill in your details for the docker registry and the location of your github repository. Then you can use `kubectl create -f example.pipeline-run.yaml` to run the pipeline.

The pipeline takes URLs for the github repository hosting the code with the devfile and the target URL for the created image. Then it:

1. Pulls the repository.
2. Reads the `devfile.yaml` and fetches the specified `Dockerfile` and deployment manifest (k8s resource).
3. Builds the image with buildah.
4. Pushes the new image to the specified location, tagged with the Git commit of the repository.
5. Fills the deployment manifest with the new image/tag.
6. Attempts to deploy the resource.

# Sample applciation for build/deploy pipeline

This is just a simple express.js application along with a devfile.yaml that provides links to :
- build information - Dockerfile to convert this source into a container image
- deployment information - k8s manifest to deploy the containerized application
