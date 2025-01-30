# Azure Pipelines for Terraform Deployment

## Project Overview
This project builds upon my previously created [Databricks ETL Pipeline with Auto Loader](https://github.com/johadamas/coffeeshop-e2e-pipeline.git). It extends the functionality by integrating Azure DevOps CI/CD pipelines to automate the deployment of Terraform configurations and Databricks notebooks. This ensures consistent infrastructure provisioning and workflow updates whenever changes are committed to the repository.

## Pipelines Overview

![Pipeline Flow](/images/_DataOps-CICD.png "Azure Pipelines")

### 1. **Build Pipeline (`azure-pipelines.yml`)**

#### Steps:
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

### 2. **Release Pipeline (`release-pipeline.yml`)**

#### Stages:
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

---

## Prerequisites
1. **Azure Subscriptions**
    - Required to provision all primary and backend infrastructure components

2. **Azure DevOps Account**
    - Needed to set up the project in Azure DevOps. Ensure that your account supports build pipeline parallelism; otherwise, the build pipeline won't run

3. **Backend Infrastructures**
    - Use Azure Storage to store the remote Terraform state
    - You can either create the backend manually (e.g., a Resource Group and Storage Account) or clone my [Backend Infrastructure](https://github.com/johadamas/terraform-backend-infra.git) repo to deploy it quickly

---

## Main Infrastructures
This project will create Databricks ETL Workflow as the Main Infrastructures with Azure DevOps CI/CD pipelines to automate the deployment of Terraform configurations and Databricks notebooks

![Pipeline Flow](/images/_Main-Infrastructure.png "Main Infrastructures")

## Project Flows:

### 1. Create a Project:  
The Airflow contains two DAGs:
- `project_setup_pipeline`:

    ![](/images/1.create_project.png "")


### 2. Import Repo:  
The Airflow contains two DAGs:
- `project_setup_pipeline`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/2.import_repo.gif "")


### 3. Upload .tfvars file:  
The Airflow contains two DAGs:
- `project_setup_pipeline`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/3.upload_tfvars.gif "")


### 4. Build Pipeline:  
**Setup Build Pipeline:**
- `project_setup_pipeline`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/4.setup_build.gif "")

**Build azure-pipelines.yml**:
- `project_setup_pipeline`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/5.build_yaml.gif "")


### 5. Build Pipeline Run: 
The Airflow contains two DAGs:
- `project_setup_pipeline`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/6.run_build.gif "")

The Airflow contains two DAGs:
- `project_setup_pipeline`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/8.finish_build.gif "")

The Airflow contains two DAGs:
- `project_setup_pipeline`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/9.build_tf_plan.png "")


### 6. Release Pipeline: 
**Pipeline Setup:**
- `project_setup_pipeline`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/10.setup_release.gif "")

**Deploy Stage:**
- `Extract Files`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/11.deploy_extract.gif "")

- `Install Terraform`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/12.deploy_install_tf.gif "")

- `Terraform Init`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/13.deploy_tf_init.gif "")

- `Terraform Apply`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/13.deploy_tf_apply.png "")

**Destroy Stage:**
- `Clone the Deploy Stage`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/14.clone_stage.gif "")

- `Terraform Destroy`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/15.destroy_stage_retry.png "")

- `Destroy Stage Trigger`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/15.destroy_stage_trigger.gif "")


### 7. CICD Pipeline Test: 
**Build Pipeline:**
- `project_setup_pipeline`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/16.cicd_build.gif "")

**Release Pipeline:**
- `release pipeline start`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/17.cicd_release.gif "")

- `release pipeline run`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/18.cicd_run.gif "")

- `release pipeline complete`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/19.build_error.png "")

- `release pipeline error`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/19.build_error_2.png "")

- `release pipeline retry`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/19.build_error_3.png "")

**Pending Destroy Stage:**
- `Destroy Infra approval`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/20.cicd_deploy2.gif "")


### 8. Main Infrastructures: 
**Databricks Pipeline Test:**
- `generate fake data`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/21.etl_test_1.gif "")

- `etl pipeline run`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/22.etl_test_2.gif "")

- `etl pipeline succeed`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/23.etl_finish_1.png "")

- `delta tables`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/24.delta_tables.png "")

- `etl result check`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/25.etl_test_1.png "")

- `etl result check 2`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/25.etl_test_2.png "")

### 9. Destroy Main Infrastructures: 
**Databricks Pipeline Test:**
- `destroy stage approved`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/26.cicd_destroy_1.gif "")

- `destroy stage run`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/26.cicd_destroy2.gif "")

- `destroy stage complete`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/27.destroy_error_1.png "")

- `destroy stage error`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/27.destroy_error_2.png "")

- `destroy stage error`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/28.destroy_finish.png "")

### 10. Project Summary: 
- `Pipelines Status`: Runs terraform task, generates fake coffee shop sales data and then upload the data to the landing container in ADLS

    ![](/images/29.project_stats.png "")