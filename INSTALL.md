# Installation Visual Flow to Amazon Elastic Kubernetes Service (EKS)

## Prerequisites

To install Visual Flow you should have the following software already installed:

- AWS CLI (to install you can use the following link <https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html>)
- kubectl (to install you can use the following link <https://kubernetes.io/docs/tasks/tools/>)
- eksctl (to install you can use the following link <https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html>)
- Helm CLI (to install you can use the following link <https://helm.sh/docs/intro/install/>)
- Git (to install you can use the following link <https://git-scm.com/downloads>)

**IMPORTANT**: all the actions are recommended to be performed from the admin/root AWS account.

If you have just installed the AWS CLI, then you need to log in:

`aws configure`

And enter the Access key ID and the Secret access key.

## Create an EKS Cluster

Visual Flow should be installed on an EKS Cluster, if you do not have it, then create it using following guide:

<https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html>

## Connect to existing EKS Cluster from local machine

If you have an EKS Cluster, you can connect to it using the following command:

`aws eks --region <REGION> update-kubeconfig --name <CLUSTER_NAME>`

## Install Visual Flow

1. Clone (or download) the amazon branch from [the Visual-Flow-Deploy repository](https://github.com/ibagomel/Visual-Flow-deploy/tree/amazon) on your local computer using following command:

    `git clone -b amazon https://github.com/ibagomel/Visual-Flow-deploy.git Visual-Flow-deploy`

2. Go to the directory "[visual-flow](https://github.com/ibagomel/Visual-Flow-deploy/blob/amazon/charts/visual-flow)" of the downloaded "Visual-Flow-Deploy" repository with the following command:

    `cd Visual-Flow-deploy/charts/visual-flow`

3. *(Optional)* Configure Slack notifications in [values.yaml](./charts/visual-flow/values.yaml) using following guide:

    <https://github.com/ibagomel/Visual-Flow-deploy/blob/main/SLACK_NOTIFICATION.md>

4. Set superusers in [values.yaml](./charts/visual-flow/values.yaml).

    New Visual Flow users will have no access in the app. The superusers(admins) need to be configured to manage user access. Specify the superusers real GitHub nicknames in [values.yaml](./charts/visual-flow/values.yaml) in the yaml list format:

    ```yaml
    superusers:
      - your-github-nickname
    ```

5. If you have installed kube-metrics then update values.yaml file according to the example below.

    1. Check that the kube-metrics installed using the following command:

        ```bash
        kubectl top pods
        ```

        Output if the kube-metrics isn't installed:

        `error: Metrics API not available`

        If the kube-metrics isn't installed then go to step 6.

    2. Edit [values.yaml](./charts/visual-flow/values.yaml) file according to the example below:

        ```yaml
        ...
        kube-metrics:
          install: false
        ```

6. If you have installed Argo workflows then update values.yaml file according to the example below.

    1. Check that the Argo workflows installed using the following command:

        ```bash
        kubectl get workflow
        ```

        Output if the Argo workflows isn't installed:

        `error: the server doesn't have a resource type "workflow"`

        If the Argo workflows isn't installed then go to step 7.

    2. Edit [values.yaml](./charts/visual-flow/values.yaml) file according to the example below:

        ```yaml
        ...
        argo:
          install: false
        vf-app:
          backend:
            configFile:
              argoServerUrl: <Argo-Server-URL>
        ```

7. Create a GitHub OAuth app:

    1. Go to GitHub user's OAuth apps (`https://github.com/settings/developers`) or organization's OAuth apps (`https://github.com/organizations/<ORG_NAME>/settings/applications`).
    2. Click the **Register a new application** or the **New OAuth App** button.
    3. Fill the required fields:
        - Set **Homepage URL** to `https://localhost/vf/ui/`
        - Set **Authorization callback URL** to `https://localhost/vf/ui/callback`
    4. Click the **Register application** button.
    5. Replace "DUMMY_ID" with the Client ID value in [values.yaml](./charts/visual-flow/values.yaml).
    6. Click **Generate a new client secret** and replace in [values.yaml](./charts/visual-flow/values.yaml) "DUMMY_SECRET" with the generated Client secret value  (Please note that you will not be able to see the full secret value later).

8. Install the app using the updated [values.yaml](./charts/visual-flow/values.yaml) file with the following command:

    `helm install vf-app . -f values.yaml`

9. Check that the app is successfully installed and all pods are running with the following command:

    `kubectl get pods --all-namespaces`

## Use Visual Flow

1. All Visual Flow users (including superusers) need active Github account in order to be authenticated in application. Setup Github profile as per following steps:

    1. Navigate to the [account settings](https://github.com/settings/profile)
    2. Go to **Emails** tab: set email as public by unchecking **Keep my email addresses private** checkbox
    3. Go to **Profile** tab: fill in **Name** and **Public email** fields

2. Start port forwarding to get access to app with the following command:

    ```bash
    kubectl port-forward svc/vf-app-frontend 443:443
    ```

3. Open the app's web page using the following link:

    `https://localhost/vf/ui/`

4. See the guide on how to work with the Visual Flow at the following link: <https://github.com/ibagomel/Visual-Flow/blob/main/Visual%20Flow%20User%20Guide%20V0.4.pdf>

5. For each project Visual Flow generates a new namespace. For each namespace, you should create a Fargate profile to allow running jobs and pipelines in the corresponding project. Create a Fargate profile with the following command:

    `eksctl create fargateprofile --cluster <CLUSTER_NAME> --region <REGION> --name vf-app --namespace <NAMESPACE>`

## Delete Visual Flow

1. If the app is no longer required, you can delete it using the following command:

    `helm uninstall vf-app`

## Delete EKS

1. If the EKS is no longer required, you can delete it using the following guide:

    <https://docs.aws.amazon.com/eks/latest/userguide/delete-cluster.html>
