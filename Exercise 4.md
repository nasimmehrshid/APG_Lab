

## Task 1: Create Lake database:
The lake database in Azure Synapse Analytics enables customers to bring together database design, meta information about the data that is stored and a possibility to describe how and where the data should be stored. Lake database addresses the challenge of today's data lakes where it is hard to understand how data is structured. Azure Synapse Analytics provides industry specific database templates to help standardize data in the lake. These templates provide schemas for predefined business areas, enabling data to be loaded into a lake database in a structured way. The other two options are Custom tables and from data lake. In this exercise we are using Custom option.

1.![image](https://user-images.githubusercontent.com/31285245/188736679-cc6b97b4-778e-4f4c-8769-b54371cc2edb.png)

2.![image](https://user-images.githubusercontent.com/31285245/188736852-18b23942-4726-4e64-b617-54c358d19208.png)

  
3. ![image](https://user-images.githubusercontent.com/31285245/188737132-5de8b581-8ddf-4497-95c3-1b72e0f8f401.png)
   ![image](https://user-images.githubusercontent.com/31285245/188737450-f5c69a80-04ee-44fa-9805-e926cfedc172.png)
4.![image](https://user-images.githubusercontent.com/31285245/188737830-feab80a7-4d8f-4df0-a1dd-bc0f12eb401b.png)
 follow the same instruction and add the other columns. 
 ![image](https://user-images.githubusercontent.com/31285245/188739289-3ee064a6-3b94-486d-8b80-34e795a21b0c.png)
create 3 other tables : LegalEntityCustomer, Order, OrderLine. 

  Legal EntityCustomer: 
![image](https://user-images.githubusercontent.com/31285245/188739647-40271876-9957-45d8-a1a3-60e7261222b8.png)

Order: 
![image](https://user-images.githubusercontent.com/31285245/188739748-e362bc3c-55ee-4c4f-9e0e-7f5cdc5ab600.png)

OrderLine:
![image](https://user-images.githubusercontent.com/31285245/188739878-56605b26-0bc9-4c6f-a947-6525926cdde0.png)

After creating the tables, add the relationship between the tables: 

  Customer:
![image](https://user-images.githubusercontent.com/31285245/188743616-0e656bdc-80d5-47d6-b464-c7119eee9e75.png)

Legal EntityCustomer: 
![image](https://user-images.githubusercontent.com/31285245/188744019-0ccefa8d-b7b4-401c-ac60-fe272249e5cb.png)

Order:
![image](https://user-images.githubusercontent.com/31285245/188744162-44e0c417-5815-4fd6-81db-b082b71567b7.png)

OrderLine:
![image](https://user-images.githubusercontent.com/31285245/188744275-b039244d-1846-43a8-b931-e9aaf8b6ac1c.png)


 
5. Now that we have our Lake database schema ready we need to populate tables with data. There are 3 approaches to achieve this; Spark, Sql, Dataflows. feel free to choose the approach that fits you the best. for sake of this task we will investigate Dataflows. Navigate to Develope blade and click on + icon and create a new Data flow. add a new source by clicking on "Add Sources".  
**Source settings:** Rename the source to WWICustomer, we will be loading customer data from datalake raw zone into lake database customer table. Choose Inline for Source type and select Parquet as dataset type, choose Linke service to ADLSG2 and test connection. ![image](https://user-images.githubusercontent.com/40135849/174074353-8b77d17b-0f57-4c94-bf2a-34077b4c6d02.png)  
**Source options:** Navigate to Source options tab, choose File as File mode, browse and select Customer.parquet from datalake raw zone. 
![image](https://user-images.githubusercontent.com/31285245/188633394-19e9f965-9c9d-497b-a682-fbd94c17f704.png)
**Projection:** Import schema to show all columns in Customer.parquet file.  
**Data preview:** Make sure you can preview your data here.  
6. Add a Sink step to the data flow. click on + and select sink from the list.  
![image](https://user-images.githubusercontent.com/40135849/174098977-73939afd-c021-470e-a3a9-6173e12108cb.png)  
**Sink:** Rename Output stream name to Customer, keep incomin stream to WWICustomer, choose Workspace DB as Sink type. select Database you've created at step 1. Select Customer table.  
![image](https://user-images.githubusercontent.com/40135849/174099972-ec67f719-4c29-4439-b23a-3696f030807f.png)  
**Mapping:** Make sure you have Skip duplicate input and output columns tick for all steps below.  
![image](https://user-images.githubusercontent.com/40135849/174661898-2fd03b4e-4815-4ae5-8424-989bf9d6ed62.png)  
7. Un-check Auto mapping and manualy set below mappings. Make sure that you include only the below columns.  
![image](https://user-images.githubusercontent.com/40135849/174100629-ffc0bd83-5872-482b-a73b-b42b96f6828e.png)  
8. Create another Sink activity from WWICustomer output stream. Rename the sink to LegalEntityCustomer and select LegalEntityCustomer as Database Table. follow same step as above. Make mapping to the below columns:  
![image](https://user-images.githubusercontent.com/40135849/174661702-0a86c362-505f-43d6-8e57-4a4b5f894693.png)  
9. Repeat steps 5 and 6 to create a new source for WWIOrder and a Sink to Order table. Make below mapping:  
![image](https://user-images.githubusercontent.com/40135849/174102652-de37b284-c2c9-428a-83cd-2db5e6653091.png)  
10.Repeat steps 5 and 6 to create a new source for WWIOrderLine and a Sink to OrderLine table. Make below mapping:  
![image](https://user-images.githubusercontent.com/40135849/174103031-172200f2-1d95-4696-b710-a11b6d6440e0.png)  
11. Step 10 might rise a column type mismatch, add a "Derived Column" activity in between steps to cast types before sink.  
![image](https://user-images.githubusercontent.com/40135849/174663547-a38cc55e-d233-4944-9d4b-df482a7956d3.png)  
12. Final Data flow should look like below:   
![image](https://user-images.githubusercontent.com/31285245/187787039-d6af5541-2923-4226-943b-59cb61e7fd60.png)

13. At this step we want to create a pipeline and trigger Data flow that we made. Navigate to Integrate blade and click on + icon. Create a new pipeline. Select a Data flow activity.  
 ![image](https://user-images.githubusercontent.com/40135849/174104621-748a37dd-f35f-498e-9f7c-d7e15b7c2c18.png)    
14. In Setting tab choose the right dataflow. Add trigger and Trigger now.
15. Navigate to Monitor blade to follow pipeline run. make sure your pipeline runs successfully.  
![image](https://user-images.githubusercontent.com/40135849/174663010-0b0b4049-64f5-4a49-82a5-f73d83cb1916.png)





