# Azure CI/CD Pipelines for Terraform Deployment

## Table of Contents
1. [Project Overview](#project-overview)
2. [Prerequisites](#prerequisites)
3. [Quick Start](#quick-start)
4. [Get Started](#get-started)
5. [CI/CD Pipeline Test](#cicd-pipeline-test)
6. [Main Infrastructures Test](#main-infrastructures-test)
7. [Destroy Infrastructures](#destroy-infrastructures)
8. [Project Summary](#project-summary)
9. [Troubleshooting](#troubleshooting)

## Project Overview
This project builds upon my previously created [Databricks ETL Pipeline with Auto Loader](https://github.com/johadamas/coffeeshop-e2e-pipeline.git). It extends the functionality by integrating Azure DevOps CI/CD pipelines to automate the deployment of Terraform configurations and Databricks notebooks. This ensures consistent infrastructure provisioning and workflow updates whenever changes are committed to the repository.

### A. Azure Pipeline Flows
![Pipeline Flow](/images/_DataOps-CICD.png "Azure Pipelines")

#### **Build Pipeline (`azure-pipelines.yml`)**
**Pipeline Stages:**

- **Terraform Init**:
  Configures Terraform and sets up the backend for state storage.

- **Terraform Validate**:
  Ensures the Terraform configuration syntax and logic are correct.

- **Terraform Plan**:
  Creates a Terraform plan file that outlines the changes Terraform will make.

- **Artifact Archival**:
  Archives the plan file and other relevant files for use in the release pipeline.

#### Trigger:
- Triggered by changes to the `main` branch.

---

#### **Release Pipeline (`release-pipeline.yml`)**
**Pipeline Stages:**

- **Deploy Infrastructures**:
  - Extracts the build artifact from the CI pipeline.
  - Installs Terraform.
  - Initializes Terraform and applies the plan to deploy the main infrastructure.
  - Retries the `apply` step twice on failure for added robustness.
  
- **Destroy Infrastructures (Optional)**:
  - Extracts the build artifact from the CI pipeline.
  - Installs Terraform.
  - Initializes Terraform and destroys the deployed resources.
  - This stage needs approval first

#### Trigger:
- Triggered by changes to the `main` branch.

### B. Main Infrastructures
This project will create Databricks ETL Workflow as the Main Infrastructures with Azure DevOps CI/CD pipelines to automate the deployment of Terraform configurations and Databricks notebooks

![Main Infrastructures](/images/_Main-Infrastructure.png "Main Infrastructures")

## Prerequisites
1. **Azure Subscriptions**
    - Required to provision all primary and backend infrastructure components

2. **Azure DevOps Account**
    - Needed to set up the project in Azure DevOps. Ensure that your account supports build pipeline parallelism; otherwise, the build pipeline won't run

3. **Backend Infrastructures**
    - Use Azure Storage to store the remote Terraform state
    - You can either create the backend manually (e.g., a Resource Group and Storage Account) or clone my [Backend Infrastructure](https://github.com/johadamas/terraform-backend-infra.git) repo to deploy it quickly

---

## Quick Start
1. **Clone this repository**:
    ```bash
    git clone https://github.com/johadamas/azure-e2e-dataops.git
    ```

2. **Upload `terraform.tfvars`**:
- Create a `terraform.tfvars` file with the following content, and then upload the file to your Azure DevOps repo

    ```hcl
    subscription_id = "YOUR_SUBSCRIPTION_ID"
    user_email      = "YOUR_EMAIL_ACCOUNT"
    ```

3. **Create Build Pipeline**:
- In Azure DevOps, create a new Build Pipeline using the `azure-pipelines.yml` file

4. **Create Release Pipeline**:
- In Azure DevOps, create a new Release Pipeline using the `release-pipeline.yml` file

5. **Trigger the Pipeline**:
- Commit changes to the `main` branch to trigger the Build and Release Pipelines

## Get Started:

### 1. Create a Project:  
- Go to your **Azure DevOps** account and create a new project

    ![](/images/1.create_project.png "")


### 2. Import Repo:  
- Once the project is created, import or clone the repository using the **GitHub** URL

    ![](/images/2.import_repo.gif "")


### 3. Upload .tfvars file:  
- After successfully importing the repository, upload your `terraform.tfvars` file. It should contain the following:

    ```hcl
    subscription_id = "YOUR_SUBSCRIPTION_ID"
    user_email      = "YOUR_EMAIL_ACCOUNT"
    ```
- Here is the detailed step in GIF:

    ![](/images/3.upload_tfvars.gif "")


### 4. Create Build Pipeline:  
**A. Build Setup:**
- Now that our `terraform.tfvars` file has been successfully uploaded, we can start creating the **Build Pipeline** using the **Starter Pipeline**

    ![](/images/4.setup_build.gif "")

**B. Build Assistant**:
- By default, the starter pipeline provides us with a basic setup including the **trigger, pool, and stages**. However, we still need to define the necessary tasks ourselves

- To create the required tasks, we can use the **Build Assistant** to help construct our `azure-pipelines.yml` file

    ![](/images/5.build_yaml.gif "")

**C. Complete azure-pipelines.yml File**:
- Here is the full YAML configuration:

    ```yaml
    trigger:
    - main

    pool:
    vmImage: 'ubuntu-latest'  # Microsoft-hosted agent

    stages:
    - stage: Build
        jobs:
        - job: Build
            pool:
            vmImage: 'ubuntu-latest'  # Microsoft-hosted agent
            steps:
            - checkout: self

            - task: TerraformTaskV4@4
            displayName: Terraform Init
            inputs:
                provider: 'azurerm'
                command: 'init'
                backendServiceArm: 'YOUR_SUBSCRIPTION_ID'
                backendAzureRmResourceGroupName: 'YOUR_BACKEND_RESOURCE_GROUP_NAME'
                backendAzureRmStorageAccountName: 'YOUR_BACKEND_STORAGE_ACCOUNT_NAME'
                backendAzureRmContainerName: 'YOUR_BACKEND_STORAGE_CONTAINER_NAME'
                backendAzureRmKey: 'YOUR_BACKEND_TFSTATE_KEY_NAME'

            - task: TerraformTaskV4@4
            displayName: Terraform Validate
            inputs:
                provider: 'azurerm'
                command: 'validate'
            
            - task: TerraformTaskV4@4
            displayName: Terraform Plan
            inputs:
                provider: 'azurerm'
                command: 'plan'
                commandOptions: '-out $(Build.SourcesDirectory)/tfplanfile'
                environmentServiceNameAzureRM: 'YOUR_SUBSCRIPTION_ID'
            
            - task: ArchiveFiles@2
            displayName: Archiving Artifact
            inputs:
                rootFolderOrFile: '$(Build.SourcesDirectory)/'
                includeRootFolder: false
                archiveType: 'zip'
                archiveFile: '$(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip'
                replaceExistingArchive: true

            - task: PublishBuildArtifacts@1
            displayName: Publishing Artifact
            inputs:
                PathtoPublish: '$(Build.ArtifactStagingDirectory)'
                ArtifactName: '$(Build.BuildId)-build'
                publishLocation: 'Container'
    ```


### 5. Build Pipeline Run:
**A. Start the Build Pipeline:**
- After the `azure-pipelines.yml` is complete, save and run the pipeline

- Before the jobs run, you must grant access to the service connection used to start the build pipeline

    ![](/images/6.run_build.gif "")

**B. Build Pipeline Complete:**
- Once the build pipeline completes successfully, you will see green checkmarks on the left-hand side, indicating that all tasks have executed correctly

- The pipeline will also generate an `artifact`, which will be used later to create the **Release Pipeline**

    ![](/images/8.finish_build.gif "")


**C. Terraform Plan Output:**
- Below is an example of the `terraform plan` output, showing 24 resources that will be created as part of our main infrastructure

    ![](/images/9.build_tf_plan.png "")


### 6. Create Release Pipeline: 
**A. Pipeline Setup:**
- To create the **Release Pipeline**, we first need to add the artifact generated by the **Build Pipeline**. This artifact will serve as the source for our deployment

- Once the artifact is added, we can enable the **Continuous Deployment (CD) trigger**

- The CD trigger will automatically execute the release pipeline whenever there are **new changes** or a **new build** on the default branch

    ![](/images/10.setup_release.gif "")

**B. Deploy Stage:**

This stage consists of **1 job and 4 tasks**. The Agent job will automatically download the artifact generated in the **Build Pipeline**

- `Extract Files`: This task extracts the archived **Build Artifact**, making it available for deployment

    ![](/images/11.deploy_extract.gif "")

- `Install Terraform`: Installs Terraform inside the agent. Here, we specify version 1.9.8 to match the version used in my previously created [Databricks ETL Pipeline with Auto Loader](https://github.com/johadamas/coffeeshop-e2e-pipeline.git)

    ![](/images/12.deploy_install_tf.gif "")

- `Terraform Init`: Initializes Terraform by connecting to **Azure Subscriptions** (or **Service Connection**) and the **Backend Infrastructure** (Storage Account to store `tfstate` files)

    ![](/images/13.deploy_tf_init.gif "")

- `Terraform Apply`: Applies the Terraform configuration to provision the **Main Infrastructure**. The `--auto-approve` flag must be added to **Additional Command Arguments**, otherwise, the process will require manual approval and won’t execute automatically

    ![](/images/13.deploy_tf_apply.png "")

- `Apply Retries`: I’ve added 2 retries for this task because Terraform occasionally fails to upload Databricks notebooks due to transient issues

    ![](/images/13.deploy_tf_apply2.png "")

**C. Destroy Stage:**

This stage consists of one job and four tasks, following the same structure as the Deploy Stage, but with key modifications:

- `Clone the Deploy Stage`: Instead of creating a new stage manually we can clone the Deploy Stage for efficiency, simply by clicking on the **Clone** tab

    ![](/images/14.clone_stage.gif "")

- `Terraform Destroy`: Replace the **Terraform Apply** task with **Terraform Destroy** to tear down the infrastructure

    ![](/images/15.destroy_stage_retry.png "")

- `Destroy Stage Trigger`: To ensures that the Destroy Stage **only executes with explicit approval**

    ![](/images/15.destroy_stage_trigger.gif "")

**D. Complete release-pipelines.yml File**:
- Here is the full YAML configuration:

    ```yaml
    # Deploy Infrastructures Stage
    # Extract Files
    steps:
    - task: ExtractFiles@1
    displayName: 'Extract files '
    inputs:
        archiveFilePatterns: '_$(Build.DefinitionName)/$(Build.BuildId)-build/$(Build.BuildId).zip'
        destinationFolder: '$(System.DefaultWorkingDirectory)/$(Build.BuildId)-build'

    # Install Terraform
    steps:
    - task: ms-devlabs.custom-terraform-tasks.custom-terraform-installer-task.TerraformInstaller@1
    displayName: 'Install Terraform v1.9.8'
    inputs:
        terraformVersion: 1.9.8

    # Terraform Init
    steps:
    - task: ms-devlabs.custom-terraform-tasks.custom-terraform-release-task.TerraformTaskV4@4
    displayName: 'Terraform : init'
    inputs:
        workingDirectory: '$(System.DefaultWorkingDirectory)/$(Build.BuildId)-build'
        backendAzureRmUseEnvironmentVariablesForAuthentication: false
        backendAzureRmUseEntraIdForAuthentication: false
        backendServiceArm: 'YOUR_SUBSCRIPTION_ID'
        backendAzureRmResourceGroupName: 'YOUR_BACKEND_RESOURCE_GROUP_NAME'
        backendAzureRmStorageAccountName: 'YOUR_BACKEND_STORAGE_ACCOUNT_NAME'
        backendAzureRmContainerName: 'YOUR_BACKEND_STORAGE_CONTAINER_NAME'
        backendAzureRmKey: 'YOUR_BACKEND_TFSTATE_KEY_NAME'

    # Terraform Apply
    steps:
    - task: ms-devlabs.custom-terraform-tasks.custom-terraform-release-task.TerraformTaskV4@4
    displayName: 'Terraform : apply'
    inputs:
        command: apply
        workingDirectory: '$(System.DefaultWorkingDirectory)/$(Build.BuildId)-build'
        environmentServiceNameAzureRM: 'YOUR_SUBSCRIPTION_ID'
        backendAzureRmUseEnvironmentVariablesForAuthentication: false
        backendAzureRmUseEntraIdForAuthentication: false
    retryCountOnTaskFailure: 2


    # Destroy Infrastructures Stage
    # Extract Files
    steps:
    - task: ExtractFiles@1
    displayName: 'Extract files '
    inputs:
        archiveFilePatterns: '_$(Build.DefinitionName)/$(Build.BuildId)-build/$(Build.BuildId).zip'
        destinationFolder: '$(System.DefaultWorkingDirectory)/$(Build.BuildId)-build'

    # Install Terraform
    steps:
    - task: ms-devlabs.custom-terraform-tasks.custom-terraform-installer-task.TerraformInstaller@1
    displayName: 'Install Terraform v1.9.8'
    inputs:
        terraformVersion: 1.9.8

    # Terraform Init
    steps:
    - task: ms-devlabs.custom-terraform-tasks.custom-terraform-release-task.TerraformTaskV4@4
    displayName: 'Terraform : init'
    inputs:
        workingDirectory: '$(System.DefaultWorkingDirectory)/$(Build.BuildId)-build'
        backendAzureRmUseEnvironmentVariablesForAuthentication: false
        backendAzureRmUseEntraIdForAuthentication: false
        backendServiceArm: 'YOUR_SUBSCRIPTION_ID'
        backendAzureRmResourceGroupName: 'YOUR_BACKEND_RESOURCE_GROUP_NAME'
        backendAzureRmStorageAccountName: 'YOUR_BACKEND_STORAGE_ACCOUNT_NAME'
        backendAzureRmContainerName: 'YOUR_BACKEND_STORAGE_CONTAINER_NAME'
        backendAzureRmKey: 'YOUR_BACKEND_TFSTATE_KEY_NAME'

    # Terraform Destroy
    steps:
    - task: ms-devlabs.custom-terraform-tasks.custom-terraform-release-task.TerraformTaskV4@4
    displayName: 'Terraform : destroy'
    inputs:
        command: destroy
        workingDirectory: '$(System.DefaultWorkingDirectory)/$(Build.BuildId)-build'
        environmentServiceNameAzureRM: 'YOUR_SUBSCRIPTION_ID'
        backendAzureRmUseEnvironmentVariablesForAuthentication: false
        backendAzureRmUseEntraIdForAuthentication: false
    retryCountOnTaskFailure: 2
    ```

## CI/CD Pipeline Test:

In this section, we will test our **end-to-end CI/CD pipeline** by making a small change in the Git repository to trigger the workflow

**A. Build Pipeline (CI):**
- To test the pipeline, we edit the `README.md` file in the Git repository by adding a simple text change

- This modification will **automatically trigger the Build Pipeline (CI)** upon committing the changes

    ![](/images/16.cicd_build.gif "")

- As soon as we commit the changes, the **Build Pipeline** starts running automatically

**B. Release Pipeline (CD):**
- Once the **Build Pipeline (CI)** successfully completes, the **Release Pipeline (CD)** is automatically triggered 

    ![](/images/17.cicd_release.gif "")

- While the Release Pipeline is running, we can observe that Azure resources for our **Main Infrastructure** are gradually being created during the **Terraform Apply** task, with the `resource_id = doberman`

    ![](/images/18.cicd_run.gif "")

- As shown in the screenshot below, the **Release Pipeline** successfully completes, even though the **Terraform Apply** task encountered an error

    ![](/images/19.build_error.png "")

- The error states that Terraform failed to create a Databricks notebook because the User folder is protected

    ![](/images/19.build_error_2.png "")

- Fortunately, since we previously added retry attempts to the Terraform Apply task, the pipeline was able to recover, and all resources were successfully created as intended

    ![](/images/19.build_error_3.png "")

**C. Pending Stage:**
- Once the **Deploy Stage** is complete, the **Destroy Stage** enters a **"Pending Approval" state**

- This ensures that the **Destroy Stage** does not execute automatically—it requires manual approval before proceeding

    ![](/images/20.cicd_deploy2.gif "")


## Main Infrastructures Test: 

In this section, we will test the **Main Infrastructure**, which consists of the **Databricks ETL Pipeline**, starting from data generation to validation

**Generate Fake Data :**
- We will use the `generate_and_load_new_data` Airflow DAG from our [previous project](https://github.com/johadamas/coffeeshop-e2e-pipeline.git) to generate and load the data into the **landing container**

    ![](/images/21.etl_test_1.gif "")

- Once the data is successfully loaded into the landing container, we manually trigger the **Coffee Shop ETL Pipeline** from the **Databricks UI**

    ![](/images/22.etl_test_2.gif "")

- The screenshot below confirms that our **ETL Pipeline** completed successfully, and all Databricks notebooks ran without errors

    ![](/images/23.etl_finish_1.png "")

- The screenshot below shows that our gold dataset was successfully loaded into **Delta Tables** in the **Databricks Catalog**

    ![](/images/24.delta_tables.png "")

- The screenshot below demonstrates that we can query data from the **Delta Tables** without any issues 

    ![](/images/25.etl_test_1.png "")

- The screenshot below confirms that the number of distinct transactions is `1000`, which matches the input parameter defined in the data generation function within the **Airflow DAG**

    ![](/images/25.etl_test_2.png "")

## Destroy Infrastructures:

In this section, we will execute the **Destroy Stage** to tear down our Main Infrastructures from our **Release Pipeline**, following a step-by-step approach

**Approve Destroy:**
- Click the **"Approve"** button located directly below the Destroy Stage

- This action will immediately trigger the agent, executing the Destroy Stage

    ![](/images/26.cicd_destroy_1.gif "")

- As the **Destroy Stage** runs, we can observe that Azure resources for our Main Infrastructure are gradually being deleted by the **Terraform Destroy task**

    ![](/images/26.cicd_destroy2.gif "")

- The screenshot below shows that the Release Pipeline successfully completed, even though the Terraform Destroy task encountered an error

    ![](/images/27.destroy_error_1.png "")

- The error message states that Terraform failed to purge Azure Key Vault secrets

    ![](/images/27.destroy_error_2.png "")

- Fortunately, since we previously added retry attempts to the Terraform Destroy task, the pipeline was able to recover automatically, and all resources were successfully destroyed

    ![](/images/28.destroy_finish.png "")

## Project Summary: 
Our CI/CD pipeline performed effectively, despite encountering some errors. However, we successfully mitigated these issues by implementing retry mechanisms, ensuring pipeline stability

Overall, the pipeline is well-optimized, achieving:
- 100% Build Success
- 100% Deployment Success

    ![](/images/29.project_stats.png "")

## Troubleshooting
### Common Issues and Solutions

1. **Terraform-related notebooks (e.g., uploading Databricks notebooks, purging Key Vault secrets, or permission issues)**

   - **Solution**: Ensure the user folder is not protected or add retries to the pipeline

2. **Terraform Destroy takes too long (10+ minutes) to purge Key Vault secrets**

    - **Current Status**: This issue occurs because Azure enforces a soft delete and purge protection policy on Key Vault secrets, which can slow down Terraform operations

    - **Possible Solutions:**
        - Manually delete the secrets or add retries to the pipeline. Additionally, update the Terraform provider configuration:

            ```hcl
            provider "azurerm" {
                features {
                    key_vault {
                        purge_soft_delete_on_destroy    = false
                        recover_soft_deleted_key_vaults = false
                    }
                }
            }
            ```
            
## License
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
