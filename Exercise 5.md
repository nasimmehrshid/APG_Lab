# Design a modern data warehouse
A modern data warehouse lets you bring together all your data at any scale easily, and means you can get insights through analytical dashboards, operational reports, or advanced analytics for all your users.
In this exersice you will practice a small scale mapping of OLTP model to OLAP. By the end of this exercise you have a Synapsecdedicated sql pool populated with your star schema fact and dimension tables. 
## Task 1: Determine table category:
Review data that resides in your Lake database. Think of how to design data warehouse fact and dimension tables. Map Customer, LegalEntityCustomer, Order, OrderLine table to 1 fact and 1 dimention tabel.
You can follow mapping below:   
![image](https://user-images.githubusercontent.com/40135849/174661007-e129ab94-76b6-41b4-bc1e-865c549703cb.png)
To design a well performing distributed tables using dedicated SQL pool we need to follow best practices, see tables below for a short review. for more info on Synapse distributions see [here](<https://docs.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-distribute>). 

| Table Indexing | Recommended use |
|--------------|-------------|
| Clustered Columnstore | Recommended for tables with greater than 100 million rows, offers the highest data compression with best overall query performance. |
| Heap Tables | Smaller tables with less than 100 million rows, commonly used as a staging table prior to transformation. |
| Clustered Index | Large lookup tables (> 100 million rows) where querying will only result in a single row returned. |
| Clustered Index + non-clustered secondary index | Large tables (> 100 million rows) when single (or very few) records are being returned in queries. |

| Table Distribution/Partition Type | Recommended use |
|--------------------|-------------|
| Hash distribution | Tables that are larger than 2 GBs with infrequent insert/update/delete operations, works well for large fact tables in a star schema. |
| Round robin distribution | Default distribution, when little is known about the data or how it will be used. Use this distribution for staging tables. |
| Replicated tables | Smaller lookup tables, less than 1.5 GB in size. |


## Task 2: Create Dedicated SQL pool and star schema:
Now that we have our star schema conceptually designed, we need to create a dedicated sql pool and generate tables. 
1. Navigate to Manage blade and select SQL pools. if you have a Dedicated pool active use that, if it's paused resume and use. if there is no dedicated sql pool create one by clicking on +New.
![image](https://user-images.githubusercontent.com/40135849/174266273-4b0de2f3-f26b-415f-8778-61cce9211896.png)
Choose a name for your deducated SQL pool and put performance level to DQ100c. Review + create. Wait for successful deployment.
2. Navigate to Develope blade click on + and create a new SQL script. Connect to your dedicated SQL pool.
![image](https://user-images.githubusercontent.com/40135849/174281029-7e2f3299-41ff-4efe-866a-04a0380082f4.png)

4. Create a WWI Schema, CustomerDim and OrderFact tables. Run scripts below on your dedicated sql pool. These scrips create a new schema and CustomerDim and FactOrder.
``` sql
 CREATE SCHEMA WWI;
```
``` sql
 CREATE TABLE [WWI].[CustomerDim]
 (
   [CustomerId] [int] NOT NULL,
   [CustomerEstablishedDate] [datetime ]  NULL,
   [CustomerTypeId] [int]  NULL,
   [LedgerId] [int]  NULL,
   [LegalEntityName] [nvarchar](200)  NULL,
   [LegalEntityDateOfEstablishment] [datetime] NULL,
   [StateOfLegalEntityEstablishment] [nvarchar](150)  NULL
 )
 WITH
 (
   DISTRIBUTION = REPLICATE,
   CLUSTERED COLUMNSTORE INDEX
 );
```
``` sql
CREATE TABLE [WWI].[OrderFact]
  (
    [OrderId] [int]  NOT NULL,
    [OrderConfirmationNumber] [int]  NOT NULL,
    [OrderEnteredByEmployeeId] [smallint]  NOT NULL,
    [OrderReceivedTimestamp] [datetime]   NULL,
    [OrderEntryTimestamp] [datetime]  NULL,
    [OrderRequestedDeliveryDate] [datetime]  NULL,
    [CustomerId] [int]  NOT NULL,
    [OrderLineCount] [int]  NOT NULL,
    [OrderQuantity] [int]  NOT NULL,
    [OrderAmount] [money ]  NOT NULL
  )
  WITH
  (
    DISTRIBUTION = HASH ( [CustomerId] ),
    CLUSTERED COLUMNSTORE INDEX
  );

```
## Task 3: Populate data warehouse tables with Spark pool
Now we have our Datawarehouse schema and tables ready. We need to create a pipeline to populate fact and dimention tables. We can achieve this using Spark pool, Sql and Dataflow. Task 3 and Task 4 are resulting in same output. you can choose between either of them.
Within this task we will use Spark pool to read data from Lake database do some transformation and write to dedicated sql database.  

### CustomerDim  

1. Navigate to Data blade, under Lake database expand WWI. expand Tables and hover over Customer table actions, choose new notebook> Load to DataFrame. This will generate pyspark code to read data from lake database. Attach your spark pool to the notebook and run the cell. This might take sometime to warm up Spark cluster.
![image](https://user-images.githubusercontent.com/40135849/174665368-891736e6-7625-4302-9dee-d3d907b5ebcc.png)  
2. Replace the cell with below code to read data from Customer and LegalEntityCustomer, join data and cast types.
``` python
from pyspark.sql.functions import col
import pyspark.sql.types
from pyspark.sql.types import *

%%pyspark
# Reading data from Customer table, select columns that we will use.
CustomerDF = spark.sql("SELECT CustomerId, CustomerEstablishedDate, CustomerTypeId, LedgerId  FROM `WWI_Hack`.`Customer` ")
CustomerDF.show(10)

# Reading data from LegalEntityCustomer table, select columns that we will use.
LECustomerDF = spark.sql("SELECT CustomerId,LegalEntityName,LegalEntityDateOfEstablishment,StateOfLegalEntityEstablishment FROM `WWI_Hack`.`LegalEntityCustomer` ")
LECustomerDF.show(10)

# Joining customer data 
inner_join = CustomerDF.alias("a").join(LECustomerDF.alias("b"), CustomerDF.CustomerId == LECustomerDF.CustomerId).select("a.*","b.LegalEntityName","b.LegalEntityDateOfEstablishment","b.StateOfLegalEntityEstablishment")

# Check schema for mismatch with CustomerDim table
inner_join.printSchema()

# Casting TimestampType
inner_join=inner_join.withColumn("CustomerEstablishedDate",col("CustomerEstablishedDate").cast(TimestampType()))\
                     .withColumn("LegalEntityDateOfEstablishment",col("LegalEntityDateOfEstablishment").cast(TimestampType()))
                     
# Create a schema to match CustomerDim table
CustomerDimSchema =    [StructField('CustomerId',IntegerType(),False),\
                        StructField('CustomerEstablishedDate',TimestampType(),True),\
                        StructField('CustomerTypeId',IntegerType(),True),\
                        StructField('LedgerId',IntegerType(),True),\
                        StructField('LegalEntityName',StringType(),True),\
                        StructField('LegalEntityDateOfEstablishment',TimestampType(),True),\
                        StructField('StateOfLegalEntityEstablishment',StringType(),True)]
CustomerDim = sqlContext.createDataFrame(inner_join.rdd, StructType(CustomerDimSchema))

CustomerDim.show(10)
```
3.  Use below code to write the spark Dataframe from step 2 into CustomerDim sql table. Fill in values in < > . For more info read [here](<https://docs.microsoft.com/en-us/azure/synapse-analytics/spark/synapse-spark-sql-pool-import-export?tabs=python%2Cpython1%2Cpython2%2Cpython3%2Cpython4%2Cpython5#write-using-azure-ad-based-authentication>)
``` python
# Write using AAD Auth to internal table
# Add required imports
import com.microsoft.spark.sqlanalytics
from com.microsoft.spark.sqlanalytics.Constants import Constants

# Configure and submit the request to write to Synapse Dedicated SQL Pool
# Sample below is using AAD-based authentication approach; See further examples to leverage SQL Basic auth.
(CustomerDim.write
 # If `Constants.SERVER` is not provided, the `<database_name>` from the three-part table name argument
 # to `synapsesql` method is used to infer the Synapse Dedicated SQL End Point.
 .option(Constants.SERVER, "<sql-server-name>.sql.azuresynapse.net")
 # Like-wise, if `Constants.TEMP_FOLDER` is not provided, the connector will use the runtime staging directory config (see section on Configuration Options for details).
 .option(Constants.TEMP_FOLDER, "abfss://<container_name>@<storage_account_name>.dfs.core.windows.net/<some_base_path_for_temporary_staging_folders>")
 # Choose a save mode that is apt for your use case.
 # Options for save modes are "error" or "errorifexists" (default), "overwrite", "append", "ignore".
 # refer to https://spark.apache.org/docs/latest/sql-data-sources-load-save-functions.html#save-modes
 .mode("overwrite")
 # Required parameter - Three-part table name to which data will be written
 .synapsesql("<database_name>.<schema_name>.<table_name>"))
```
4. Navigate to Data blade, under SQL database expand dedicated sql pool. Select Top 100 rows on WWI.CustomerDim to make sure our data has been moved.
![image](https://user-images.githubusercontent.com/40135849/174674946-6a6e4d69-88bb-488a-b434-133663f0e981.png)

### OrderFact

1. Follow steps above for Order and OrderLine tables.
2. Use below code to populate PrderFact table.

``` python
from pyspark.sql.functions import col
from pyspark.sql import functions as f
from pyspark.sql.types import IntegerType, ShortType, TimestampType, StringType, DecimalType, StructType, StructField

%%pyspark
# Reading data from Order table.
OrderDF = spark.sql("SELECT * FROM `WWI_Hack`.`Order`")
#OrderDF.show(10)
# Reading data from OrderLine.
OrderLineDF =  spark.sql("SELECT * FROM `WWI_Hack`.`OrderLine` ")
#OrderLineDF.show(10)
OrderLineDF=OrderLineDF.withColumn('Order Amount',OrderLineDF['Quantity']*OrderLineDF['ProductSalesPriceAmount']) 
OrderLineDF.show(10)
OrderLineDF=OrderLineDF.groupBy('OrderId').agg(f.sum("Order Amount"),f.count("*"),f.sum("Quantity"))
# Joining Order data 
inner_join = OrderLineDF.alias("a").join(OrderDF.alias("b"), OrderDF.OrderId == OrderLineDF.OrderId).select("a.*","b.*")
inner_join=inner_join.select("a.OrderId","OrderConfirmationNumber","OrderEnteredByEmployeeId","OrderReceivedTimestamp","OrderEntryTimestamp","OrderRequestedDeliveryDate","CustomerId","count(1)","sum(Quantity)", "sum(Order Amount)")
inner_join=inner_join.withColumn("OrderConfirmationNumber",col("OrderConfirmationNumber").cast(IntegerType()))\
                     .withColumn("OrderRequestedDeliveryDate",col("OrderRequestedDeliveryDate").cast(TimestampType()))\
                     .withColumn("sum(Quantity)",col("sum(Quantity)").cast(IntegerType()))
                     
# Create a schema to match OrderFact table
OrderFactSchema =     [ StructField('OrderId',IntegerType(),False),
                        StructField('OrderConfirmationNumber',IntegerType(),False),
                        StructField('OrderEnteredByEmployeeId',ShortType(),False),
                        StructField('OrderReceivedTimestamp',TimestampType(),True),
                        StructField('OrderEntryTimestamp',TimestampType(),True), 
                        StructField('OrderRequestedDeliveryDate',TimestampType(),True), 
                        StructField('CustomerId',IntegerType(),False), 
                        StructField('OrderLineCount',IntegerType(),False), 
                        StructField('OrderQuantity',IntegerType(),False),
                        StructField('OrderAmount',DecimalType(19,4),False)]
OrderFact = sqlContext.createDataFrame(inner_join.rdd, StructType(OrderFactSchema))
OrderFact.show(10)
```
3. Use below code to write the spark Dataframe from step 2 into OrderFact sql table. Fill in values in < > . For more info read [here](<https://docs.microsoft.com/en-us/azure/synapse-analytics/spark/synapse-spark-sql-pool-import-export?tabs=python%2Cpython1%2Cpython2%2Cpython3%2Cpython4%2Cpython5#write-using-azure-ad-based-authentication>)
``` python
# Write using AAD Auth to internal table
# Add required imports
import com.microsoft.spark.sqlanalytics
from com.microsoft.spark.sqlanalytics.Constants import Constants

# Configure and submit the request to write to Synapse Dedicated SQL Pool
# Sample below is using AAD-based authentication approach; See further examples to leverage SQL Basic auth.
(OrderFact.write
 # If `Constants.SERVER` is not provided, the `<database_name>` from the three-part table name argument
 # to `synapsesql` method is used to infer the Synapse Dedicated SQL End Point.
 .option(Constants.SERVER, "<sql-server-name>.sql.azuresynapse.net")
 # Like-wise, if `Constants.TEMP_FOLDER` is not provided, the connector will use the runtime staging directory config (see section on Configuration Options for details).
 .option(Constants.TEMP_FOLDER, "abfss://<container_name>@<storage_account_name>.dfs.core.windows.net/<some_base_path_for_temporary_staging_folders>")
 # Choose a save mode that is apt for your use case.
 # Options for save modes are "error" or "errorifexists" (default), "overwrite", "append", "ignore".
 # refer to https://spark.apache.org/docs/latest/sql-data-sources-load-save-functions.html#save-modes
 .mode("overwrite")
 # Required parameter - Three-part table name to which data will be written
 .synapsesql("<database_name>.<schema_name>.<table_name>"))
```
4. Navigate to Data blade, under SQL database expand dedicated sql pool. Select Top 100 rows on WWI.OrderFact to make sure our data has been moved.

