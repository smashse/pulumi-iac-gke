This is a model for creating in Python an GKE environment with the GCP provider on Pulumi using GitHub Actions.

- Access "https://github.com/smashse/pulumi-iac-gke"
- Click in "Use this template"
- Create a new repository from template "pulumi-iac-gke"(example "pulumi-iac-gke") and chose as "Private"

# Install GCP (Optional)

```bash
cd /tmp
sudo touch /etc/apt/sources.list.d/google-cloud-sdk.list
sudo chmod 666 /etc/apt/sources.list.d/google-cloud-sdk.list
sudo curl -fsSL 'https://packages.cloud.google.com/apt/doc/apt-key.gpg' | sudo apt-key add -
sudo echo "deb [arch=amd64] http://packages.cloud.google.com/apt cloud-sdk main" > "/etc/apt/sources.list.d/google-cloud-sdk.list"
sudo chmod 644 /etc/apt/sources.list.d/google-cloud-sdk.list
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 54A647F9048D5688D7DA2ABE6A030B21BA07F4FB
sudo apt update --fix-missing
sudo apt -y install google-cloud-sdk
gcloud init
```

## Configure environment

### Authorize gcloud to access the Cloud Platform with Google user credentials

```bash
gcloud auth login
```

### List organizations accessible by the active account

```bash
gcloud organizations list
```

### List billing account ID

```bash
gcloud beta billing accounts list --filter=open=true
```

### Create the PULUMI Project

**Note:** Project IDs are immutable and can be set only during project creation. They must start with a lowercase letter and can have lowercase ASCII letters, digits or hyphens. Project IDs must be between 6 and 30 characters. To avoid conflicts, when creating a project the ID is generated randomly, if you want to use a fixed ID after the project name, do as below.

```bash
export PROJECT_ID=`date +%M%S%N`
gcloud projects create gke-project-$PROJECT_ID --name=gke-project --set-as-default
gcloud config set project gke-project-$PROJECT_ID
gcloud beta billing projects link gke-project-$PROJECT_ID --billing-account `gcloud beta billing accounts list --filter=open=true --uri | cut -f 6 -d "/"`
```

### Create the PULUMI service account

```bash
gcloud iam service-accounts create gkeadmin --display-name "GKE Admin"
gcloud iam service-accounts keys create ~/.config/gcloud/gkeadmin-account.json --iam-account gkeadmin@gke-project-$PROJECT_ID.iam.gserviceaccount.com
```

### Enable API Services

```bash
gcloud services enable cloudbilling.googleapis.com
gcloud services enable cloudresourcemanager.googleapis.com
gcloud services enable compute.googleapis.com
gcloud services enable container.googleapis.com
gcloud services enable gkeconnect.googleapis.com
gcloud services enable iam.googleapis.com
gcloud services enable serviceusage.googleapis.com
```

### Grant the service account permission

```bash
gcloud projects add-iam-policy-binding gke-project-$PROJECT_ID --member serviceAccount:gkeadmin@gke-project-$PROJECT_ID.iam.gserviceaccount.com --role roles/compute.admin
gcloud projects add-iam-policy-binding gke-project-$PROJECT_ID --member serviceAccount:gkeadmin@gke-project-$PROJECT_ID.iam.gserviceaccount.com --role roles/container.clusterAdmin
gcloud projects add-iam-policy-binding gke-project-$PROJECT_ID --member serviceAccount:gkeadmin@gke-project-$PROJECT_ID.iam.gserviceaccount.com --role roles/iam.serviceAccountAdmin
gcloud projects add-iam-policy-binding gke-project-$PROJECT_ID --member serviceAccount:gkeadmin@gke-project-$PROJECT_ID.iam.gserviceaccount.com --role roles/iam.serviceAccountKeyAdmin
gcloud projects add-iam-policy-binding gke-project-$PROJECT_ID --member serviceAccount:gkeadmin@gke-project-$PROJECT_ID.iam.gserviceaccount.com --role roles/iam.serviceAccountUser
gcloud projects add-iam-policy-binding gke-project-$PROJECT_ID --member serviceAccount:gkeadmin@gke-project-$PROJECT_ID.iam.gserviceaccount.com --role roles/iap.httpsResourceAccessor
gcloud projects add-iam-policy-binding gke-project-$PROJECT_ID --member serviceAccount:gkeadmin@gke-project-$PROJECT_ID.iam.gserviceaccount.com --role roles/storage.admin
gcloud projects add-iam-policy-binding gke-project-$PROJECT_ID --member serviceAccount:gkeadmin@gke-project-$PROJECT_ID.iam.gserviceaccount.com --role roles/viewer
```

## Download the PULUMI template

```bash
mkdir -p $HOME/Pulumi
cd $HOME/Pulumi
git clone https://github.com/yourgithubuser/pulumi-iac-gke.git
cd pulumi-iac-gke
```

## Install Pulumi on Linux by running the installation script:

```bash
curl -fsSL https://get.pulumi.com | sh && bash
```

## Install Python VirtualEnv:

```bash
sudo apt -y install python3-virtualenv
```

## Create a "pulumi_gke_py" project:

```bash
cd $HOME/Pulumi/pulumi-iac-gke/pulumi_gke_py
```

**Note:** If you want to change the name given to Kubernetes cluster, execute the command below in the template folder.

```bash
sed -i "s/"template-"/"desiredname-"/g" *.py
```

## Install Python Requirements

```bash
python3 -m venv venv
source venv/bin/activate
python -m pip install --upgrade pip setuptools wheel
python -m pip install -r requirements.txt
```

## Perform an initial deployment, run the following commands:

```bash
pulumi login
pulumi stack init pulumi_gke_py
```

## Set GCP_PROJECT:

```bash
pulumi config set gcp:project gke-project-$PROJECT_ID
```

## Set GCP_REGION:

```bash
pulumi config set gcp:zone us-west1-a
```

## Review the "pulumi_gke_py" project

```bash
pulumi preview
```

## Enable Workflow

```bash
cd $HOME/Pulumi/pulumi-iac-gke/.github/workflows
```

```bash
mv pull_request.yml.template pull_request.yml
mv push.yml.template push.yml
```

## Environment Variables

There are a number of Environment Variables that can be set to interact with the action:

- By default, Pulumi will try to connect to the [Pulumi SaaS](https://app.pulumi.com/). For this to happen, the GitHub Action needs to be passed a "PULUMI_ACCESS_TOKEN".

## Google Cloud Platform (GCP)

For GCP, you'll need to create or use or use an existing service account key. Please see
[the Pulumi documentation page](https://pulumi.io/quickstart/gcp/setup.html) for pointers
to the relevant GCP documentation for doing this.

As soon as you have credentials in hand, you'll set the environment variable "GOOGLE_CREDENTIALS" to contain the
credentials JSON using GitHub Secrets, and then consume it in your action:

How to get JSON for credentials?

```bash
gcloud auth application-default login
cat $HOME/.config/gcloud/application_default_credentials.json
```

**Note:** Go to Settings> Secrets and add "PULUMI_ACCESS_TOKEN" and "GOOGLE_CREDENTIALS" as new repository secret.

## Commit the changes

```bash
cd $HOME/Pulumi/pulumi-iac-gke/
```

```bash
git add *
git add .github/workflows/*
git add .pulumi/*
git add pulumi_gke_py/*
git commit -m "pulumi-iac-gke"
git push
```

## Access GKE Kubernetes cluster

```bash
sudo snap install kubectl --classic
pulumi stack output kubeconfig > kubeconfig.yaml
KUBECONFIG=./kubeconfig.yaml kubectl get po --all-namespaces
```

## Destroy the "pulumi_gke_py" project

```bash
cd $HOME/Pulumi/pulumi-iac-gke/pulumi_gke_py
pulumi destroy
```

## Remove the "pulumi_gke_py" project from Stack

```bash
cd $HOME/Pulumi/pulumi-iac-gke/pulumi_gke_py
pulumi stack rm pulumi_gke_py
```

## Source:

<https://www.pulumi.com/docs/get-started/>

<https://www.pulumi.com/docs/reference/pkg/>

<https://www.pulumi.com/docs/intro/concepts/state/>

<https://www.pulumi.com/docs/guides/continuous-delivery/github-actions/>

<https://github.com/pulumi/actions>
