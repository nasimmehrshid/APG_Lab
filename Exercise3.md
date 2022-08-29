# Unpivot data in Excel file and ingest it into ADLS(Parquet) and Synapse SQL Pool
In this exercise you will upload data from Excel file into the Parquet file on ADLS and a table in Synapse SQL Pool.
The excel file is available on your Data Lake under labs/unpivot/FruitConsumption.xlsx . We will tranform the data and change columns into rows (unpivot transformation).

Excel file format:

![image](https://user-images.githubusercontent.com/110227904/186141913-f5da5602-2fdf-4218-8b0c-e8315d7986e0.png)

After transformation and loading in SQL Pool data will look like:

![image](https://user-images.githubusercontent.com/110227904/186145321-bd6b4715-7dd7-4ae8-bd4c-52908c94e4ff.png)

# Help graphic to understand UNPIVOT Data Flow task

![image](https://user-images.githubusercontent.com/110227904/186380581-a526b26e-b732-498c-b45b-d0eb27e66969.png)



# To achieve the goal we need to follow the steps:

Create the SQL table to load tranformed data

![image](https://user-images.githubusercontent.com/110227904/186150428-0d577f0e-aabe-4d41-83b9-de171ef5e20a.png)

Copy the below code and paste to your empty script: (replace the name with your name)


```
CREATE TABLE [dbo].[UnpivotTable_yourname]
( 
	[Department] [nvarchar](100)  NULL,
	[Fruit] [nvarchar](50)  NULL,
	[FruitCount] [int]  NULL
)
WITH
(
	DISTRIBUTION = ROUND_ROBIN,
	HEAP
)
GO
```



Execute the script:

![image](https://user-images.githubusercontent.com/110227904/186150855-71ab2781-3461-48cd-b6b0-a7d276cdd9e8.png)



Create new Data Flow

![image](https://user-images.githubusercontent.com/110227904/186146118-9243b66b-ed09-4f4a-8e49-49799ca7ed30.png)

Rename it to Unpivot_[your name]

![image](https://user-images.githubusercontent.com/110227904/186146588-67bcf79f-ab8d-4948-aa13-af7e0b8f47b1.png)

Turn on Debug Data Flow (choose default options)

![image](https://user-images.githubusercontent.com/110227904/186146828-58cdd6f7-7347-4c42-baff-383cc65c2ea4.png)

Follow steps on screenshots:

![image](https://user-images.githubusercontent.com/110227904/186147009-281422e9-a961-449a-97dd-53c7e5dcedf2.png)

![image](https://user-images.githubusercontent.com/110227904/186147146-8abe0c5e-fbdf-49af-9e77-af470cc934f5.png)

![image](https://user-images.githubusercontent.com/110227904/186147257-70e78f48-b29d-4a05-b3a3-51ee64de25e4.png)

![image](https://user-images.githubusercontent.com/110227904/186147413-9511ec01-b148-450b-867c-e82d54667e25.png)

![image](https://user-images.githubusercontent.com/110227904/186147952-265daeb8-aa33-418a-b424-9ce56f4c7324.png)


**Validate that the data can be properly read from Excel file:**


![image](https://user-images.githubusercontent.com/110227904/186148254-52da6b37-0edb-4768-a2bf-75172d752efc.png)

![image](https://user-images.githubusercontent.com/110227904/186148543-cb683291-1679-4299-9ddb-3ec1367791a8.png)

![image](https://user-images.githubusercontent.com/110227904/186148686-59216e60-770d-4e04-b01c-f10353921379.png)

![image](https://user-images.githubusercontent.com/110227904/186149191-f221596f-68bf-46c0-b5bb-760d31628265.png)

![image](https://user-images.githubusercontent.com/110227904/186149358-d717df93-b4a7-4b6c-9de7-c14237dbeb58.png)

![image](https://user-images.githubusercontent.com/110227904/186149476-9b838ca8-7439-4c11-8bb1-b2a75984bd77.png)

![image](https://user-images.githubusercontent.com/110227904/186149598-21c789a7-5fe3-4d02-93ae-c24fc0477540.png)

![image](https://user-images.githubusercontent.com/110227904/186149719-1d686874-b3f4-4660-bd70-76d35e29b3cd.png)

![image](https://user-images.githubusercontent.com/110227904/186151433-991e3a5f-2860-4c81-9d86-5fe6eec56a65.png)

![image](https://user-images.githubusercontent.com/110227904/186151712-755defeb-1a29-4078-a0da-4a64409f44f4.png)

![image](https://user-images.githubusercontent.com/110227904/186152061-0431d10d-824b-4add-837c-9611e7e441aa.png)


**Optional exercie is to add second SINK - PARQUET file:**


![image](https://user-images.githubusercontent.com/110227904/186152416-9eed3fb3-ff47-4b26-a8f8-ef36dddf99eb.png)

![image](https://user-images.githubusercontent.com/110227904/186152587-b4d464d7-79eb-43d8-bd5e-71bbf49030b9.png)

![image](https://user-images.githubusercontent.com/110227904/186152732-fa658fb3-3518-4088-be0b-f00f1b5fa417.png)

![image](https://user-images.githubusercontent.com/110227904/186152848-cf2523a3-0ac0-4fda-8f96-8466de51317c.png)

![image](https://user-images.githubusercontent.com/110227904/186152929-db9d1b27-6e8b-44f2-9d48-eb81e4805935.png)

![image](https://user-images.githubusercontent.com/110227904/186153127-d5adb6bf-9593-48f4-8808-d18f625f5d87.png)


The below step will request you to confirm change of partitioning - accept the change

![image](https://user-images.githubusercontent.com/110227904/186155203-08a9f76b-edc5-4428-bf23-ab89a90b76d2.png)


**End of optional step.**



**At this stage we have reeady dataflow, the next step is to create the pipeline**

![image](https://user-images.githubusercontent.com/110227904/186154188-ee53d25d-ff33-49bb-abd8-2839fa2b6acb.png)

![image](https://user-images.githubusercontent.com/110227904/186154460-7968bd87-33d6-4a1e-b7bd-9db136f0d143.png)

![image](https://user-images.githubusercontent.com/110227904/186154680-91834f8e-750d-4944-bd0b-cf601d0fc42b.png)

![image](https://user-images.githubusercontent.com/110227904/186154817-dc64039e-e660-46ab-ade3-3ab8a2245dcf.png)


**Validate your data in SQL table and parquet file:**

![image](https://user-images.githubusercontent.com/110227904/186157674-e9563e7d-c291-4b24-a828-2b81c281e911.png)


Optional step: Create trigger for a pipeline







