# create Azure Synapse Lake Database with Data Flows
In this exercise we are going to create a Synapse Lake Database with Synapse Data Flows.
![image](https://user-images.githubusercontent.com/31285245/182042219-db1f5f2a-5987-4714-b586-1c93ff04ac4a.png)
## Task 1: Copy raw data to Azure Data Lake Storage:
Folw exercise 1
Using the Azure Storage Explorer tool,  create a new folder, call “FinancialSample”, and copy two file (France and Germany) there.

## Task 2: Create a Lake Database with a Dimensional Model:
In this task, we are going to create a Lake Database with a Dimensional Model. One nice thing about Synapse is that it has a built-in Database Designer that will help me to create this Lake Database. Our dimensional model will have 5 Dimension tables and 1 Fact table.

As a starting point, open your Synape workspace, switch to the Data page and click on small + sign to create a Lake Database. Renamed your Lake DB to FinanceDB.
![image](https://user-images.githubusercontent.com/31285245/182043536-cedd9b30-4352-4311-8075-7b82a359400b.png)
![image](https://user-images.githubusercontent.com/31285245/182043585-a0add207-09f6-475a-991d-0b9d6648760a.png)
Click on + Table button above to create a Custom Table.

There is also another option called ‘from template’, which gives you more than 5000 templates from 15 industries to choose! These templates are called Synapse Database Templates and you can read more about them [here](<https://docs.microsoft.com/en-us/azure/synapse-analytics/database-designer/overview-database-templates>).
![image](https://user-images.githubusercontent.com/31285245/182043629-2f084b65-af40-4fcd-86a4-b8c8db02d766.png)
Customize this table and turn it into a fact table.
![image](https://user-images.githubusercontent.com/31285245/182044010-259ecbf0-ea00-4959-8fe5-094a0d58c93d.png)
Create 5 more custom tables, each will represent a dimension table.
![image](https://user-images.githubusercontent.com/31285245/182044273-2f4e541b-8c06-4fb5-83c0-f6c9ee98890b.png)
Connect them by creating relations from the Fact (Sale) table to the Dimension tables.
![image](https://user-images.githubusercontent.com/31285245/182044485-ca52c663-efd0-419e-a40c-c8eb7a2cccdb.png)
There you have a logical dimensional model. Of course there is no physical model yet; in other words, data has not been stored physically in the Lake DB, which we will do in the next section. Before you continue, make sure to Publish/Commit your changes.

## Task 3: Create a Data Flow Pipeline:
In your Synapse workspace, switch to the Develop page, click on + button and select Data flow. This will give you an empty Data Flow canvas. Change the name of the Data Flow to ‘StoreDim’ to imply that it will push the data to the Store Dimension table. In the Paramets tab, add a new parameter called ‘new_file_name’, which will be used to select the file to be processed. Add a default value ‘France.xlsx’ so that you can start building our Data Flow processing this file. Also, turn on the Data flow debug.
![image](https://user-images.githubusercontent.com/31285245/182044858-09a86a4d-4ba9-45f8-9588-ac92563937ac.png)
![image](https://user-images.githubusercontent.com/31285245/182044875-d1f36e01-da70-4344-baa9-3bb8fe4ae84c.png)

Add a data source to the canvas to read the raw ‘France’ dataset, whose source type is ‘Inline’ and dataset type is DelimitedText. Rename the source data to ‘RawData’. Create a Linked Service to your storage account to read the data, using the Managed Identity or the Authentication method of your choice. If you want to use the Managed Identity to acces your storage account (which I think is the easiest option), you need to assign [Storage Blob Storage Data Contributer role to your Synapse Workspace ](<https://docs.microsoft.com/en-us/azure/synapse-analytics/security/how-to-grant-workspace-managed-identity-permissions#grant-permissions-to-managed-identity-after-workspace-creation>).
![image](https://user-images.githubusercontent.com/31285245/182045780-e3588299-1991-4660-9f2a-69b217b3538c.png)




![image](https://user-images.githubusercontent.com/31285245/182045112-316f3823-ba5e-447b-b0ba-c120194aef54.png)
Open the Source Options pane and fill in the file path:
container name / folder path / file name.
![image](https://user-images.githubusercontent.com/31285245/182045801-6b62ea3f-8b21-4c8e-a5ae-322444dde672.png)
Open the Projection Option pane click on ‘Import Schema’. Select the Data format ‘dd/MM/YYYY’, Numerical Whole Number ‘integer’, and the Numerical Faction ‘float’.
![image](https://user-images.githubusercontent.com/31285245/182045557-fde9a738-24d4-49ab-87d0-cd1756d8be44.png)
![image](https://user-images.githubusercontent.com/31285245/182045743-88519643-4ea4-41dd-9137-c4b9d5276c97.png)
If you have done everything correct (and have your Debug cluster running) you should see the data in the Data Preview pane.
![image](https://user-images.githubusercontent.com/31285245/182046198-8ed845d0-346d-472f-b602-fc4b07bf2afc.png)
Add an aggregate operation to your Data Flow by clicking the little + button on the lower right corner of our Source Data (RawData). Group by the Store column.
![image](https://user-images.githubusercontent.com/31285245/182046225-1f8242cf-cde9-4150-88ec-7e7e7b01bf06.png)
![image](https://user-images.githubusercontent.com/31285245/182046266-e7032ea5-a985-4a1b-a20a-2830f598aecd.png)
Switch to the Aggregates, aggregate by the first value, and asssign it to a new column ‘store_name’.
![image](https://user-images.githubusercontent.com/31285245/182046294-e249cba6-d020-418b-bcb3-afda6dafab4e.png)
Switch to the Data preview, you should see two columns as a result of the aggregate operation.
![image](https://user-images.githubusercontent.com/31285245/182046316-ae4d45ab-c9b4-4681-8bb7-a0ca34e5dcb2.png)
Add a Surrogate Key schema modifier to your Data Flow after the Aggregate operation you just added. Set the column name to ‘store_id’ and give it a start value of ‘1’.
![image](https://user-images.githubusercontent.com/31285245/182046339-f10173cc-845c-4b44-8b2b-3fe0016395fc.png)
![image](https://user-images.githubusercontent.com/31285245/182046404-ff5e734a-7df9-4800-bb3e-497b29d2ba24.png)
You should see the following result in the Data preview.
![image](https://user-images.githubusercontent.com/31285245/182046390-392273a5-8cff-48a6-b90d-8733aff95b13.png)
Add a Select operation to your Data Flow after the Surrogate Key schema modifier. Disable Auto mapping and then remove the Store colum.
![image](https://user-images.githubusercontent.com/31285245/182046427-689d02e8-6433-4054-94f1-8688ee352bdd.png)
![image](https://user-images.githubusercontent.com/31285245/182046469-d10fbfc6-bddb-4b73-8629-aedff05853ee.png)
You should see the following result in the Data preview.
![image](https://user-images.githubusercontent.com/31285245/182046516-fee57c54-c3b2-4147-a5a4-5f02d69a2a3f.png)
Add a Sink operator to write the data to Store Dimension table. Choose the Workspace DB as the Sink typ. Select the Lake DB you have created ’FinanceDB’ and select the ‘Store’ table.
![image](https://user-images.githubusercontent.com/31285245/182046567-7b7a9735-4f17-44a9-8644-fd52a85bb083.png)
![image](https://user-images.githubusercontent.com/31285245/182046608-352b8812-9acd-4754-b362-c14f52261531.png)
Switch to the Mapping pane, disable Auto mapping and check that the input columns (coming from our select operator) match the output columns (target table columns).
![image](https://user-images.githubusercontent.com/31285245/182046656-450b5354-168f-4373-8b37-ec2056d2c810.png)
You should see the following result in the Data preview.
![image](https://user-images.githubusercontent.com/31285245/182046710-f74ef4ef-2e61-4688-878a-73d0d9ffb8ce.png)
Our Data Flow is ready to write data to the ‘Store’ table of our Lake DB. Clone this Data Flow and create a similar Data Flow for each remaining Dimension table: Country, Date, Product, and Segment.













