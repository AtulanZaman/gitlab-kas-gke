## KAS Agent configuration
This section describes how to deploy and manage KAS agent integration resources for workloads, deployed through the workload factory. It is recommended to read [this](https://about.gitlab.com/blog/2021/09/10/setting-up-the-k-agent/) document get an understanding of the architecture and resources involved.

### Pre-requisites
- Gitlab project which will hold the agentk configuration and deployment manifest exists.
- KAS agent server in Gitlab must running, if using a self-managed server. The agent server address is specified in the `kas_address` variable
- A GKE cluster is running where KAS agent client can be running in a pod.
- Namespace where application will be onboarded is provided as an example.
- Permissions for account to create a GAR in the project hosting the GKE cluster, for the KAS agent container registry
- Manifest files for projects should be in directory structure for example as such:

&nbsp;&nbsp;<gitlab server>/<namespace>/<project>/

&nbsp;&nbsp;&nbsp;&nbsp;-- /manifests/  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-- project-deployment.yaml 

The directory structure is used by the template in `templates/agent-config.yaml.tpl` to detect changes in manifest files.

### Resources created
**Gitlab Resources**
- Gitlab agent in the the Gitlab project
- Gitlab agent token for workload
- Config file based on template in `templates/agent-config.yaml.tpl`

**Agentk K8s Resources**
- Deploy KAS agent client through Helm chart
- Namespace for agentk image for each project
- GAR in GKE project where agentk image will be pushed

### Variables to configure (Example)
kas_address = "wss://kas.closedimage.dev/"
agentk_image_name = "agentk"
agentk_image_tag = "v15.2.0"

### How to upload agentk image (Example)
`docker pull registry.gitlab.com/gitlab-org/cluster-integration/gitlab-agent/agentk:v15.2.0`

`gcloud auth configure-docker northamerica-northeast1-docker.pkg.dev01`

`docker tag registry.gitlab.com/gitlab-org/cluster-integration/gitlab-agent/agentk:v15.2.0  northamerica-northeast1-docker.pkg.dev/{GKE Project URL}/{GAR name}/agentk:v15.2.0`

`docker push  northamerica-northeast1-docker.pkg.dev/{GKE Project URL}/{GAR name}/agentk:v15.2.0`

### Enhancements
- Pipeline driven lifecycle management for agentk image in GAR
- Pipeline for upgrading KAS agent server and client instances

### References and public docs
- [Using GitOps with a Kubernetes cluster](https://docs.gitlab.com/ee/user/clusters/agent/gitops.html)
- [How to deploy the GitLab Agent for Kubernetes with limited permissions](https://about.gitlab.com/blog/2021/09/10/setting-up-the-k-agent/)<a name="helpful">
- [Troubleshooting](https://repository.prace-ri.eu/git/help/user/clusters/agent/troubleshooting.md)
- [Installing the agent for Kubernetes](https://docs.gitlab.com/ee/user/clusters/agent/install)


PROJECT_ID=$(gcloud config get-value core/project) \
USER_ID=$(gcloud config get-value core/account) \
gcloud projects add-iam-policy-binding ${PROJECT_ID} --member=user:${USER_ID} --role=roles/container.admin \
kubectl create clusterrolebinding cluster-admin-binding --clusterrole cluster-admin --user ${USER_ID} ]