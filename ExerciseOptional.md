# For those not willing to run Python code but willing to learn more about Form Recognizer models

Please visit site:
https://formrecognizer.appliedai.azure.com/studio

During the labs we will help you to configure it and you can use it with you own files.


# Load data from PDF file with Spark Notebooks

This is optional exercise where we will execute Python that will call Form Recognizer Cognitive Service.
The code is already prepared by us in the Notebook:

![image](https://user-images.githubusercontent.com/110227904/188098575-94cddfca-a9a5-4285-a017-5a6ba6e720fd.png)

The data (PDF files) that we are going to analyze can be found in the Data Lake in apg/pdf folder:

![image](https://user-images.githubusercontent.com/110227904/188099855-5b020d31-2c25-46b9-bf1c-c7c10b2a7c04.png)

To execute code start with first CELL. **The first time you run it**, it will take several minutes to start the cluster. Time needed to execute next cells will be faster as cluster will be already started.

![image](https://user-images.githubusercontent.com/110227904/188100553-16875328-54f7-4b1b-892f-2d522f8d768c.png)

Third CELL is the place where code will invoke Form Recognizer APIs and analyze your PDF files.
We have prepared 3 PDF files (provided by APG) ready to Analyze.
You can comment/uncomment code for a given file to get different results.

![image](https://user-images.githubusercontent.com/110227904/188101632-3a24ad8e-1f6b-4477-8da8-fb61104285fe.png)


The last CELL is where you can format your PDF output and save it to CSV.
If you decide to run this code - please change the CSV file name so you won't overwrite your own results:

![image](https://user-images.githubusercontent.com/110227904/188102786-1ec54793-4e3e-48e0-9e07-6ff80e56d213.png)

After succesfuly running a code you can use SQL Serverless to ad-hoc query your CSV file (or you can download it to your laptop and open with Excel).

![image](https://user-images.githubusercontent.com/110227904/188103196-5b0d81c8-34bc-4392-b6f8-75d81158c030.png)

![image](https://user-images.githubusercontent.com/110227904/188103384-c905fbe6-5580-432b-bc01-5ba6fe39ce33.png)




