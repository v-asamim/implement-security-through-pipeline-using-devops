---
lab:
    title: Configure agents and agent pools for secure pipelines
    module: 'Module 3: Configure secure access to pipeline resources'
---

# Configure agents and agent pools for secure pipelines

In this lab, you will learn how to configure Azure DevOps agents and agent Pools and manage permissions for those pools. Azure DevOps Agent Pools provide the resources to run your build and release pipelines.

These exercises take approximately **25** minutes.

## Before you start

You'll need an Azure subscription, Azure DevOps organization, and the eShopOnWeb application to follow the labs.

- Follow the steps to [validate your lab environment](APL2001_M00_Validate_Lab_Environment.md).
- PAT token for the agent configuration.

## Instructions

You'll create agents and configure self-hosted agents using Windows. If you want to configure agents on Linux or MacOS, follow the instructions in the [Azure DevOps documentation](https://docs.microsoft.com/azure/devops/pipelines/agents/v2-linux).

During the configuration, keep in mind the following:

- **Maintain separate agents per project**: Each agent can only be tied to one pool. While sharing agent pools across projects can save on infrastructure costs, it also creates the risk of lateral movement. Therefore, it's best to have separate agent pools with dedicated agents for each project to prevent cross-contamination.
- **Utilize low-privileged accounts for running agents**: Running an agent under an identity with direct access to Azure DevOps resources can pose security threats. Operating the agent under a non-privileged local account like Network Service is advisable, which minimizes the risk.
- **Beware of misleading group names**: The "Project Collection Service Accounts" group in Azure DevOps is a potential security risk. Running agents using an identity that's part of this group and backed by Azure AD can jeopardize the security of your entire Azure DevOps organization.
- **Avoid high-privileged accounts for self-hosted agents**: Using high-privileged accounts to run self-hosted agents, particularly for accessing secrets or production environments, can expose your system to severe threats if a pipeline is compromised.
- **Prioritize security**: To safeguard your systems, use the least privileged account to run self-hosted agents. For instance, consider using your machine account or a managed service identity. It's also advisable to allow Azure Pipelines to handle access to secrets and environments.

### Exercise 1: Create agents and configure agent pools

In this exercise, you will create an agent and configure agent pools.

#### Task 1: Create an agent pool

1. Navigate to the Azure DevOps portal at https://dev.azure.com and open your organization.
2. Open the eShopOnWeb project, and click on "Project settings" from the left-side bottom menu.
3. From Pipelines > Agent Pools, click on the "Add pool" button.
4. Choose the "Self-hosted" pool type.
5. Provide a name for the agent pool, such as "eShopOnWebSelfPool," and add an optional description.
6. Leave the "Grant access permission to all pipelines" option unchecked.

    ![Screenshot showing add agent pool options with self-hosted type.](media/create-new-agent-pool-self-hosted-agent.png)

7. Click on "Create" button to create the agent pool.

#### Task 2: Create an agent

1. Click on the newly created agent pool, and then click on the "Agents" tab.
2. Click on the "New agent" button and then "Download" button from the "Download agent" in the new pop-up window.
3. Follow the installation instructions to install the agent on your machine from the pop-up window.
   1. Run the following commands from Powershell to create a new agent folder in your machine.

        ```powershell
        mkdir agent ; cd agent        
        ```

        > [!NOTE]
        > Make sure you are in the root folder of your user profile or the folder where you want to install the agent.

   2. If you choose the "Download" folder in your machine, from Powershell, run the suggested command:

        ```powershell
        Add-Type -AssemblyName System.IO.Compression.FileSystem ; [SysteIO.Compression.ZipFile]::ExtractToDirecto("$HOME\Downloads\vsts-agent-win-x64-3.220.2.zip", "$PWD")
        
        ``
        > [!NOTE]
        > If you downloaded the agent to a different location, replacthe path in the above command.

#### Task 3: Create a PAT token

Before configuring your agent, create a new PAT token or choose an existing one. To create a new PAT token, follow the steps below:

1. Navigate to the Azure DevOps portal at https://dev.azure.com and open your organization.
2. Navigate to the eShopOnWeb project, and click on "User settings" from the right-side top menu (left of your user profile picture).
3. Click on the "Personal Access Tokens" menu.

    ![Screenshot showing the personal access tokens menu.](media/personal-access-token-menu.png)

4. Click on the "New Token" button.
5. Provide a name for the token, such as "eShopOnWebToken".
6. Select the Azure DevOps organization you want to use the token.
7. Set the expiration date for the token (only used to configure the agent).
8. Select the custom defined scope.
9. Click to show all scopes.
10. Select the "Agent Pools (Read & Manage)" scope.
11. Click on the "Create" button to create the token.
12. Copy the token value and save it in a safe place (you will not be able to see it again. You can only regenerate the token).

    ![Screenshot showing the personal access token configuration.](media/personal-access-token-configuration.png)

    > [!IMPORTANT]
    > Use the last privilege option, "Agent Pools (Read & Manage)," only for the agent configuration. Also, make sure you set the minimum expiration date for the token if it's the only purpose for the token. You can create a new token with the same privileges if you need to configure the agent again.

#### Task 4: Configure the agent

1. Open a new Powershell window and navigate to the agent folder you created in the previous step.
2. To configure your agent, run the following command:

    ```powershell
    .\config.cmd
    ```

    > [!NOTE]
    > Optionally run the agent interactively by running .\run.cmd. You cannot close the command prompt window while running interactively.

3. Enter the following information when prompted to configure the agent:
    1. Enter the URL of the Azure DevOps organization: https://dev.azure.com/{your organization name}.
    2. Choose the authentication type: PAT.
    3. Enter the PAT token value you created in the previous step.
    4. Enter the agent pool name "eShopOnWebSelfPool" you created in the previous step.
    5. Enter the agent name "eShopOnWebSelfAgent".
    6. Choose the agent work folder (default is _work).
    7. Choose the agent run mode (Y to run as service).
    8. Enter Y to enable SERVICE_SID_TYPE_UNRESTRICTED for the agent service (Windows only).
    9. Enter the user account to use for the service.

        > [!IMPORTANT]
        > For running the agent service, refrain from using high-privileged accounts. Instead, employ a low-privileged account that holds the minimum permissions necessary for the operation of the service. This approach helps to maintain a secure and stable environment.

    10. Enter whether to prevent service starting immediately after configuration is finished (N to start the service).

        ![Screenshot showing the agent configuration.](media/agent-configuration.png)

    11. Check the agent status by navigating to the agent pool and clicking on the "Agents" tab. You should see the new agent in the list.

        ![Screenshot showing the agent status.](media/agent-status.png)

For more details on Windows agents, see: [Self-hosted Windows agents](https://learn.microsoft.com/azure/devops/pipelines/agents/windows-agent)

### Exercise 2: Create and configure a new security group for the agent pool

In this exercise, you will create a new security group for the agent pool.

#### Task 1: Create a new security group

1. Navigate to the Azure DevOps portal at https://dev.azure.com and open your organization.
2. Open the eShopOnWeb project, and click on "Project settings" from the left-side bottom menu.
3. Open Permissions under General.
4. Click on the "New Group" button.
5. Provide a name for the group, such as "eShopOnWeb Security Group".
6. Add the users you want to be part of the group.
7. Click on the "Create" button to create the group.

    ![Screenshot showing the security group creation.](media/create-security-group.png)

#### Task 2: Configure the security group

1. Open the new group and click on the "Settings" tab.
2. Deny unnecessary permissions for the group, such as "Rename team project", "Permanently delete work items", or any other permissions you don't want the group to have since it is used only for the agent pool.

    ![Screenshot showing the security group settings.](media/security-group-settings.png)

    > [!IMPORTANT]
    > If you leave permissions you don't want the group to have, scripts or tasks running on the agent can use the group permissions to perform actions you don't want them to perform.

### Exercise 3: Manage agent pool permissions

In this exercise, you will manage permissions for the agent pool.

1. Navigate to the Azure DevOps portal at https://dev.azure.com and open your organization.
2. Open the eShopOnWeb project, and click on "Project settings" from the left-side bottom menu.
3. Select Pipelines, and then select Agent pools.
4. Click on the "eShopOnWebSelfPool" agent pool.
5. In the agent pool details view, click on the Security tab.
6. Click the Add button and add the new group "eShopOnWeb Security Group" to the agent pool's user permissions.
7. You can choose to add users or group permissions to the specific project or the entire Azure DevOps organization. In this case, choose **Project**.
8. Choose the appropriate role for the user or group, such as Agent Pool Reader, User or Administrator. In this case, choose **User**.
9. Click Add to apply the permissions.

    ![Screenshot showing the agent pool security configuration.](media/agent-pool-security.png)

You are now ready to securely use the agent pool in your pipelines. For more details on agent pools, see: [Agent pools](https://learn.microsoft.com/azure/devops/pipelines/agents/pools-queues).

### Exercise 4: Remove the resources used in the lab

1. Stop and remove the agent service by running .config.cmd remove.
2. Delete the agent pool.
3. Delete the security group.
4. Revoke the PAT token.

## Review

In this lab, you learned how to configure Azure DevOps self-hosted agent and agent pools and manage permissions for those pools. By managing permissions effectively, you can ensure that the right users have access to the resources they need while maintaining the security and integrity of your DevOps processes.
