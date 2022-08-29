# Load data from source systems
In this exercise we are going to integrate an Azure Synapse Analytics workspace with an Azure Purview workspace.
Synapse Analytics is the enterprise scale Data Analytics platform and Purview is the Data Governance platform of Azure.
After completing this exercise, you will be able to visualize a Data Lineage, that is coming from an Azure Synapse pipeline, using Azure Purview.

# Load CSV file
In this exercise we will load data(csv file) from Azure data lake storage (ADLS) to Synapse Dedicated SQL Pool.

  
![image](https://user-images.githubusercontent.com/31285245/182120969-370e38f5-e701-4842-9473-cf587cd4d938.png)

# Load data from source systems
Before being able to do any analysis over our different source systems, we need to copy data from different source systems to a single data lake. A data lake is often designed to be a central repository of unstructured, semi-structured and structured data. in this scenario we would like our analysis to be decoupled from our source systems, therefore we will copy raw  data to a raw zone of a data lake. 

## Task 1: Create a raw zone in your ADLSG2:
1. Navigate to [Azure portal](<https://ms.portal.azure.com/>), find you storage account and click on Containers.
2. Click on +Container to create a new container named rawdata.

![image](https://user-images.githubusercontent.com/31285245/182031785-b3a5d368-41b7-4123-841d-f66c5caa8ab5.png))

For this exercise we are using Germany CSV file.

To upload the excel file to my storage account, I am going to use the Azure Storage Explorer desktop application.

![image](https://user-images.githubusercontent.com/31285245/182033395-1157951d-bff5-4291-b68e-cafec4738112.png)

## Task 2: Create a link from Synapse to Purview:
Before we start working on the Synapse Analytics workspace, we will link the Synapse workspace to our Purview workspace.

In your Synapse workspace, open the Manage view, click on Azure Purview and Connect to Purview Accout. Here you will be able to browse existing Purview workspaces and select the right one.

![image](https://user-images.githubusercontent.com/31285245/182033831-8f64b3c8-5998-4555-b89a-4667cc687f68.png)

Once the connection succeeds, you should see a green check.

![image](https://user-images.githubusercontent.com/31285245/182033887-5f8f91f3-4aeb-475d-ad40-e73ed6b61e37.png)

![image](https://user-images.githubusercontent.com/31285245/182033959-47192ca0-2fb9-4feb-ba1c-9cfaecf5f5a7.png)

## Task 3: Create a Synapse pipeline with a Copy Activity:
In this task, we will create a Synapse Pipeline with a Copy data activity that will read the csv file from the data lake storage and copy it into Synapse Dedicated SQL Pool.
First we need to create Synapse Dedicated SQL Pool (if you don't have)
In your Synapse workspace, switch to the manage pane.Click on + button and give a name to your Dedicated SQL pool. and choose DW100c for your performance level.
![image](https://user-images.githubusercontent.com/31285245/182107067-aa39bb7d-9ab1-4ed1-91b2-02c15f6d151c.png)
![image](https://user-images.githubusercontent.com/31285245/182107233-72cda238-3260-42a4-a58a-2a64a4d671c6.png)

Switch to the Integrate pane. Click on + button and select pipeline. You can rename your pipeline to “Copy from Storage to Dedicated SQLPool” from the Properties pane on the right side.
Open the Activities, drag and drop Copy Data activity to the empty canvas. You can change the name of your copy data activity to “Copy from Storage to Dedicated SQLPool”.
![image](https://user-images.githubusercontent.com/31285245/182363498-f67852f8-4a78-4738-8b27-3353b9260155.png)
Choosing the Source Dataset: Click on “Source” and then “New” button to create an integration dataset. Select “Azure data lake Storage Gen2” from the list and then select “CSV” as the data format then set the properties.Since this table does not exist yet, there is no schema information we can import. For this reason, I will not import a schema. 
![image](https://user-images.githubusercontent.com/31285245/182385156-4432025c-64d7-4624-8b27-f279bc07752e.png)
![image](https://user-images.githubusercontent.com/31285245/182385462-69b8706b-acb7-49e5-967b-3141b9621646.png)
Now, you should be able to use the “Preview Data” button to view the contect of your csv file, that is stored in Adls gen2.
![image](https://user-images.githubusercontent.com/31285245/182385604-76258ebd-2582-4562-a10f-86f7c55e6804.png)
Choosing the Sink Dataset: Click on Sink and then New button to create a new integration dataset to the target Synapse Dediacted SQL Pool.
![image](https://user-images.githubusercontent.com/31285245/182365093-7aeeff65-f1f1-445a-86b7-f23cfc769d4c.png)
Enable Auto Create Table so that Copy Data activity can create the target table on the fly, while running the activity.
![image](https://user-images.githubusercontent.com/31285245/182386571-9273659f-9fde-4a01-a68b-dcc4bf9a3dfe.png)

## Task 4:Run the Copy data from Blob Storage to Synapse Dedicated Pool
Now that you secured all the changes, there are two options to run the pipeline:

Debugging is great if you want to test your pipelines. Please be aware that the activity results will be persistent, meaning that the data will be copied to the SQL DB for option1 and to the Synapse Dediacated Pool for Option 2. In this approach it won’t be rolled back when debug is finished.
Triggering is usually preferred on Development and Production pipelines since they can be scheduled. 

For simplicity, I will continue with debugging.
![image](https://user-images.githubusercontent.com/31285245/182035939-3bdb385e-4f25-46a4-8794-27930336ccd5.png)
When the debug finishes, there will be a green check mark.

![image](https://user-images.githubusercontent.com/31285245/182035998-04d56f58-1f89-4ff3-87b1-68b16723421f.png)


Intermediate Step (Optional): Viewing copied data
# Option1:
At this point, the copy activity created a target table in the Azure SQL DB and copied the data coming from the excel file that was in Blob Storage to the Azure SQL DB. You can confirm this by looking into the database.

If you are using the Azure Portal to query your Azure SQL DB, please make sure that you have added your client IP Address to the SQL Server Firewall Settings, otherwise you won’t be able to login to the DB.

![image](https://user-images.githubusercontent.com/31285245/182036378-b2239497-eab9-48f5-81ce-13bcb02a54a5.png)


# Option2:
To see the copied data in your Synapse dediacated pool, switch to the data pane. Click on the Workspace, under the SQL databse ypou should be able to find your chosen database (in my case it named APGDedacatedSQLPool), under the table you will be able to find your table and you can query your data. 

![image](https://user-images.githubusercontent.com/31285245/182391429-65e60a7e-7978-4634-9bdc-a1aa765df5c1.png)


## Task5: Visualize the Data Lineage in Purview
In this task, we are going to use the Data Lineage capability of Azure Purview to visualize the lineage of the raw dataset, in other words; we will see where the raw dataset initially was (Blog Storage) and where it ended up (SQL DB/ Synapse Dedicated SQL Pool).

Open your Purview workspace and click on Browse Assets button.

![image](https://user-images.githubusercontent.com/31285245/182036444-3f4f7ad5-e618-43b0-8231-c668c665fe22.png)
Here you will have the option to see your assets By collection or By source Type. Select By source type and you will see that Purview discovered 6 Assets
Be aware that these assets will be visible only after you run a pipeline from Synapse and it might take some time for Purview to show them.
![image](https://user-images.githubusercontent.com/31285245/182392609-f1527a4f-62d3-49de-b3ec-44b1a59d9df4.png)
Feel free to investigate all the source types, but for now I will select Azure Synapse Analytics. After selecting it, you will see that Purview captured the Synapse Pipeline which has the Copy activity.
![image](https://user-images.githubusercontent.com/31285245/182037200-f7b6e332-65f8-4702-a977-664205b37ac2.png)
![image](https://user-images.githubusercontent.com/31285245/182398884-b16f4935-c7cf-485b-ae4a-bcbbdce01619.png)

Click on the Copy activity name and you will be directed to the Asset overview.

Here is an Overview of the Copy Activity asset that Purview has discovered from the Synapse workspace.

![image](https://user-images.githubusercontent.com/31285245/182037107-e20a6020-c606-480a-b329-b708d12e56fc.png)

The most important view right now is the Lineage view, since it shows the lineage of the raw dataset: where it was and where it goes.

In the middle, you see that the raw dataset was used by a Copy activity and stored in SQL database. This information is coming from our Synapse workspace and it confirms that our Synapse + Purview integration was successful.

On the left side, you can also see the column information of the input dataset or you can select the output dataset.

![image](https://user-images.githubusercontent.com/31285245/182037040-63f5cb53-5c13-414c-b0e3-3cbe48f576e4.png)

Here is an Overview of the Copy Activity asset that Purview has discovered from the Synapse workspace
![image](https://user-images.githubusercontent.com/31285245/182400223-aaf1dc53-2eb2-44bf-ad50-60064a0eea33.png)
The most important view right now is the Lineage view, since it shows the lineage of the raw dataset: where it was and where it goes.
![image](https://user-images.githubusercontent.com/31285245/182400465-ff854682-9366-42e0-a28d-3a68af90fdbd.png)
In the middle, you see that the raw dataset was used by a Copy activity and stored in SQL database. This information is coming from our Synapse workspace and it confirms that our Synapse + Purview integration was successful.

On the left side, you can also see the column information of the input dataset or you can select the output dataset.
![image](https://user-images.githubusercontent.com/31285245/182398403-ec921dfe-b7b1-43bf-a6e3-cd128b730cd1.png)



# Load from SQL Database
This is the final architecture we are going to build:


![image](https://user-images.githubusercontent.com/31285245/182119988-d33b0209-d09e-48b7-815f-58591506bca5.png)


## Task 6: Create a raw zone in your ADLSG2:
1. Navigate to [Azure portal](<https://ms.portal.azure.com/>), find you storage account and click on Containers.
2. Click on +Container to create a new container named rawdata.

![image](https://user-images.githubusercontent.com/31285245/182031785-b3a5d368-41b7-4123-841d-f66c5caa8ab5.png))

## Task 7: Copy data from SQL to ADLS with metadatadriven copy task
![image](https://user-images.githubusercontent.com/31285245/182606143-fbf17e74-5a73-4d4a-99d0-0e426c08d56d.png)
![image](https://user-images.githubusercontent.com/31285245/182606374-f8fe424c-1609-43ed-b4e0-cfabac86cd16.png)
![image](https://user-images.githubusercontent.com/31285245/182606432-94085664-4667-4124-b015-690cd814de11.png)
![image](https://user-images.githubusercontent.com/31285245/182606791-25eed81a-0adc-4b9e-9789-3d5b9aa53fa7.png)
![image](https://user-images.githubusercontent.com/31285245/182606943-f309c8d3-76a5-427f-bb10-b5bd390017b5.png)
![image](https://user-images.githubusercontent.com/31285245/182607018-61da4fa1-f398-491d-9c30-8b9ba56b06cf.png)
![image](https://user-images.githubusercontent.com/31285245/182607259-bb40740f-d2c1-4c19-82dd-773f94a59356.png)
![image](https://user-images.githubusercontent.com/31285245/182608853-0c7c2f11-eb90-4aaf-b36f-9ab8eacfdea7.png)
![image](https://user-images.githubusercontent.com/31285245/182608933-de03055f-901f-403f-b59c-5a624fac43a8.png)
![image](https://user-images.githubusercontent.com/31285245/182609145-ae81982e-576a-46c7-b3a0-f6d90afd93f1.png)
![image](https://user-images.githubusercontent.com/31285245/182609617-83afb1e4-bfb8-4357-a3d8-6c03c02202fb.png)
![image](https://user-images.githubusercontent.com/31285245/182610089-39436b40-d1ee-4897-b2ae-caff7d6cc3d5.png)
![image](https://user-images.githubusercontent.com/31285245/182611825-9ca33d08-fabb-4fba-ac2c-b5059004314a.png)
![image](https://user-images.githubusercontent.com/31285245/182611968-4b949a86-fe7f-48c7-8ed1-9a296a598c2f.png)


From ADLS to Synapse Dedicated SQL Pool usinf metadata-driven copy task
![image](https://user-images.githubusercontent.com/31285245/182612464-9a0450f3-9863-4fc0-8be4-8a31eff1a1ac.png)
![image](https://user-images.githubusercontent.com/31285245/182612582-c851b301-29e8-431d-9593-958529415c1c.png)
![image](https://user-images.githubusercontent.com/31285245/182612966-d29a8993-6577-403f-a2c4-c332fdc1974e.png)
![image](https://user-images.githubusercontent.com/31285245/182613152-b994fad4-4bd0-48ba-8795-82dde52f408d.png)
![image](https://user-images.githubusercontent.com/31285245/182613214-8a1959e7-c214-456a-b978-2d06be3155ae.png)
!![image](https://user-images.githubusercontent.com/31285245/182613738-16310b83-e2ce-4eb3-918b-da48dfda66b7.png)
![image](https://user-images.githubusercontent.com/31285245/182613795-0a23b4d0-fd45-4d98-92d7-fecc2d5d63ce.png)
![image](https://user-images.githubusercontent.com/31285245/182613846-db17eff0-ad29-432a-9686-26a606a9cf82.png)
![image](https://user-images.githubusercontent.com/31285245/182614226-07312776-3717-48ae-8473-c33a98a9fb96.png)












## Task 7: Create copy pipeline:
At this step we need to create data pipelines to ingest data from WWI Sales database, these pipelines should keep a metadata to load at large scale.
  Data Source | User | Pass
  -------|-----|------
  apg-demo-sql-server.database.windows.net  |  ServerAdmin | ?????
1. Navigate to Integrate blade, click on the plus and choose "Copy Data Tool". 
 ![image](https://user-images.githubusercontent.com/31285245/182038588-840c687e-e217-40b7-9102-f833579e005e.png)

3. Choose Metadata-driven copy task and select a database to create and maintain a control table. Metadata-driven copy task make use of a control table to create parameterized pipelines.                                                                                                                      
![image](https://user-![image](https://user-images.githubusercontent.com/31285245/182038667-afed2196-45f5-43cd-b0d8-340fd068bd31.png)
5. Create a Link Service to control table server. Linked services are much like connection strings, which define the connection information needed for the service to connect to external resources.                                                                                                           
image image image?????

5. Follow the wizard to choose a source data store and select tables you would like to copy over. For this task make sure you have "Show views" checked. please copy over below tables with following loading behavior.     
   Schema | View Name | Load behaviour | Watermark column name | Watermak column value start
   -------|-----------|--------------|--------------|----------
   WWIHACK_SALES | Customers | Delta load | LastEditedWhen | 2013-01-01
   WWIHACK_SALES | CustomerCategories | Full load
   WWIHACK_SALES | CustomerTransactions | Full load
   WWIHACK_SALES | Orders | Delta load | LastEditedWhen | 2013-01-01
   WWIHACK_SALES | OrderLines | Full load
   WWIHACK_SALES | Invoices | Delta load | LastEditedWhen | 2013-01-01
   WWIHACK_SALES | InvoiceLines | Full load
   WWIHACK_SALES | SpecialDeals | Full load
 ![image](https://user-images.githubusercontent.com/31285245/182038832-483b748a-0081-4677-b1b3-f7368c8b36b9.png)


6.  Choose a target destination to land data. make sure to choose .parquet or .csv format as this will be a requirement  to use lakedatabase patterns in Exercise 2.    ![image](https://user-images.githubusercontent.com/31285245/182038928-0e12b950-b934-49fa-a991-fcf1ba97da50.png)
 ![image](https://user-images.githubusercontent.com/31285245/182038958-1c444507-5129-4857-81d3-6e4a77404e26.png)
 ![image](https://user-images.githubusercontent.com/31285245/182038976-fcd12842-1a9f-42c7-824f-d27090c03406.png)

7.  Review summary of the pipeline and deploy. At this stage Synapse is creating two SQL scripts based on the wizard's setup.                           
![image](https://user-images.githubusercontent.com/31285245/182039211-7a8407dc-8458-4ced-8a63-ad52b8f53245.png)

    - The first SQL script is used to create two control tables. The main control table stores the table list, file path or copy behaviors. The connection control table stores the connection value of your data store if you used parameterized linked service.
                                

8. Open SSMS or Azure Studio to connect to your control table server, and run the two SQL scripts to create control tables and store procedure.
9. Query the main control table and connection control table in SSMS or Azure Studio  to review pipelines metadata. Read more about Control Tables [here](<https://docs.microsoft.com/en-us/azure/data-factory/copy-data-tool-metadata-driven#control-tables>).                                            
 ![image](https://user-images.githubusercontent.com/31285245/182039308-e9ced0fa-b021-40c5-b727-bb0e24523e85.png)

10. Navigate back to Integrate blade, under Pipelines there is a new folder with three pipelines. Review the created pipelines, try to understand what are the activities in each pipeline and try to track the logic. Read more about the pipelines [here](<https://docs.microsoft.com/en-us/azure/data-factory/copy-data-tool-metadata-driven#pipelines>). 
![image](https://user-images.githubusercontent.com/31285245/182039407-93aa553a-e86a-4007-b22f-1ecc92352477.png)
11. Trigger the pipeline to move data to your "raw zone".  
![image](https://user-images.githubusercontent.com/31285245/182039430-f5ce69cf-d5b3-49bf-9f09-15468450e78a.png) 
## Task 8: Run your pipeline and monitor:
  Navigate to Monitor blade and monitor your pipelines run. make sure that all pipelines run successfully before moving to next step.
![image](https://user-images.githubusercontent.com/31285245/182039590-bf6534e5-ea2a-4f23-951d-30fc0b56e898.png)
![image](https://user-images.githubusercontent.com/31285245/182039612-c806234b-01df-4630-88a9-85f5ebe58b73.png)
Same setpes to see the lineage on purview
![image](https://user-images.githubusercontent.com/31285245/182039796-b99936da-73ee-484f-b20a-323e9cd0091c.png)


