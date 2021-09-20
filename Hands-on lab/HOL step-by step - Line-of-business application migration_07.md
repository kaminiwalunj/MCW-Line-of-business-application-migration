## Exercise 2: Migrate the Application Database

Duration: 60 minutes

In this exercise you will migrate the application database from the on-premises Hyper-V virtual machine to a new database hosted in the Azure SQL Database service. You will use the Azure Database Migration Service to complete the migration, which uses the Microsoft Data Migration Assistant for the database assessment and schema migration phases.

### Task 1: Create an Azure SQL Database

In this task you will create a new Azure SQL database to migrate the on-premises database to.

> **Note**: This lab focuses on simplicity to teach the participant the technical tools required. In the , more consideration should go into the long-term plan prior to creating the first DB. For instance: Will this DB live in an Azure landing zone? Who will operate this environment post-migration? What policies are in place that the migration team should be aware of prior to migration? These landing zone and operating model related topics are covered in the Cloud Adoption Framework’s Ready methodology. You don’t need to deviate from the script, but be familiar with the four-step process in that methodology, so you can field those types of a questions if they come up in the lab.

1. Open the Azure portal at https://portal.azure.com and log in using your subscription credentials if it's not still up.

2. Expand the portal's left navigation by selecting **Show portal menu** in the top left then select **+ Create a resource**, then select **Databases**, then select **SQL Database**.

   ![Azure portal screenshot showing the select path to create a SQL Database.](images/Exercise2/new-sql-db.png "New SQL Database")

3. The **Create SQL Database** blade opens, showing the **Basics** tab. Complete the form as follows:

   - Subscription: **Select your subscription**.
  
   - Resource group: (select existing) **SmartHotelDBRG**
  
   - Database name: **smarthoteldb**
  
   - Server: Select **Create new** and fill in the New server blade as follows then select **OK**:
  
        - Server name: **smarthoteldb<inject key="DeploymentID" enableCopy="false" />**
  
        - Server admin login: **demouser**
  
        - Password: **<inject key="SmartHotelHost Admin Password" />**
  
        - Location: **IMPORTANT: Select the same region as the resource group - this makes migration faster.**

   > **Note**: You can verify the location by navigating to `https://portal.azure.com` in the new browser tab, and then selecting the resource group from the left navigation menu.

   ![Screenshot from the Azure portal showing the New server blade (when creating a SQL database).](https://github.com/Shivashant25/MCW-Line-of-business-application-migration/blob/snapshot/Hands-on%20lab/images/Exercise1/e2%20t1%20s3.png?raw=true "Create Server for SQL Database")

    - Want to use SQL elastic pool?: **No**
  
    - Compute + storage: **Standard**

   > **Note**: To select the **Standard** database tier, select **Configure database**, then select **Standard** from the dropdown for **Service tier** and select **Apply**.

   ![Screenshot for selecting database tier.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/snapshot/Hands-on%20lab/images/Exercise2/Exe2Task1Step3-1.png?raw=true "selecting database tier")

   ![Screenshot for selecting database tier.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/snapshot/Hands-on%20lab/images/Exercise2/Exe2Task1Step3-2.png?raw=true "selecting database tier")

    The final screenshot will look like this:

   ![Screenshot from the Azure portal showing the Create SQL Database blade.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/snapshot/Hands-on%20lab/images/Exercise2/create-sql-db.png?raw=true "Create SQL Database")


4. Select **Next: Networking >** to move to the **Networking** tab. Confirm that **No access** is selected.

   > **Note**: We will configure private endpoints to access our database later in the lab.

5. Select **Review + Create**, then select **Create** to create the database. Wait for the deployment to complete.

#### Task summary 

In this task you created an Azure SQL Database running on an Azure SQL Database Server.

### Task 2: Create the Database Migration Service

In this task you will create an Azure Database Migration Service resource. This resource is managed by the Microsoft.DataMigration resource provider.

> **Note**: The Azure Database Migrate Service (DMS) requires network access to your on-premises database to retrieve the data to transfer. To achieve this access, the DMS is deployed into an Azure VNet. You are then responsible for connecting that VNet securely to your database, for example by using a Site-to-Site VPN or ExpressRoute connection.
>
> In this lab, the 'on-premises' environment is simulated by a Hyper-V host running in an Azure VM. This VM is deployed to the 'smarthotelvnet' VNet. The DMS will be deployed to a separate VNet called 'DMSVnet'. To simulate the on-premises connection, these two VNet have been peered.

1. In the Azure portal, expand the portal's left navigation and select **+ Create a resource**, search for **Azure Database Migration Service** and select it.

2. On the **Azure Database Migration Service** blade, select **Create**.

   ![Screenshot showing the DMS 'create' button.](images/Exercise2/dms-create-1.png "Create Azure Database Migration Service")

3. In the **Create Migration Service** blade, on the **Basics** tab, enter the following values:
   
   - Subscription: **Select your Azure subscription**.
  
   - Resource group: **AzureMigrateRG**
  
   - Service Name: **SmartHotelDBMigration**
  
   - Location: **Choose the same region as the Resource Group**.

   - Service mode: **Azure**
  
   - Pricing tier: **Standard: 1 vCore**

   ![Screenshot showing the Create DMS 'Basics' tab.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/snapshot/Hands-on%20lab/images/Exercise2/create-dms.png?raw=true "Create DMS - Basics")

4. Select **Next: Networking** to move to the **Networking** tab, and select the **DMSvnet/DMS** virtual network and subnet in the **SmartHotelHostRG** resource group.
   
   ![Screenshot showing the Create DMS 'Networking' tab.](images/Exercise2/create-dms-network.png "Create DMS - Networking")

5. Select **Review + create**, followed by **Create**.

> **Note**: Creating a new migration service can take around 20 minutes. You can continue to the next task without waiting for the operation to complete. You will not use the Database Migration Service until task 4.

#### Task summary 

In this task you created a new Azure Database Migration Service resource.

### Task 3: Assess the on-premises database using Data Migration Assistant

In this task you will use Microsoft Data Migration Assistant (DMA) to assess the on-premises database. DMA is integrated with Azure Migrate providing a single hub for assessment and migration tools.

1. Return to the **Azure Migrate** blade in the Azure portal. Select the **Overview** panel, then select **Assess and migrate databases**.

   ![Screenshot showing the Azure Migrate Overview blade in the Azure portal, with the 'Assess and migrate databases' button highlighted.](images/Exercise2/Exe2Task3Step1.png "Assess and migrate databases button")  

2. Under **Assessment tools**, click on **Click here** link to add a tool.

   ![Screenshot showing the 'Select assessment tool' step of the 'Add a tool' wizard in Azure Migrate, with the 'Azure Migrate: Database Assessment' tool selected.](images/Exercise2/Exe2Task3Step2.png "Add database assessment tool")
   
 > **Note**: Change the Project in the right hand top corner if you don't see the **Azure Migrate: Database Assessment** panel.

3. Select **Azure Migrate: Database Assessment**, then select **Add tool**
    
   ![](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/fix/Hands-on%20lab/images/local/addtool-2.png?raw=true)
   

4. Under **Migration tool**, click on **Click here** link to add a tool.

   ![Screenshot showing the 'Select assessment tool' step of the 'Add a tool' wizard in Azure Migrate, with the 'Azure Migrate: Database Migration' tool selected.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/fix/Hands-on%20lab/images/local/addtool-3.png?raw=true "Add database migration tool")
    
5. Select **Azure Migrate: Database Migration**, then select **Add tool**.
    
   ![](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/fix/Hands-on%20lab/images/local/addtool-4.png?raw=true)

6. Once the tools are installed in Azure Migrate, the portal should show the **Azure Migrate - SQL Server(only)** blade. 

7. From within **JumpVM**, open **Windows Explorer** and navigate to the **C:\\Program Files\\Microsoft Data Migration Assistant** folder. Open the **Dma.exe.config** file using Notepad. Search for **AzureMigrate** and remove the **\<\!-- and --\>** around the line setting the **EnableAssessmentUploadToAzureMigrate** key. **Save** the file and close Notepad when done.

   ![Screenshot showing the Dma.exe.config setting enabling upload to Azure Migrate.](images/Exercise2/dma-enable-upload.png "Dma.exe.config file")

8. Launch **Microsoft Data Migration Assistant** using the desktop icon .

   ![Screenshot for installing Data Migration Assistant.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/fix/Hands-on%20lab/images/local/dma5.png?raw=true "Data Migration Assistant")
  

9. In the Data Migration Assistant, select the **+ New** icon.  Fill in the project details as follows:

   - Project type: **Assessment**
  
   - Project name: **SmartHotelAssessment**
  
   - Assessment type: **Database Engine**
  
   - Source server type: **SQL Server**
  
   - Target server type: **Azure SQL Database**
     
10. Select **Create** to create the project.

    ![Screenshot showing the new DMA project creation dialog.](images/Exercise2/new-dma-assessment.png "New DMA assessment")

11. On the **Options** tab **select Next**.

12. On the **Select sources** page, in the **Connect to a server** dialog box, provide the connection details to the SQL Server, and then select **Connect**.

    - Server name: **192.168.0.6**
  
    - Authentication type: **SQL Server Authentication**
  
    - Username: **sa**
  
    - Password: **<inject key="SmartHotelHost Admin Password" />**
  
    - Encrypt connection: **Checked**
  
    - Trust server certificate: **Checked**

    ![Screenshot showing the DMA connect to a server dialog.](images/Exercise2/connect-to-a-server.png "Connect to server")

13. In the **Add sources** dialog box, select **SmartHotel.Registration**, then select **Add**.

    ![Screenshot of the DMA showing the 'Add sources' dialog.](images/Exercise2/add-sources.png "Add sources")

14. Select **Start Assessment** to start the assessment. 

    ![Screenshot of the DMA showing assessment in progress.](images/Exercise2/assessment-in-progress.png "Start assessment")

15. **Wait** for the assessment to complete, and review the results. The results should show two unsupported features, **Service Broker feature is not supported in Azure SQL Database** and **Azure SQL Database does not support EKM and Azure Key Vault integration**. For this migration, you can ignore these issues.

    > **Note**: For Azure SQL Database, the assessments identify feature parity issues and migration blocking issues.

    >- The SQL Server feature parity category provides a comprehensive set of recommendations, alternative approaches available in Azure, and mitigating steps to help you plan the effort into your migration projects.

    >- The Compatibility issues category identifies partially supported or unsupported features that reflect compatibility issues that might block migrating on-premises SQL Server database(s) to Azure SQL Database. Recommendations are also provided to help you address those issues.

16. Select **Upload to Azure Migrate** to upload the database assessment to your Azure Migrate project (this button may take a few seconds to become enabled).

    ![Screenshot of the DMA showing the assessment results and the 'Update to Azure Migrate' button.](images/Exercise2/db-upload-btn.png "Upload to Azure Migrate")

17. On the **Connect to Azure** blade, select **Azure** from the dropdown on the right then select **Connect**. Enter your Azure credentials when prompted. Select your **Subscription** and **Azure Migrate Project** using the dropdowns, then select **Upload**. Once the upload is complete, select **OK** to dismiss the notification.

    ![Screenshot of the DMA showing the assessment results upload panel.](images/Exercise2/db-upload.png "Upload to Azure Migrate")

18. Return to the **Azure Migrate - SQL Server(only)** blade in the Azure portal. Refreshing the page should now show the assessed database.

    ![Screenshot of the 'Azure Migrate - Databases' blade in the Azure portal, showing 1 assessed database.](images/Exercise2/Exe2Task3Step18.png "Azure Migrate - Database Assessment")

#### Task summary 

In this task you used Data Migration Assistant to assess an on-premises database for readiness to migrate to Azure SQL, and uploaded the assessment results to your Azure Migrate project. The DMA is integrated with Azure Migrate providing a single hub for assessment and migration tools.

### Task 4: Create a DMS migration project

In this task you will create a Migration Project within the Azure Database Migration Service (DMS). This project contains the connection details for both the source and target databases. In order to connect to the target database, you will also create a private endpoint allowing connectivity from the subnet used by the DMS.

In subsequent tasks, you will use this project to migrate both the database schema and the data itself from the on-premises SQL Server database to the Azure SQL Database.

We'll start by creating the private endpoint that allows the DMS to access the database server.

1. In the Azure portal, expand the portal's left navigation by selecting **Show portal menu** in the top left, then click on **Resource groups** and select the **SmartHotelDBRG** resource group, select the database server **smarthoteldb<inject key="DeploymentID" enableCopy="false" />**.
 
   ![Azure portal screenshot showing the select path to create a SQL Database.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/fix/Hands-on%20lab/images/local/db1.png?raw=true "New SQL Database")
   
   ![Azure portal screenshot showing the select path to create a SQL Database.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/fix/Hands-on%20lab/images/local/db2.png?raw=true "New SQL Database")

   ![Azure portal screenshot showing the select path to create a SQL Database.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/fix/Hands-on%20lab/images/local/db3.png?raw=true "New SQL Database")

2. From the left hand side menu, select **Private endpoint connections** under **Security**, then **+ Private endpoint**.

   ![Azure portal screenshot showing the select path to create a SQL Database.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/fix/Hands-on%20lab/images/local/dbendpoint.png?raw=true "New SQL Database")

3. On the **Basics** tab that appears, enter the following configuration then select **Next: Resource**. 

   - Resource group: **SmartHotelDBRG**
  
   - Name: **SmartHotel-DB-for-DMS**
  
   - Region: **Select the same region as the Resource Group**.
  
   ![Screenshot showing the 'Create a private endpoint' blade, 'Basics' tab.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/snapshot/Hands-on%20lab/images/Exercise2/create-dms-endpoint.png?raw=true "Private Endpoint - Basics")

4. On the **Resource** tab, entering the following configuration then select **Next: Configuration**. 

   - Connection method: **Connect to an Azure resource in my directory**.
  
   - Subscription: **Select your subscription**.
  
   - Resource type: **Microsoft.Sql/servers**
  
   - Resource: Select SQL database server **smarthoteldb<inject key="DeploymentID" enableCopy="false" />** from the dropdown which you created previously.
  
   - Target sub-resource: **sqlServer**

   ![Screenshot showing the 'Create a private endpoint' blade, 'Resource' tab.](images/Exercise2/private-endpoint-2.png "Private Endpoint - Resource")
   
5. On the **Configuration** tab enter the following configuration then select **Review + create**, then **Create**.

   - Virtual network: **DMSvnet**
  
   - Subnet: **DMS (10.1.0.0/24)**
  
   - Integrate with private DNS zone: **Yes**
  
   - Private DNS zones: (default) **privatelink.database.windows.net**

   ![Screenshot showing the 'Create a private endpoint' blade, 'Configuration' tab.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/fix/Hands-on%20lab/images/local/private-endpoint.png?raw=true "Private Endpoint - Configuration")

6. **Wait** for the deployment to complete. Once the deployment is done, click on  **Go to resource**.

   ![Screenshot showing the DNS entry for the SQL database server private endpoint](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/fix/Hands-on%20lab/images/local/dbendpoint1.png?raw=true "Private Endpoint")


7. On the **SmartHotel-DB-for-DMS** private endpoint blade, from the left hand side menu select **DNS configuration** which is under **Settings**.

   ![Screenshot showing step 1 to find the DNS entry for the SQL database server private endpoint](images/Exercise2/private-endpoint-dns1.png "Find DNS for Private Endpoint")

8. On the **SmartHotel-DB-for-DMS | DNS configuration**, select the **Private DNS Zone** **privatelink.database.windows.net**.

   ![Screenshot showing step 2 to find the DNS entry for the SQL database server private endpoint](images/Exercise2/private-endpoint-dns2.png "Private DNS integration")

9. On the **privatelink.database.windows.net** overview blade, note that the **Record set** with name **smarthoteldb<inject key="DeploymentID" enableCopy="false" />** and an internal IP address **10.1.0.5** is listed.

   ![Screenshot showing step 3 to find the DNS entry for the SQL database server private endpoint](images/Exercise2/private-endpoint-dns3.png "Private Endpoint IP address")

   >**Note**: Private DNS is used so that the database domain name, **\<your server\>.database.windows.net** resolves to the internal private endpoint IP address **10.1.0.5** when resolved from the DMSvnet, but resolves to the Internet-facing IP address of the database server when resolved from outside the DMSvnet. This means the same connection string (which contains the domain name) can be used in both cases.

10. Return to the Database server blade. Under **Security**, select **Firewalls and virtual networks**. Set 'Deny public network access' to **Yes**, then **Save** your changes.

    ![Screenshot showing the link to add an existing virtual network to the SQL database network security settings.](images/Exercise2/db-network.png "Database Server - Firewalls and virtual networks")
    
    In the Azure portal, navigate to the **SmartHotelDBRG** resource group, and then to the database server **smarthoteldb<inject key="DeploymentID" enableCopy="false" />**.   From the Oveview page, copy the server name of the database and keep this in a text editor as we will be using this further.

11. Check that the Database Migration Service resource you created in task 3 has completed provisioning. You can check the deployment status from the **Deployments** pane in the **AzureMigrateRG** resource group blade.

    ![Screenshot showing the AzureMigrateRG - Deployments blade in the Azure portal. The Microsoft.AzureDMS deployment shows status 'Successful'.](images/Exercise2/dms-deploy.png "DMS deployment complete")

12. Navigate to the Database Migration Service **SmartHotelDBMigration** resource blade in the **AzureMigrateRG** resource group and select **+ New Migration Project**.

    ![Screenshot showing the Database Migration Service blade in the Azure portal, with the 'New Migration Project' button highlighted.](images/Exercise2/new-dms-project.png "New DMS migration project")
 
13. In the **New migration project** blade, enter the below information,

    - Project name : Enter DBMigrate 
    - Leave the source server type as SQL Server and target server type as Azure SQL Database.
    - Choose type of activity : Select Create project only from the dropdown
    - Select **Create**.
 
    ![Screenshot showing the Database Migration Service blade in the Azure portal, with the 'New Migration Project' button highlighted.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/fix/Hands-on%20lab/images/local/Migration-1.png?raw=true "DMS migration project - settings")

14. The Migration Wizard opens, showing the **Select source** step. Complete the settings as follows, then select **Next: Select databases**.

    - Source SQL Server instance name: **10.0.0.4**
  
    - Authentication type: **SQL Authentication**
  
    - User Name: **sa**
  
    - Password: **<inject key="SmartHotelHost Admin Password" />**

    - Encryption connection: **Checked**
  
    - Trust server certificate: **Checked**

    ![Screenshot showing the 'Select source' step of the DMS Migration Wizard.](images/Exercise2/select-source.png "DMS project - Select source")

    > **Note**: The DMS service connects to the Hyper-V host, which has been pre-configured with a NAT rule to forward incoming SQL requests (TCP port 1433) to the SQL Server VM. In a real-world migration, the SQL Server VM would most likely have its own IP address on the internal network, via an external Hyper-V switch.
    > The Hyper-V host is accessed via its private IP address (10.0.0.4). The DMS service accesses this IP address over the peering connection between the DMS VNet and the SmartHotelHost VNet. This simulates a VPN or ExpressRoute connection between a DMS VNet and an on-premises network.

15. In the **Select databases** step, the **Smarthotel.Registration** database should already be selected. Select **Next: Select target**.

    ![Screenshot showing the 'Select databases' step of the DMS Migration Wizard.](images/Exercise2/select-databases.png "DMS project - Select databases")

16. Complete the **Select target** step as follows, then select **Next: Summary**:

    - Target server name: **Paste the server name value you copied earlier, {something}.database.windows.net**.
  
    - Authentication type: **SQL Authentication**
  
    - User Name: **demouser**
  
    - Password: **<inject key="SmartHotelHost Admin Password" />**
  
    - Encrypt connection: **Checked**

    ![Screenshot showing the DMS migration target settings.](images/Exercise2/select-target.png "DMS project - select target")

    > **Note**: You can find the target server name in the Azure portal by browsing to your database.

    ![Screenshot showing the Azure SQL Database server name.](images/Exercise2/sql-db-name.png "SQL database server name")

17. At the **Project summary** step, review the settings and select **Save project** to create the migration project.

    ![Screenshot showing the DMS project summary.](images/Exercise2/project-summary.png "DMS project - summary")

#### Task summary 

In this task you created a Migration Project within the Azure Database Migration Service. This project contains the connection details for both the source and target databases. A private endpoint was used to avoid exposing the database on a public IP address.

### Task 5: Migrate the database schema

In this task you will use the Azure Database Migration Service to migrate the database schema to Azure SQL Database. This step is a prerequisite to migrating the data itself.

The schema migration will be carried out using a schema migration activity within the migration project created in task 5.

1. Following task 5, the Azure portal should show a blade for the DBMigrate DMS project. Select **+ New Activity** and select **Schema only migration** from the drop-down.

   ![Screenshot showing the 'New Activity' button within an Azure Database Migration Service project, with 'Schema only migration' selected from the drop-down.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/fix/Hands-on%20lab/images/local/Migration2.png?raw=true "New Activity")

2. The Migration Wizard is shown. Most settings are already populated from the existing migration project. At the **Select source** step, re-enter the source database password **<inject key="SmartHotelHost Admin Password" />**, then select **Next: Select target**.

   ![Screenshot showing the 'Select source' step of the DMS Migration Wizard. The source database password is highlighted.](images/Exercise2/select-source-pwd-only.png "Select source")

3. At the **Select target** step, enter the password **<inject key="SmartHotelHost Admin Password" />** and select **Next: Select database and schema**.

   ![Screenshot showing the 'Select target' step of the DMS Migration Wizard. The target database password is highlighted.](images/Exercise2/select-target-pwd-only.png "Select target")

4. At the **Select database and schema** step, select the below information,
   - Check that the **SmartHotel.Registration** database is selected.
   - **Target Database**: Select **smarthoteldb**.
   - **Schema Source**: Select **Generate from source**. 
   - Select **Next: Summary**.
 
   ![Screenshot showing the 'Select database and schema' step of the DMS Migration Wizard.](images/Exercise2/select-database-and-schema.png "Select database and schema")

5. At the **Summary** step, enter **SchemaMigration** as the **Activity name**. Select **Start migration** to start the schema migration process.

   ![Screenshot showing the 'Summary' step of the DMS Migration Wizard. The activity name, validation option, and 'Run migration' button are highlighted](images/Exercise2/run-schema-migration.png "Schema migration summary")

6. The schema migration will begin. Select the **Refresh** button and watch the migration progress, until it shows as **Completed**.

   ![Screenshot showing the SchemaMigration progress blade. The status is 'Completed'.](images/Exercise2/schema-completed.png "Schema migration completed")

#### Task summary 

In this task you used a schema migration activity in the Azure Database Migration Service to migrate the database schema from the on-premises SQL Server database to the Azure SQL database.

### Task 6: Migrate the on-premises data

In this task you will use the Azure Database Migration Service to migrate the database data to Azure SQL Database.

The schema migration will be carried out using an offline data migration activity within the migration project created in task 5.

1. Return to the Azure portal blade for your **DBMigrate** migration project in DMS. Select **+ New Activity** and select **Data migration** from the drop-down.

   ![Screenshot showing the 'New Activity' button within an Azure Database Migration Service project, with 'Offline data migration' selected from the drop-down.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/fix/Hands-on%20lab/images/local/Migration2.1.png?raw=true "New Activity - Offline data migration")

2. The Migration Wizard is shown. Most settings are already populated from the existing migration project. At the **Select source** step, re-enter the source database password **<inject key="SmartHotelHost Admin Password" />**, then select **Next: *Select databases***.

   ![Screenshot showing the 'Select source' step of the DMS Migration Wizard. The source database password is highlighted.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/fix/Hands-on%20lab/images/local/Migration-3.png?raw=true "Select source")

3. At the **Select databases** blade, select **SmartHotel.Registration** database then select **Next: Select target**.

   ![](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/fix/Hands-on%20lab/images/local/Migration-4.png?raw=true)
    
4. At the **Select target** step, enter the password **<inject key="SmartHotelHost Admin Password" />** and select **Next: Map to target databases**.

   ![Screenshot showing the 'Select target' step of the DMS Migration Wizard. The target database password is highlighted.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/fix/Hands-on%20lab/images/local/Migration-5.png?raw=true "Select target")

5. At the **Map to target databases** step, check the **SmartHotel.Registration** database. Under **Target Database** select **smarthoteldb** from the dropdown and click on **Next: Configure migration settings**.

   ![Screenshot showing the 'Map to target databases' step of the DMS Migration Wizard.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/fix/Hands-on%20lab/images/local/Migration-6.png?raw=true "Map to target databases")

6. On the **Configure migration settings** blade, **expand** the **SmartHotel.Registration 2 of 2** option and select the **Bookings** table (Make sure the **MigrationHistory** table is not checked). Then, click on **Next: Summary**.

   > Note: The Configure migration settings step allows you to specify which tables should have their data migrated.
  
   ![Screenshot from DMS showing tables being selected for replication.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/fix/Hands-on%20lab/images/local/Migration-7.png?raw=true "Configure migration settings - select tables")

7. At the **Migration summary** step, enter **DataMigration** as the **Activity name**. Select **Start migration**.

   ![Screenshot from DMS showing a summary of the migration settings.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/fix/Hands-on%20lab/images/local/Migration-8.png?raw=true "Start migration")

8. The data migration will begin. Select the **Refresh** button and watch the migration progress, until it shows as **Completed**.

   ![Screenshot from DMS showing the data migration in completed.](images/Exercise2/data-migration-completed.png "Data migration completed")

As a final step, we will remove the private endpoint that allows the DMS service access to the database, since this access is no longer required.

9.  In the Azure portal, expand the portal's left navigation by selecting **Show portal menu** in the top left.

    ![Azure portal screenshot showing the select path to create a SQL Database.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/fix/Hands-on%20lab/images/local/db1.png?raw=true "New SQL Database")
   
    - select **Resource groups**. Select **SmartHotelDBRG** resource group.
   
    ![Azure portal screenshot showing the select path to create a SQL Database.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/fix/Hands-on%20lab/images/local/db2.png?raw=true "New SQL Database")
   
    - Select the database server **smarthoteldb<inject key="DeploymentID" />**.

    ![Azure portal screenshot showing the select path to create a SQL Database.](https://github.com/CloudLabs-MCW/MCW-Line-of-business-application-migration/blob/fix/Hands-on%20lab/images/local/db3.png?raw=true "New SQL Database")
   
   - Under **Security**, select **Private endpoint connections**.
   
   ![Azure portal screenshot showing the select path to create a SQL Database.](https://github.com/Shivashant25/MCW-Line-of-business-application-migration/blob/snapshot/Hands-on%20lab/images/Exercise1/e2%20t6%20ss9.png?raw=true "New SQL Database")
   
10.  Select the **SmartHotel-DB-for-DMS** endpoint added earlier, and select **Remove**, followed by **Yes**.

     ![Screenshot from the SQL server showing the SmartHotel-DB-for-DMS private endpoint being removed.](images/Exercise2/private-endpoint-remove.png "Remove private endpoint")

#### Task summary 

In this task you used an off-line data migration activity in the Azure Database Migration Service to migrate the database data from the on-premises SQL Server database to the Azure SQL database.

#### Exercise summary 

In this exercise you migrated the application database from on-premises to Azure SQL Database. The Microsoft Data Migration Assistant was used for migration assessment, and the Azure Database Migration Service was used for schema migration and data migration.

