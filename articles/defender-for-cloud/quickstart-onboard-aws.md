---
title: Connect your AWS account to Microsoft Defender for Cloud
description: Defend your AWS resources with Microsoft Defender for Cloud
ms.topic: quickstart
ms.date: 03/27/2022
zone_pivot_groups: connect-aws-accounts
ms.custom: mode-other
---
#  Connect your AWS accounts to Microsoft Defender for Cloud

[!INCLUDE [Banner for top of topics](./includes/banner.md)]

With cloud workloads commonly spanning multiple cloud platforms, cloud security services must do the same.

Microsoft Defender for Cloud protects workloads in Azure, Amazon Web Services (AWS), and Google Cloud Platform (GCP).

To protect your AWS-based resources, you can connect an account with one of two mechanisms:

- **Classic cloud connectors experience** - As part of the initial multi-cloud offering, we introduced these cloud connectors as a way to connect your AWS and GCP projects. If you've already configured an AWS connector through the classic cloud connectors experience, we recommend deleting these connectors (as explained in [Remove classic connectors](#remove-classic-connectors)), and connecting the account again using the newer mechanism. If you don't do this before creating the new connector through the environment settings page, do so afterwards to avoid seeing duplicate recommendations.

- **Environment settings page (in preview)** (recommended) - This preview page provides a greatly improved, simpler, onboarding experience (including auto provisioning). This mechanism also extends Defender for Cloud's enhanced security features to your AWS resources:

    - **Defender for Cloud's CSPM features** extend to your AWS resources. This agentless plan assesses your AWS resources according to AWS-specific security recommendations and these are included in your secure score. The resources will also be assessed for compliance with built-in standards specific to AWS (AWS CIS, AWS PCI DSS, and AWS Foundational Security Best Practices). Defender for Cloud's [asset inventory page](asset-inventory.md) is a multi-cloud enabled feature helping you manage your AWS resources alongside your Azure resources.
    - **Microsoft Defender for Containers** brings threat detection and advanced defenses to your Amazon EKS clusters. This plan includes Kubernetes threat protection, behavioral analytics, Kubernetes best practices, admission control recommendations and more. You can view the full list of available features in [Defender for Containers feature availability](supported-machines-endpoint-solutions-clouds-containers.md).
    - **Microsoft Defender for servers** brings threat detection and advanced defenses to your Windows and Linux EC2 instances. This plan includes the integrated license for Microsoft Defender for Endpoint, security baselines and OS level assessments, vulnerability assessment scanning, adaptive application controls (AAC), file integrity monitoring (FIM), and more. You can view the full list of available features in the [feature availability table](supported-machines-endpoint-solutions-clouds-servers.md?tabs=tab/features-multi-cloud).

For a reference list of all the recommendations Defender for Cloud can provide for AWS resources, see [Security recommendations for AWS resources - a reference guide](recommendations-reference-aws.md).

This screenshot shows AWS accounts displayed in Defender for Cloud's [overview dashboard](overview-page.md).

:::image type="content" source="./media/quickstart-onboard-aws/aws-account-in-overview.png" alt-text="Four AWS projects listed on Defender for Cloud's overview dashboard" lightbox="./media/quickstart-onboard-aws/aws-account-in-overview.png":::


::: zone pivot="env-settings"

## Availability

|Aspect|Details|
|----|:----|
|Release state:|Preview.<br>[!INCLUDE [Legalese](../../includes/defender-for-cloud-preview-legal-text.md)]|
|Pricing:|The **CSPM plan** is free.<br>The **[Defender for Containers](defender-for-containers-introduction.md)** plan is free during the preview. After which, it will be billed for AWS at the same price as for Azure resources.<br>For every AWS machine connected to Azure with [Azure Arc-enabled servers](../azure-arc/servers/overview.md), the **Defender for servers** plan is billed at the same price as the [Microsoft Defender for servers](defender-for-servers-introduction.md) plan for Azure machines. If an AWS EC2 doesn't have the Azure Arc agent deployed, you won't be charged for that machine.|
|Required roles and permissions:|**Contributor** permission for the relevant Azure subscription.|
|Clouds:|:::image type="icon" source="./media/icons/yes-icon.png"::: Commercial clouds<br>:::image type="icon" source="./media/icons/no-icon.png"::: National (Azure Government, Azure China 21Vianet)|


## Prerequisites

- Access to an AWS account.

- **To enable the Defender for Containers plan**, you'll need:
    - At least one Amazon EKS cluster with permission to access to the EKS K8s API server. If you need to create a new EKS cluster, follow the instructions in [Getting started with Amazon EKS – eksctl](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html).
    - The resource capacity to create a new SQS queue, Kinesis Fire Hose delivery stream, and S3 bucket in the cluster's region.

- **To enable the Defender for servers plan**, you'll need:
    
    - Microsoft Defender for servers enabled on your subscription. Learn how to enable plans in the [Enable enhanced security features](enable-enhanced-security.md) article.
    
    - An active AWS account, with EC2 instances.
    
    - Azure Arc for servers installed on your EC2 instances. 
        - (Recommended) Use the auto provisioning process to install Azure Arc on all of your existing, and future EC2 instances managed by AWS Systems Manager (SSM) and using the SSM agent. Some Amazon Machine Images (AMIs) already have the SSM agent pre-installed. If that is the case, their AMI's are listed in [AMIs with SSM Agent preinstalled](https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent-technical-details.html#ami-preinstalled-agent). If your EC2 instances don't have the SSM Agent, you will need to install it using either of the following relevant instructions from Amazon:
            - [Install SSM Agent for a hybrid environment (Windows)](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-install-managed-win.html)
            - [Install SSM Agent for a hybrid environment (Linux)](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-install-managed-linux.html)
        > [!NOTE]
        > To enable the Azure Arc auto-provisioning, you'll need an **Owner** permission on the relevant Azure subscription.
        
        - To manually install Azure Arc on your existing and future EC2 instances, follow the instructions in the [EC2 instances should be connected to Azure Arc](https://portal.azure.com/#blade/Microsoft_Azure_Security/RecommendationsBlade/assessmentKey/231dee23-84db-44d2-bd9d-c32fbcfb42a3) recommendation.
        
    - Additional extensions should be enabled on the Arc-connected machines. These extensions are currently configured in the subscription level. It means that all the multicloud accounts and projects (from both AWS and GCP) under the same subscription will inherit the subscription settings with regards to these components.
        - Microsoft Defender for Endpoint
        - VA solution (TVM/ Qualys)
        - Log Analytics (LA) agent on Arc machines. Ensure the selected workspace has security solution installed.

> [!Note]
> Without the Arc agent, you will be unable to take advantage of Defender for server's value. The Arc agent can also be installed manually, and not by the auto-provisioning process.

## Connect your AWS account

Follow the steps below to create your AWS cloud connector. 

### Remove 'classic' connectors

If you have any existing connectors created with the classic cloud connectors experience, remove them first:

1. Sign in to the [Azure portal](https://portal.azure.com). 

1. Navigate to **Defender for Cloud** > **Environment settings**.

1. Select the option to switch back to the classic connectors experience.

    :::image type="content" source="media/quickstart-onboard-gcp/classic-connectors-experience.png" alt-text="Switching back to the classic cloud connectors experience in Defender for Cloud.":::

1. For each connector, select the three dot button **…** at the end of the row, and select **Delete**.

1. On AWS, delete the role ARN, or the credentials created for the integration.

### Create a new connector

**To create a new connector**:

1. Sign in to the [Azure portal](https://portal.azure.com). 

1. Navigate to **Defender for Cloud** > **Environment settings**.

1. Select **Add environment** > **Amazon Web Services**.

    :::image type="content" source="media/quickstart-onboard-aws/add-aws-account-environment-settings.png" alt-text="Connecting an AWS account to an Azure subscription.":::

1. Enter the details of the AWS account, including the location where you'll store the connector resource.

    :::image type="content" source="media/quickstart-onboard-aws/add-aws-account-details.png" alt-text="Step 1 of the add AWS account wizard: Enter the account details.":::

1. Select **Next: Select plans**.

    > [!NOTE]
    > Each plan has its own requirements for permissions, and might incur charges.

    :::image type="content" source="media/quickstart-onboard-aws/add-aws-account-plans-selection.png" alt-text="The select plans tab is where you choose which Defender for Cloud capabilities to enable for this AWS account.":::

    > [!IMPORTANT]
    > To present the current status of your recommendations, the CSPM plan queries the AWS resource APIs several times a day. These read-only API calls incur no charges, but they *are* registered in CloudTrail if you've enabled a trail for read events. As explained in [the AWS documentation](https://aws.amazon.com/cloudtrail/pricing/), there are no additional charges for keeping one trail. If you're exporting the data out of AWS (for example, to an external SIEM), this increased volume of calls might also increase ingestion costs. In such cases, We recommend filtering out the read-only calls from the Defender for Cloud user or role ARN: `arn:aws:iam::[accountId]:role/CspmMonitorAws` (this is the default role name, confirm the role name configured on your account).

1. By default the **Servers** plan is set to **On**. This is necessary to extend Defender for server's coverage to your AWS EC2.
    
    - (Optional) Select **Configure**, to edit the configuration as required. 

1. By default the **Containers** plan is set to **On**. This is necessary to have Defender for Containers protect your AWS EKS clusters. Ensure you have fulfilled the [network requirements](defender-for-containers-enable.md?tabs=defender-for-container-eks#network-requirements) for the Defender for Containers plan.

    > [!Note] 
    > Azure Arc-enabled Kubernetes, the Defender Arc extension, and the Azure Policy Arc extension should be installed. Use the dedicated Defender for Cloud recommendations to deploy the extensions (and Arc, if necessary) as explained in [Protect Amazon Elastic Kubernetes Service clusters](defender-for-containers-enable.md?tabs=defender-for-container-eks).


    - (Optional) Select **Configure**, to edit the configuration as required. If you choose to disable this configuration, the `Threat detection (control plane)` feature will be disabled. Learn more about the [feature availability](supported-machines-endpoint-solutions-clouds-containers.md).

1. Select **Next: Configure access**.

1. Download the CloudFormation template.
    
1. Using the downloaded CloudFormation template, create the stack in AWS as instructed on screen.
    
1. Select **Next: Review and generate**.
    
1. Select **Create**.

Defender for Cloud will immediately start scanning your AWS resources and you'll see security recommendations within a few hours. For a reference list of all the recommendations Defender for Cloud can provide for AWS resources, see [Security recommendations for AWS resources - a reference guide](recommendations-reference-aws.md).

::: zone-end


::: zone pivot="classic-connector"

## Availability

|Aspect|Details|
|----|:----|
|Release state:|General availability (GA)|
|Pricing:|Requires [Microsoft Defender for servers](defender-for-servers-introduction.md)|
|Required roles and permissions:|**Owner** on the relevant Azure subscription<br>**Contributor** can also connect an AWS account if an owner provides the service principal details|
|Clouds:|:::image type="icon" source="./media/icons/yes-icon.png"::: Commercial clouds<br>:::image type="icon" source="./media/icons/no-icon.png"::: National (Azure Government, Azure China 21Vianet)|



## Connect your AWS account

Follow the steps below to create your AWS cloud connector. 

### Step 1. Set up AWS Security Hub:

1. To view security recommendations for multiple regions, repeat the following steps for each relevant region.

    > [!IMPORTANT]
    > If you're using an AWS management account, repeat the following three steps to configure the management account and all connected member accounts across all relevant regions

    1. Enable [AWS Config](https://docs.aws.amazon.com/config/latest/developerguide/gs-console.html).
    1. Enable [AWS Security Hub](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-settingup.html).
    1. Verify that data is flowing to the Security Hub. When you first enable Security Hub, it might take several hours for data to be available.

### Step 2. Set up authentication for Defender for Cloud in AWS

There are two ways to allow Defender for Cloud to authenticate to AWS:

- **Create an IAM role for Defender for Cloud** (Recommended) - The most secure method
- **AWS user for Defender for Cloud** - A less secure option if you don't have IAM enabled

#### Create an IAM role for Defender for Cloud
1. From your Amazon Web Services console, under **Security, Identity & Compliance**, select **IAM**.
    :::image type="content" source="./media/quickstart-onboard-aws/aws-identity-and-compliance.png" alt-text="AWS services.":::

1. Select **Roles** and **Create role**.
1. Select **Another AWS account**.
1. Enter the following details:

    - **Account ID** - enter the Microsoft Account ID (**158177204117**) as shown in the AWS connector page in Defender for Cloud.
    - **Require External ID** - should be selected
    - **External ID** - enter the subscription ID as shown in the AWS connector page in Defender for Cloud 

1. Select **Next**.
1. In the **Attach permission policies** section, select the following [AWS managed policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_job-functions.html):

    - SecurityAudit (`arn:aws:iam::aws:policy/SecurityAudit`)
    - AmazonSSMAutomationRole (`arn:aws:iam::aws:policy/service-role/AmazonSSMAutomationRole`)
    - AWSSecurityHubReadOnlyAccess (`arn:aws:iam::aws:policy/AWSSecurityHubReadOnlyAccess`)

1. Optionally add tags. Adding Tags to the user doesn't affect the connection.
1. Select **Next**.

1. In The Roles list, choose the role you created

1. Save the Amazon Resource Name (ARN) for later. 

#### Create an AWS user for Defender for Cloud 
1. Open the **Users** tab and select **Add user**.
1. In the **Details** step, enter a username for Defender for Cloud and ensure that you select **Programmatic access** for the AWS Access Type. 
1. Select **Next Permissions**.
1. Select **Attach existing policies directly** and apply the following policies:
    - SecurityAudit
    - AmazonSSMAutomationRole
    - AWSSecurityHubReadOnlyAccess
    
1. Select **Next: Tags**. Optionally add tags. Adding Tags to the user doesn't affect the connection.
1. Select **Review**.
1. Save the automatically generated **Access key ID** and **Secret access key** CSV file for later.
1. Review the summary and select **Create user**.


### Step 3. Configure the SSM Agent

AWS Systems Manager is required for automating tasks across your AWS resources. If your EC2 instances don't have the SSM Agent, follow the relevant instructions from Amazon:

- [Installing and Configuring SSM Agent on Windows Instances](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-install-ssm-win.html)
- [Installing and Configuring SSM Agent on Amazon EC2 Linux Instances](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-install-ssm-agent.html)


### Step 4. Complete Azure Arc prerequisites
1. Make sure the appropriate [Azure resources providers](../azure-arc/servers/prerequisites.md#azure-resource-providers) are registered:
    - Microsoft.HybridCompute
    - Microsoft.GuestConfiguration

1. Create a Service Principal for onboarding at scale. As an **Owner** on the subscription you want to use for the onboarding, create a service principal for Azure Arc onboarding as described in [Create a Service Principal for onboarding at scale](../azure-arc/servers/onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale).


### Step 5. Connect AWS to Defender for Cloud

1. From Defender for Cloud's menu, open **Environment settings** and select the option to switch back to the classic connectors experience.

    :::image type="content" source="media/quickstart-onboard-gcp/classic-connectors-experience.png" alt-text="Switching back to the classic cloud connectors experience in Defender for Cloud.":::

1. Select **Add AWS account**.
    :::image type="content" source="./media/quickstart-onboard-aws/add-aws-account.png" alt-text="Add AWS account button on Defender for Cloud's multi-cloud connectors page":::
1. Configure the options in the **AWS authentication** tab:
    1. Enter a **Display name** for the connector.
    1. Confirm that the subscription is correct. It's the subscription that will include the connector and AWS Security Hub recommendations.
    1. Depending on the authentication option, you chose in [Step 2. Set up authentication for Defender for Cloud in AWS](#step-2-set-up-authentication-for-defender-for-cloud-in-aws):
        - Select  **Assume Role** and paste the ARN from [Create an IAM role for Defender for Cloud](#create-an-iam-role-for-defender-for-cloud).

            :::image type="content" source="./media/quickstart-onboard-aws/paste-arn-in-portal.png" alt-text="Pasting the ARN file in the relevant field of the AWS connection wizard in the Azure portal.":::

            OR

        - Select **Credentials** and paste the **access key** and **secret key** from the .csv file you saved in [Create an AWS user for Defender for Cloud](#create-an-aws-user-for-defender-for-cloud).
1. Select **Next**.
1. Configure the options in the **Azure Arc Configuration** tab:

    Defender for Cloud discovers the EC2 instances in the connected AWS account and uses SSM to onboard them to Azure Arc. 

    > [!TIP]
    > For the list of supported operating systems, see [What operating systems for my EC2 instances are supported?](#what-operating-systems-for-my-ec2-instances-are-supported) in the FAQ.

    1. Select the **Resource Group** and **Azure Region** that the discovered AWS EC2s will be onboarded to in the selected subscription.
    1. Enter the **Service Principal ID** and **Service Principal Client Secret** for Azure Arc as described here [Create a Service Principal for onboarding at scale](../azure-arc/servers/onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale)
    1. If the machine is connecting to the internet via a proxy server, specify the proxy server IP address or the name and port number that the machine uses to communicate with the proxy server. Enter the value in the format ```http://<proxyURL>:<proxyport>```
    1. Select **Review + create**.

        Review the summary information

        The Tags sections will list all Azure Tags that will be automatically created for each onboarded EC2 with its own relevant details to easily recognize it in Azure. 

        Learn more about Azure Tags in [Use tags to organize your Azure resources and management hierarchy](../azure-resource-manager/management/tag-resources.md).

### Step 6. Confirmation

When the connector is successfully created, and AWS Security Hub has been configured properly:

- Defender for Cloud scans the environment for AWS EC2 instances, onboarding them to Azure Arc, enabling to install the Log Analytics agent and providing threat protection and security recommendations. 
- The Defender for Cloud service scans for new AWS EC2 instances every 6 hours and onboards them according to the configuration.
- The AWS CIS standard will be shown in the Defender for Cloud's regulatory compliance dashboard.
- If Security Hub policy is enabled, recommendations will appear in the Defender for Cloud portal and the  regulatory compliance dashboard 5-10 minutes after onboard completes.


::: zone-end

:::image type="content" source="./media/quickstart-onboard-aws/aws-resources-in-recommendations.png" alt-text="AWS resources and recommendations in Defender for Cloud's recommendations page" lightbox="./media/quickstart-onboard-aws/aws-resources-in-recommendations.png":::


## Monitoring your AWS resources

As you can see in the previous screenshot, Defender for Cloud's security recommendations page displays your AWS resources. You can use the environments filter to enjoy Defender for Cloud's multi-cloud capabilities: view the recommendations for Azure, AWS, and GCP resources together.

To view all the active recommendations for your resources by resource type, use Defender for Cloud's asset inventory page and filter to the AWS resource type in which you're interested:

:::image type="content" source="./media/quickstart-onboard-aws/aws-resource-types-in-inventory.png" alt-text="Asset inventory page's resource type filter showing the AWS options"::: 


## FAQ - AWS in Defender for Cloud

### What operating systems for my EC2 instances are supported?

For a list of the AMIs with the SSM Agent preinstalled see [this page in the AWS docs](https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent-technical-details.html#ami-preinstalled-agent).

For other operating systems, the SSM Agent should be installed manually using the following instructions:

- [Install SSM Agent for a hybrid environment (Windows)](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-install-managed-win.html)
- [Install SSM Agent for a hybrid environment (Linux)](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-install-managed-linux.html)


## Next steps

Connecting your AWS account is part of the multi-cloud experience available in Microsoft Defender for Cloud. For related information, see the following page:

- [Security recommendations for AWS resources - a reference guide](recommendations-reference-aws.md).
- [Connect your GCP projects to Microsoft Defender for Cloud](quickstart-onboard-gcp.md)
