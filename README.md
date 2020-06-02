# Configuring your tekton environment

1. Install tekton pipeline 0.11.0+. One easy way is to install Kabanero, which will automatically install the operator to configure it correctly on OpenShift:
```
curl -s -O -L https://github.com/kabanero-io/kabanero-operator/releases/download/0.9.0/install.sh && bash install.sh
```

2. Setup a default storage class in your cluster. For example,
```
oc patch sc rook-ceph-cephfs-internal -p '{"metadata": {"annotations": {"storageclass.kubernetes.io/is-default-class": "true"}}}'
```

3. Install the RuntimeComponent Operator:
```
oc new-project runtime-component-operator
OPERATOR_NAMESPACE=runtime-component-operator
WATCH_NAMESPACE='""'

oc apply -f https://raw.githubusercontent.com/application-stacks/runtime-component-operator/master/deploy/releases/0.6.0/runtime-component-crd.yaml
curl -L https://raw.githubusercontent.com/application-stacks/runtime-component-operator/master/deploy/releases/0.6.0/runtime-component-cluster-rbac.yaml | sed -e "s/RUNTIME_COMPONENT_OPERATOR_NAMESPACE/${OPERATOR_NAMESPACE}/" | oc apply -f -
curl -L https://raw.githubusercontent.com/application-stacks/runtime-component-operator/master/deploy/releases/0.6.0/runtime-component-operator.yaml | sed -e "s/RUNTIME_COMPONENT_WATCH_NAMESPACE/${WATCH_NAMESPACE}/" | oc apply -n ${OPERATOR_NAMESPACE} -f -
```

4. Add your credentials to the `tekton/Secret.docker-credentials.yaml` file, adjusting these lines:
```
stringData:
#  username: FILL IN YOUR INFO!
#  password: FILL IN YOUR INFO!
```

5. Create the resources defining the pipeline:
```
oc apply -k tekton/
```

# Running the example pipeline

6. Modify the example PipelineRun (at `tekton/example/PipelineRun.prototype-run.yaml`) to point to your github repository and docker registry, adjusting these lines:
```
    - name: repo-url
      value: https://github.com/a-roberts/user-app-build-deploy.git
    - name: image
      value: docker.io/justinkulibm/devfile-build-test
```

7. Create the example PipelineRun (and the PVC that it will use):
```
sed -e "s/PROTOTYPE_PVC/$(oc create -f tekton/example/PersistentVolumeClaim.prototype-pvc.yaml -o=jsonpath='{.metadata.name}')/" tekton/example/PipelineRun.prototype-run.yaml | oc create -f -
```

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
