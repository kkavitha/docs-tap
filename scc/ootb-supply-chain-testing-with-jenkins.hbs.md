# Out of the Box Supply Chain with Testing on Jenkins

The OOTB template package now ships with a Tekton task which triggers a build for 
a specified jenkins job.

<!-- Links pending -->
The tekton test task in both the Out of the Box Supply Chain With Testing and
Out of the Box Supply Chain With Testing and Scanning can be configured
to trigger a jenkins job

## <a id="prerequisite"></a> Prerequisites

Follow instructions from Out of the Box Supply Chain With Testing or
Out of the Box Supply Chain With Testing and Scanning to successfully install the 
required package

### <a id="updates-to-developer-namespace"></a> Updates to the developer Namespace

#### <a id="create-secret"></a> Create a secret

A secret has to be created in the developer namespace with the following properties:

- `url` **required** URL of the jenkins instance that hosts the job
- `username` **required** Username of the user that has access to trigger a build on jenkins
- `password` **required** Password of the user that has access to trigger a build on jenkins
- `ca-cert` _optional_ The CA Cert to talk to the jenkins instance

For example:

```yaml
apiVersion: v1
data:
  password: <password>
  url: <jenkins-url>
  username: <username>
  ca-cert: <ca-cert>
kind: Secret
metadata:
  name: my-secret
type: Opaque
```

#### <a id="tekton-pipeline"></a> Create a Tekton pipeline

The developer has to create a Tekton pipeline object with the
following properties

Parameters:

- `source-url`, **required** an HTTP address where a `.tar.gz` file containing all the
  source code to be tested can be found
- `source-revision`, **required** the revision of the commit or image reference (in case of
  `workload.spec.source.image` being set instead of `workload.spec.source.git`)
- `secret-name`, **required** the secret that contains the URL, username, password and certificate
  to the jenkins instance that houses the job that needs to be run
- `job-name`, **required** name of the jenkins job that needs to run
- `job-params`, _optional_ string value that passes in parameters needed for
  the jenkins job

Tasks:

- `jenkins-task`, **required** this `ClusterTask` should be one of the tasks that the
  pipeline should run inorder to successfully trigger the jenkins job

Results:

- `jenkins-job-url`, **required** a string result that outputs the url of the jenkins build
  that the tekton task triggered. This output is populated by the `jenkins-task` `ClusterTask`

For example:

```yaml
---
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: developer-defined-jenkins-tekton-pipeline
  labels:
    #! This label should be provided to workload
    apps.tanzu.vmware.com/pipeline: jenkins-pipeline
spec:
  results:
  #! Required: To show the job url on the UI
  - name: jenkins-job-url
    value: $(tasks.jenkins-task.results.jenkins-job-url)
  params:
  #! Required
  - name: source-url
  #! Required
  - name: source-revision
  #! Required
  - name: secret-name
  #! Required
  - name: job-name
  #! Optional
  - name: job-params
  tasks:
  #! Required: Include the inbuilt task that triggers the 
  #! given job in Jenkins
  - name: jenkins-task
    taskRef:
      name: jenkins-task
      kind: ClusterTask
    params:
      - name: source-url
        value: $(params.source-url)
      - name: source-revision
        value: $(params.source-revision)
      - name: secret-name
        value: $(params.secret-name)
      - name: job-name
        value: $(params.job-name)
      - name: job-params
        value: $(params.job-params)

```

## <a id="developer-workload"></a> Developer Workload

You should submit your Workload to the same namespace as the Tekton pipeline

To enable the supply chain to run jenkins task, the workload should include the following
parameters:

```yaml
params:
  #! Required: picks the pipeline
  - name: testing-pipeline-matching-labels
    value:
      #! This label should match the label on the pipeline created
      apps.tanzu.vmware.com/pipeline: jenkins-pipeline 
  #! Required: Passes parameters to pipeline
  - name: testing-pipeline-params
    value:
      #! Required: Name of the jenkins job
      job-name: job-with-parameters
      #! Optional: Parameters to pass to the jenkins job
      job-params:
      - name: GREETEE
        value: something
      #! Required: The secret create earlier to access jenkins
      secret-name: my-secret
```

The workload can also be created using the `apps` CLI plugin:

```console
tanzu apps workload create tanzu-java-web-app \
  --git-branch main \
  --git-repo https://github.com/sample-accelerators/tanzu-java-web-app \
  --label apps.tanzu.vmware.com/has-tests=true \
  --label app.kubernetes.io/part-of=tanzu-java-web-app \
  --param-yaml testing-pipeline-matching-labels='{"apps.tanzu.vmware.com/pipeline":"jenkins-pipeline"}' \
  --param-yaml testing-pipeline-params='{"secret_name":"jenkins-secret", "job_name": "jenkins-job", "job_params": [{"name":"param1","value":"value1"}]}'
  --type web
```

```console
1 + |---
      2 + |apiVersion: carto.run/v1alpha1
      3 + |kind: Workload
      4 + |metadata:
      5 + |  labels:
      6 + |    app.kubernetes.io/part-of: tanzu-java-web-app
      7 + |    apps.tanzu.vmware.com/has-tests: "true"
      8 + |  name: tanzu-java-web-app
      9 + |  namespace: default
     10 + |spec:
     11 + |  params:
     12 + |  - name: testing-pipeline-matching-labels
     13 + |    value:
     14 + |      apps.tanzu.vmware.com/pipeline: jenkins-pipeline
     15 + |  - name: testing-pipeline-params
     16 + |    value:
     17 + |      job_name: jenkins-job
     18 + |      job_params:
     19 + |      - name: param1
     20 + |        value: value1
     21 + |      secret_name: jenkins-secret
     22 + |  source:
     23 + |    git:
     24 + |      ref:
     25 + |        branch: main
     26 + |      url: https://github.com/sample-accelerators/tanzu-java-web-app
```
