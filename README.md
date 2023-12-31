## This is complete demo to show the capabilities of RHODS for model finetuning using codeflare(ray,mcad,instascale) and inferencing using TGIS-Caikit


## Pre-requsites 

## Installing the Node Feature Discovery (NFD) Operator

### Step 1: Installation

Open the OpenShift Container Platform web console.
Install the NFD Operator using the Red Hat OperatorHub catalog.

### Step 2: Verification

```bash
$ oc get pods -n openshift-nfd
```

You should see the NFD Operator running.

### Step 3: Creating an Instance of Node Feature Discovery

Go to Operators > Installed Operators.

Click NodeFeatureDiscovery under the Provided APIs field.

Click Create NodeFeatureDiscovery.

Note: The values pre-populated by the OperatorHub are valid for the GPU Operator.

### Step 4: Verification of NFD Operator Functionality
Verify the NFD Operator's functionality using the OpenShift Container Platform web console or the CLI.

Note: NVIDIA uses the PCI ID 10de.

```bash
oc describe node | egrep 'Roles|pci' | grep -v master

Roles:              worker
                    feature.node.kubernetes.io/pci-10de.present=true
                    feature.node.kubernetes.io/pci-1d0f.present=true
Roles:              worker
                    feature.node.kubernetes.io/pci-1013.present=true
                    feature.node.kubernetes.io/pci-8086.present=true
Roles:              worker
                    feature.node.kubernetes.io/pci-1013.present=true
                    feature.node.kubernetes.io/pci-8086.present=true
Roles:              worker
                    feature.node.kubernetes.io/pci-1013.present=true
                    feature.node.kubernetes.io/pci-8086.present=true
```

## Installing the NVIDIA GPU Operator

### Step 1: Installation

Navigate to Operators > OperatorHub and select All Projects.

Search for and install the NVIDIA GPU Operator.

### Step 2: Creating the Cluster Policy

Go to Operators > Installed Operators, and click NVIDIA GPU Operator.

Select the ClusterPolicy tab, then click Create ClusterPolicy.

Note: It might take 10-20 minutes to finish the installation. Verify the status as State: ready when the installation succeeds.

## Install RHODS via the OperatorHub UI

#### From the OpenShift UI, navigate to Operators --> OperatorHub and search for: Red Hat OpenShift Data Science.

#### Install CodeFlare Operator via the OperatorHub UI

#### Navigate to Operators --> OperatorHub and search for: CodeFlare operator.

### Additional Configuration

#### Execute the following commands to apply necessary roles and bindings and instantiate codeflare kdef:

```bash 
oc apply -f - <<EOF
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: rhods-operator-scc
rules:
  - verbs:
      - get
      - watch
      - list
      - create
      - update
      - patch
      - delete
    apiGroups:
      - security.openshift.io
    resources:
      - securitycontextconstraints
EOF

oc apply -f - <<EOF
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: rhods-operator-scc
subjects:
- kind: ServiceAccount
  name: rhods-operator
  namespace: redhat-ods-operator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: rhods-operator-scc
EOF

oc apply -f - <<EOF
apiVersion: kfdef.apps.kubeflow.org/v1
kind: KfDef
metadata:
  name: codeflare-stack
  namespace: redhat-ods-applications
spec:
  applications:
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: codeflare-stack
    name: codeflare-stack
  - kustomizeConfig:
      repoRef:
        name: manifests
        path: ray/operator
    name: ray-operator
  repos:
  - name: manifests
    uri: https://github.com/red-hat-data-services/distributed-workloads/tarball/main
EOF

```

#### Accessing the Dashboard/Notebook UI

Execute the following command:

```bash
oc get route -n redhat-ods-applications | grep dash | awk '{print $2}'

```
# FineTuning 
## Start with launching the CodeFlare notebook from the Red Hat OpenShift AI’s dashboard and cloning  this repository, which includes the notebook and necessary files for the demo. Try the notebook llamafinetune_demo.ipynb to demo the fine-tuning job submission.

#### After the job run is completed you should find your models in your HF repo like this for example 

```bash
https://huggingface.co/avijra/Llama-2-7b-chat-hf-fine-tuned
```

### P.S. this   job am only running for "max_steps=10" only for demo purposes, in real world scenarios consider running for maximum steps so that your loss < 0.5 

# Inferencing
## Try the notebook llamainfer_demo.ipynb to demo the model Inferencing.
#### please note for inferencing you would require notebook tunning on rhods with minimum 30GB mem and python 3.9 .Also note codeflare notebook image right now only has python 3.8 so it will not work . choose python 3.9 notebook for inferencing tasks 

