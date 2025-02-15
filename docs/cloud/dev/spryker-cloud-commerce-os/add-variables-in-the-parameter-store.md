---
title: Add variables in the Parameter Store
description: Learn how to define variables in the Parameter Store.
last_updated: May 3, 2023
template: howto-guide-template
keywords: environment variable management
redirect_from:
  - /docs/cloud/dev/spryker-cloud-commerce-os/managing-parameters-in-the-parameter-store.html
  - /docs/cloud/dev/spryker-cloud-commerce-os/define-parameter-and-secret-values-in-sccos-environments.html
  - /docs/cloud/dev/spryker-cloud-commerce-os/add-variables.html
---



Variables, such as parameters and secrets, are used for multiple purposes, like storing mail server details or providing Composer authentication details to the build and deploy process securely.

{% info_block infoBox %}

This feature is part of a gradual rollout and will be available to everyone eventually. We will notify your team once your project is onboarded.

{% endinfo_block %}

## Customer-owned and Spryker-owned variables

There are two types of environment variables in the Spryker Cloud Commerce OS: those owned by the customer and those owned by Spryker.

Customer-owned variables are created and managed by you—the onboarded customer or implementation partner. You have full control over these variables and can add or edit them according to your needs. However, keep in mind that changes to these variables don't automatically propagate into the running environment. To apply changes made to your environment variables, you need to run an ECS-updater-* pipeline to bring them to the containers (full re-deploy would also do the trick, of course). This process doesn't require intervention of the Spryker Cloud or support team.

If there is a need to modify a Spryker-owned environment variable, it must be done through the Spryker Cloud or support team, as it needs to be coordinated with our DevOps team to ensure proper application and consistency.

By adhering to these guidelines, you can effectively manage your environment variables without risking the system stability. It also helps in maintaining a seamless experience while working with the Spryker Cloud Commerce OS.

Please remember that improper management of environment variables can lead to unexpected issues. Therefore, it is recommended to consult the Spryker Cloud or support team for any complex or system-critical changes.

## Naming convention for variables

Variables must follow this naming convention: `/{project}/{environment}/{type}/{bucket}/{grant}/{variable_name}`.

{% info_block warningBox "Reserved variables" %}

Reserved variables are variable names that have special meaning in Spryker Cloud Commerce OS. Since their names are reserved, you can't define your own variables using these names.
<BR> The reserved variable names are the following:
* `DUMMY_INIT`
* `SPRYKER_*`

If you are already using these reserved variables in your code, you need to change their names to avoid any service issues.

{% endinfo_block %}

Placeholder description:

* `type`: defines the type of a variable. Possible values:
    * `config`: parameter
    * `secret`: secret

* `bucket`: defines what services a variable is used for. Possible values:
    * `common`: used by all the buckets.
    * `app`: used only by application services.
    * `scheduler`: used by the scheduler.

* `grant`: Defines access permissions to variables. Possible values:
    * `public`: readable and writable
    * `limited`: readable
    * `internal`: hidden

Path examples:

* /fashion_club_store/staging/config/common/limited/composer_pass

* /deans_jeans/prod/config/app/public/mail_host

## Variable path hierarchy

Path hierarchy is needed to cover the cases when several variables with the same name are declared. If several variables with the same name are declared, the variable with a higher priority applies. The following rules define the priority of variables:

1. For any `type` and `bucket`, the priority is `public` > `limited` > `internal`.
2. Foy any `bucket`, the priority is `bucket` > `common`.
3. For any variable with the same name, the priority is `secret` > `config`.

The following variables are arranged from lower to higher priority:

* /{project}/{environment}/config/common/internal/{variable_name}

* /{project}/{environment}/config/common/limited/{variable_name}

* /{project}/{environment}/config/common/public/{variable_name}

* /{project}/{environment}/config/{app | scheduler}/internal/{variable_name}

* /{project}/{environment}/config/{app | scheduler}/limited/{variable_name}

* /{project}/{environment}/config/{app | scheduler}/public/{variable_name}

* /{project}/{environment}/secret/common/internal/{variable_name}

* /{project}/{environment}/secret/common/limited/{variable_name}

* /{project}/{environment}/secret/common/public/{variable_name}

* /{project}/{environment}/secret/{app | scheduler}/internal/{variable_name}

* /{project}/{environment}/secret/{app | scheduler}/limited/{variable_name}

* /{project}/{environment}/secret/{app | scheduler}/public/{variable_name}

## Add variables

The following sections describe how to add parameters and secrets for different resources.

### Add parameters to all resource types

1. In the AWS Management Console, go to **Services&nbsp;<span aria-label="and then">></span> Parameter Store**.
2. In the **My parameters** pane, click **Create parameter**.
    This opens the **Create parameter** page.
3. For **Name**, enter `/{project}/{environment}/config/common/public/{variable_name}`.
    Make sure to replace the placeholders based on your requirements.
4. Optional: For **Description**, enter a description of the variable. This may be a note about what this variable is used for.
5. For **Type**, select a type of the variable based on your requirements.    
6. For **Value**, enter the value of the variable.
7. Click **Create parameter**.
    This opens the **Parameter Store** page with a success message displayed.    
8. Go to **Services**&nbsp;<span aria-label="and then">></span> **CodePipeline**.
9. On the **Pipelines** page, select the **NORMAL_Deploy_Spryker_{project}-{environemt}** pipeline.
10. On the pipeline's page, click **Release change**.
11. In the **Release change** window, click **Release**.
    After the pipeline finishes running, the variable gets available for your application.

### Add secrets to all resource types

1. In the AWS Management Console, go to **Services&nbsp;<span aria-label="and then">></span> Parameter Store**.
2. In the **My parameters** pane, click **Create parameter**.
    This opens the **Create parameter** page.
3. For **Name**, enter `/{project}/{environment}/secret/common/public/{variable_name}`.
    Make sure to replace the placeholders based on your requirements.
4. Optional: For **Description**, enter a description of the variable. This may be a note about what this variable is used for.
    This may be a note about what this variable is used for.
5. For **Type**, select **SecureString**.
6. For **Value**, enter the value of the variable.
7. Click **Create parameter**.
    This opens the **Parameter Store** page with a success message displayed.    
8. Go to **Services&nbsp;<span aria-label="and then">></span> CodePipeline**.
9. On the **Pipelines** page, select the **NORMAL_Deploy_Spryker_{project}-{environemt}** pipeline.
10. On the pipeline's page, click **Release change**.
11. In the **Release change** window, click **Release**.
    After the pipeline finishes running, the variable gets available for your application.

### Add parameters to ECS applications

1. In the AWS Management Console, go to **Services&nbsp;<span aria-label="and then">></span> Parameter Store**.
2. In the **My parameters** pane, click **Create parameter**.
    This opens the **Create parameter** page.
3. For **Name**, enter `/{project}/{environment}/config/app/public/{variable_name}`.
    Make sure to replace the placeholders based on your requirements.
4. Optional: For **Description**, enter a description of the variable. This may be a note about what this variable is used for.
    This may be a note about what this variable is used for.
5. For **Type**, select a type of the variable based on your requirements.    
6. For **Value**, enter the value of the variable.
7. Click **Create parameter**.
    This opens the **Parameter Store** page with a success message displayed.    
8. Go to **Services&nbsp;<span aria-label="and then">></span> CodePipeline**.
9. On the **Pipelines** page, select the **ECS-updater-{project}-{environemt}** pipeline.
10. On the pipeline's page, click **Release change**.
11. In the **Release change** window, click **Release**.
    After the pipeline finishes running, the variable gets available for your application.

### Adding secrets to ECS applications

1. In the AWS Management Console, go to **Services&nbsp;<span aria-label="and then">></span> Parameter Store**.
2. In the **My parameters** pane, click **Create parameter**.
    This opens the **Create parameter** page.
3. For **Name**, enter `/{project}/{environment}/secret/app/public/{variable_name}`.
    Make sure to replace the placeholders based on your requirements.
4. Optional: For **Description**, enter a description of the variable. This may be a note about what this variable is used for.
    This may be a note about what this variable is used for.
5. For **Type**, select **SecureString**.
6. For **Value**, enter the value of the variable.
7. Click **Create parameter**.
    This opens the **Parameter Store** page with a success message displayed.    
8. Go to **Services**&nbsp;<span aria-label="and then">></span> **CodePipeline**.
9. On the **Pipelines** page, select the **ECS-updater-{project}-{environemt}** pipeline.
10. On the pipeline's page, click **Release change**.
11. In the **Release change** window, click **Release**.
    After the pipeline finishes running, the variable gets available for your application.    

### Add parameters and secrets to Scheduler

1. In the AWS Management Console, go to **Services&nbsp;<span aria-label="and then">></span> Parameter Store**.
2. In the **My parameters** pane, click **Create parameter**.
    This opens the **Create parameter** page.
3. For **Name**, enter one of the following:
    * Variable: `/{project}/{environment}/config/scheduler/public/{variable_name}`
    * Secret: `/{project}/{environment}/secret/scheduler/public/{variable_name}`
        Make sure to replace the placeholders based on your requirements.
4. Optional: For **Description**, enter a description of the variable. This may be a note about what this variable is used for.
    This may be a note about what this variable is used for.
5. For **Type**, select **SecureString**.
6. For **Value**, enter the value of the variable.
7. Click **Create parameter**.
    This opens the **Parameter Store** page with a success message displayed.    
8. Go to **Services&nbsp;<span aria-label="and then">></span> CodePipeline**.
9. On the **Pipelines** page, select the **Rollout_Scheduler_{project}-{environemt}** pipeline.
10. On the pipeline's page, click **Release change**.
11. In the **Release change** window, click **Release**.
    After the pipeline finishes running, the variable gets available for your application.  
