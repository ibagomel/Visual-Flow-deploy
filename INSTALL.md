# Installation Visual Flow to Amazon Elastic Kubernetes Service (EKS)

## Prerequisites

To install Visual Flow you should have the following software already installed:

- AWS CLI ([install](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html))
- kubectl [install](https://kubernetes.io/docs/tasks/tools/))
- eksctl ([install](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html))
- Helm CLI ([install](https://helm.sh/docs/intro/install/))
- Git ([install](https://git-scm.com/downloads))

**IMPORTANT**: all the actions are recommended to be performed from the admin/root AWS account.

If you have just installed the AWS CLI, then you need to log in using following command:

`aws configure`

## Create an EKS cluster

Visual Flow should be installed on an EKS cluster, if you do not have it, then create it using following guide:

<https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html>

## Connect to existing EKS cluster from local machine

If you have an EKS cluster, you can connect to it using the following command:

`aws eks --region <REGION> update-kubeconfig --name <CLUSTER_NAME>`

## Check access to EKS cluster

Run the following command to check access to the EKS cluster from the local machine:

`kubectl get nodes`

If you get the message "`error: You must be logged in to the server (Unauthorized)`", you can try to fix it using the following guide:

<https://aws.amazon.com/premiumsupport/knowledge-center/eks-api-server-unauthorized-error/>

If you have access to the EKS cluster on a different computer, you can try to copy the credentials from that computer and log into the AWS CLI on your local computer using the copied credentials. After that, reconnect to the ECS cluster ([previous chapter](#connect-to-existing-eks-cluster-from-local-machine)).

## Install an AWS Load Balancer (ALB) to EKS

AWS Load Balancer allows you to access applications on EKS from the Internet by hostname. If you don't have it installed, then install it using following guide:

<https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html>

**IMPORTANT:** The application has been tested with load balancer helm chart version 1.1.6. We strongly recommend to install load balancer helm chart version 1.1.6 to avoid unexpected issues. You can install the load balancer helm chart of the required version by executing the following command in step 5d in the guide on the link above:

```bash
helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller --set clusterName=<CLUSTER_NAME> --set region=<REGION> --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller -n kube-system --version 1.1.6
```

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

7. Install the app using the updated [values.yaml](./charts/visual-flow/values.yaml) file with the following command:

    `helm install vf-app . -f values.yaml`

8. Check that the app is successfully installed and all pods are running with the following command:

    `kubectl get pods --all-namespaces`

9. Get the generated app's hostname with the following command:

    `kubectl get svc vf-app-frontend -o yaml | grep hostname | cut -c 17-`

    Replace the string `<HOSTNAME_FROM_SERVICE>` with the generated hostname in the next steps.

10. Create a GitHub OAuth app:

    1. Go to GitHub user's OAuth apps (`https://github.com/settings/developers`) or organization's OAuth apps (`https://github.com/organizations/<ORG_NAME>/settings/applications`).
    2. Click the **Register a new application** or the **New OAuth App** button.
    3. Fill the required fields:
        - Set **Homepage URL** to `https://<HOSTNAME_FROM_SERVICE>/vf/ui/`
        - Set **Authorization callback URL** to `https://<HOSTNAME_FROM_SERVICE>/vf/ui/callback`
    4. Click the **Register application** button.
    5. Replace "DUMMY_ID" with the Client ID value in [values.yaml](./charts/visual-flow/values.yaml).
    6. Click **Generate a new client secret** and replace in [values.yaml](./charts/visual-flow/values.yaml) "DUMMY_SECRET" with the generated Client secret value (Please note that you will not be able to see the full secret value later).

11. Update STRATEGY_CALLBACK_URL value in [values.yaml](./charts/visual-flow/values.yaml) to `https://<HOSTNAME_FROM_SERVICE>/vf/ui/callback`

12. Upgrade the app in EKS cluster using updated values.yaml:

    `helm upgrade vf-app . -f values.yaml`

13. Wait until the update is installed and all pods are running:

    `kubectl get pods --all-namespaces`

## Use Visual Flow

1. All Visual Flow users (including superusers) need active Github account in order to be authenticated in application. Setup Github profile as per following steps:

    1. Navigate to the [account settings](https://github.com/settings/profile)
    2. Go to **Emails** tab: set email as public by unchecking **Keep my email addresses private** checkbox
    3. Go to **Profile** tab: fill in **Name** and **Public email** fields

2. Open the app's web page using the following link:

    `https://<HOSTNAME_FROM_SERVICE>/vf/ui/`

3. See the guide on how to work with the Visual Flow at the following link: [Visual_Flow_User_Guide.pdf](https://github.com/ibagomel/Visual-Flow/blob/main/Visual_Flow_User_Guide.pdf)

4. For each project Visual Flow generates a new namespace. For each namespace, you should create a Fargate profile to allow running jobs and pipelines in the corresponding project. Create a Fargate profile with the following command:

    `eksctl create fargateprofile --cluster <CLUSTER_NAME> --region <REGION> --name vf-app --namespace <NAMESPACE>`

## Delete Visual Flow

1. If the app is no longer required, you can delete it using the following command:

    `helm uninstall vf-app`

2. Check that everything was successfully deleted with the command:

    `kubectl get pods --all-namespaces`

## Delete EKS

1. If the EKS is no longer required, you can delete it using the following guide:

    <https://docs.aws.amazon.com/eks/latest/userguide/delete-cluster.html>
