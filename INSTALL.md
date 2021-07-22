# Installation Visual Flow to local machine

## Prerequisites

To install Visual Flow you should have the following software already installed:

- Docker Desktop on Windows (to install you can use the following link https://docs.docker.com/docker-for-windows/install/)
- kubectl (to install you can use the following link https://kubernetes.io/docs/tasks/tools/)
- Helm CLI (to install you can use the following link https://helm.sh/docs/intro/install/)

## Create an Kubernetes cluster

Visual Flow should be installed on an Kubernetes cluster, if you do not have it, then you can install it as follows:

1. Enable Kubernetes in Docker Desktop using [following instruction](https://docs.docker.com/desktop/kubernetes/#enable-kubernetes).

## Install Ingress controller

1. Install the ingress-nginx with the following command:

    `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.41.2/deploy/static/provider/cloud/deploy.yaml`

## Install Visual Flow

1. Clone or download the `local` branch from [the Visual-Flow-Deploy repository](https://github.com/ibagomel/Visual-Flow-deploy/tree/local) on your local computer.

2. Go to the directory "[visual-flow](./charts/visual-flow)" of the downloaded "Visual-Flow-Deploy" repository in the PowerShell with the following command:

    `cd .\charts\visual-flow\`

3. Create a GitHub OAuth app:

    1. Go to GitHub user's OAuth apps (`https://github.com/settings/developers`).
    2. Click the Register a new application or the New OAuth App button.
    3. Fill the required fields:
        - Application name: `Visual-Flow`
        - Homepage URL: `https://localhost/vf/ui/`
        - Authorization callback URL: `https://localhost/vf/ui/callback`
    4. Click the Register application button.
    5. Set the Client ID value from created GitHub OAuth app to GITHUB_APP_ID in [values.yaml](./charts/visual-flow/values.yaml).
    6. Click on the button `Generate a new client secret` and set GITHUB_APP_SECRET in [values.yaml](./charts/visual-flow/values.yaml) to the generated value (Please note that you will not be able to see the full secret value later).

4. (Optional) Configure Slack notifications in [values.yaml](./charts/visual-flow/values.yaml).

    How to allow VF to send Slack notifications from a pipeline can be found [here](https://github.com/ibagomel/Visual-Flow-deploy/blob/main/SLACK_NOTIFICATION.md).

5. Set superusers in [values.yaml](./charts/visual-flow/values.yaml).

    New Visual Flow users will have no access in the app. The superusers(admins) need to be configured to manage user access. Specify the superusers' GitHub nicknames in [values.yaml](./charts/visual-flow/values.yaml) in the yaml list format:

    ```yaml
    superusers:
      - your-github-nickname
      - another-name
    ```

6. Install the app using the updated values.yaml file with the following command:

    `helm install vf-app . -f values.yaml`

7. Check that the app is successfully installed and all pods are running with the following command:

    `kubectl get pods --all-namespaces`

## Use Visual Flow

1. All Visual Flow users (including superusers) need active Github account in order to be authenticated in application. Setup Github profile as per following steps:

    1. Navigate to the account settings
    2. Go to "Emails" tab: set email as public by unchecking "Keep my email addresses private" checkbox
    3. Go to "Profile" tab: fill in "Name" and "Public email" fields

2. Open the app's web page using the following link:

    https://localhost/vf/ui/

## Delete Visual Flow

1. If the app is no longer required, you can delete it using the following command:

    `helm uninstall vf-app`

2. Check that everything was successfully deleted with the command:

    `kubectl get pods --all-namespaces`

## Delete Kubernetes

1. If the Kubernetes is no longer required, you can delete it using the following link:

    https://docs.docker.com/desktop/kubernetes/#disable-kubernetes
