# Installation Visual Flow to Amazon Elastic Kubernetes Service (EKS)

## Prerequisites
To install Visual Flow you should have the following software already installed:
- AWS CLI (to install you can use the following link https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)
- kubectl (to install you can use the following link https://kubernetes.io/docs/tasks/tools/)
- eksctl (to install you can use the following link https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)
- Helm CLI (to install you can use the following link https://helm.sh/docs/intro/install/)

**IMPORTANT**: all the actions are recommended to be performed from the admin/root AWS account.

If you have just installed the AWS CLI, then you need to log in:

`aws configure`

And enter the Access key ID and the Secret access key.

## Create an EKS Cluster

Visual Flow should be installed on an EKS Cluster, if you do not have it, then you can install it as follows:


1. Create a cluster from a template using the following command:

    `eksctl create cluster --region <REGION> --name <CLUSTER_NAME> --fargate --full-ecr-access --alb-ingress-access`

    It can take about 20 minutes

2. Сheck with both commands that the cluster has been created:

    ```bash
    kubectl get nodes
    kubectl get pods --all-namespaces
    ```


## Install an AWS Load Balancer (ALB) to EKS

AWS Load Balancer allows you to access applications on EKS from the Internet by hostname.

1. Create an IAM OIDC identity provider for your cluster with the following command:
   
    `eksctl utils associate-iam-oidc-provider --cluster <CLUSTER_NAME> --approve`


2. Create AWSLoadBalancerControllerIAMPolicy if it does not exist in your account:
   
   `aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.1.3/docs/install/iam_policy.json`


3. Create an account for ALB with the following command:
   
    `eksctl create iamserviceaccount --cluster=<CLUSTER_NAME> --namespace=kube-system --name=aws-load-balancer-controller --attach-policy-arn=<ARN_OF_AWSLoadBalancerControllerIAMPolicy> --override-existing-serviceaccounts --approve`

4. For a new ALB installation, you should add an EKS Helm repository with the following command:
   
    `helm repo add eks https://aws.github.io/eks-charts`

5. Get the VPC_ID value with the following command:
   
    `VPC_ID=$(eksctl utils describe-stacks --region=<REGION> --cluster=<CLUSTER_NAME> | grep vpc- | cut -d '"' -f 2)`

6. Install aws-load-balancer-controller with the following command:
    ```bash
    helm install aws-load-balancer-controller eks/aws-load-balancer-controller --set clusterName=<CLUSTER_NAME> --set region=<REGION> --set vpcId=$VPC_ID --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller -n kube-system --version 1.1.6
    ```

7. Check that aws-load-balancer-controller has been successfully installed with the following command:
   
    `kubectl get pods --all-namespaces`

## Install Visual Flow

1. Clone or download the amazon branch from [the Visual-Flow-Deploy repository](https://github.com/ibagomel/Visual-Flow-deploy/tree/amazon) on your local computer.

2. Go to the directory "[visual-flow](https://github.com/ibagomel/Visual-Flow-deploy/blob/amazon/charts/visual-flow)" of the downloaded "Visual-Flow-Deploy" repository with the following command:

    `cd ./charts/visual-flow`

3. *(Optional)* Configure Slack notifications in values.yaml.

    In order to allow VF to send Slack notifications from a pipeline, VF needs to be added as a bot user to your Slack Workspace. The documentation on how to add a bot user in Slack and get the bot user token can be found on [Slack help page](https://slack.com/help/articles/215770388-Create-and-regenerate-API-tokens#bot-user-tokens). After generating token, update SLACK_API_TOKEN variable in values.yaml with the token generated.

4. Set superusers in values.yaml.

    New Visual Flow users will have no access in the app. The superusers(admins) need to be configured to manage user access. Specify the superusers' GitHub nicknames in values.yaml in the yaml list format:

    ```yaml
    superusers:
      - your-github-nickname
      - another-name
    ```

5. All Visual Flow users (including superusers) need active Github account in order to be authenticated in application. Setup Github profile as per following steps:

    - Navigate to the account settings
    - Go to "Emails" tab: set email as public by unchecking "Keep my email addresses private" checkbox
    - Go to "Profile" tab: fill in "Name" and "Public email" fields

6. Install the app using the updated values.yaml file with the following command:

    `helm install vf-app . -f values.yaml`

7. Check that the app is successfully installed and all pods are running with the following command:

    `kubectl get pods --all-namespaces`

8. Get the generated app's hostname with the following command:
  
   `kubectl get svc vf-app-frontend -o yaml | grep hostname | cut -c 17-`

9. Create a GitHub OAuth app:
   1. Go to GitHub user's OAuth apps (`https://github.com/settings/developers`) or organization's OAuth apps (`https://github.com/organizations/<ORG_NAME>/settings/applications`).
   2. Click the Register a new application or the New OAuth App button.
   3. Fill the required fields (Set Authorization callback URL to `https://<HOSTNAME_FROM_SERVICE>/vf/ui/callback`), click the Register application button.
   4. Set the Client ID value to GITHUB_APP_ID in values.yaml.
   5. Click Generate a new client secret and set GITHUB_APP_SECRET in values.yaml to the value generated (Please note that you will not be able to see the full secret value later).

10. Complete the app's configuration:

   1. Set STRATEGY_CALLBACK_URL in values.yaml to `https://<HOSTNAME_FROM_SERVICE>/vf/ui/callback`.
   2. Upgrade the app in EKS:

        `helm upgrade vf-app . -f values.yaml`

   3. Wait until the update is installed and all pods are running.

## Use Visual Flow

1. Open the app's web page using the following link:

   `https://<HOSTNAME_FROM_SERVICE>/vf/ui/`

2. For each project Visual Flow generates a new namespace. For each namespace, you should create a Fargate profile to allow running jobs and pipelines in the corresponding project. Create a Fargate profile with the following command:

    `eksctl create fargateprofile --cluster <CLUSTER_NAME> --region <REGION> --name vf-app --namespace <NAMESPACE>`

## Delete Visual Flow

1. If the app is no longer required, you can delete it using the following command:

    `helm uninstall vf-app`

2. Check that everything was successfully deleted with the command:

    `kubectl get pods --all-namespaces`

## Delete EKS

1. If the EKS is no longer required, you can delete it using the following command:

    `eksctl delete cluster --region <REGION> --name <CLUSTER_NAME>`

    It can take about 10 minutes.

2. You can check that everything was successfully deleted here: 

    `https://console.aws.amazon.com/cloudformation/home?region=<REGION>#/stacks`
