# Azure-DE-services-practice

![image](https://github.com/user-attachments/assets/5cb6029d-1c40-4bb5-8b2a-dcbb16220e92)

In this small I decided to explore some DE services in Azure, I will be working with a small dataset provided by Scibearia on Kaggle: https://www.kaggle.com/datasets/scibearia/meat-consumption-per-capita.
This dataset has information about the meat consumption per capita. It has 2 files, one contains the information related to the consume of diverse types of meat by country and year. The other one has info related to GDP and some more columns.
The goal here is to use Azure data factory, similar to AWS Glue, to do some transformations, and then, persist all information in an Azure SQL database. Creating a simple pipeline.

---

## 1. Getting Started: Preparing Azure Services
1. First of all, I need to create a resource group in azure, to group all the resources I will using in this lab.

![image](https://github.com/user-attachments/assets/81aa2580-96ad-4be3-814c-0346e201648d)

2. Then I need to create a storage account, to save the JSON files about meat consumption:
  ![image](https://github.com/user-attachments/assets/b7a1c16e-0050-49cb-9205-6d8164166d67)

3. Then, I created the Azure SQL database. You can apply for a discount, Azure immediately offers the promotion, and the first Azure database will cost you 0 USD.
   
![image](https://github.com/user-attachments/assets/460b2b75-7512-4407-80fb-08ca569352a0)


## 2. Azure Data Factory
Azure Data Factory is the one of the main services to do transformations. I need to create a simple workspace in first place (Azure guided me throgh this process) and then you can launch Azure data studio.
I will use a basic data Flow, just to see if the information is being consumed properly. I connected the initial JSON file to an Azure SQL database, just to see if in the preview in Azure SQL step I can see data. It is ok.

![image](https://github.com/user-attachments/assets/014e0d25-2f92-40fd-94c7-2a6911ef626c)

The main idea is to join 2 files:
The one who contains information about meat consumption
Other one that contains the GDP by country and year, and some more information.

I want to create a new table in Azure SQL that contains all the columns for the first file, and 2 columns from the second one (Continent and GDP). I need to join this 2 files to do that. Azure offers this feature in the drag and drop tool:

![image](https://github.com/user-attachments/assets/5a856d44-7c28-4cdd-8370-b77d59bf05d6)

And then I can select by which columns I want to join. In my case I need to join by Country and Year, each country has a different GDP every year.

![image](https://github.com/user-attachments/assets/dacc9556-b120-4ef0-9485-3604596ec27b)

The final pipeline would be this one:

![image](https://github.com/user-attachments/assets/2be168fe-b159-45ee-8d77-e44cc6fb5900)

This create the new file JSON and save it in the Blob storage account, it worked perfectly. When the row match by year and country with the file "GDP", the new JSON contains the new 2 columns.

![image](https://github.com/user-attachments/assets/96c14866-e994-42ff-a586-07e35d1e1c58)

![image](https://github.com/user-attachments/assets/01148990-2326-4b3c-bee3-ad33ca9b82bd)


BUT, NULL != empty
Too basic, but this is a problem when I want to persist the info into Azure SQL database. Why? 
Because Azure SQL table needs to has columns for all the rows, no matter if the row has information or not.
I can resolve this with a simple logic, forcing to add the new column (if matches records or not), but adding a simpe logic.

![image](https://github.com/user-attachments/assets/5d75d9be-e386-42d1-a294-de533d77f88a)

iif(isNull(toString({GDP per capita, PPP (constant 2017 international $)})), '', toString({GDP per capita, PPP (constant 2017 international $)}))
iif(isNull(toString({Сontinent})), '', toString({Сontinent}))

![image](https://github.com/user-attachments/assets/af7dace4-a1d4-4fe8-9672-fa1838d26244)

With this logic, I can ensure that the fields will be String and if a cell is null, will be empty. So all records will have all the columns.




## 3. Azure SQL Database
Finally I create a pipeline, with the transformation logic inside, taking the transformation as a source and setting the Azure SQL database previosuly configurated as a destination.

![image](https://github.com/user-attachments/assets/060f6104-2ea7-4ffe-b18f-86c62dd7f954)

It is a good practice to set up variables at te level of Azure SQL configuration. I created 2 variables, one related to schema and other related to table, so I can specifiy the name of both at the pipeline.

![image](https://github.com/user-attachments/assets/30cd6b06-a14e-4422-bbc7-5f37c3ee044f)

It is important to check the "Auto create table", so the pipeline can create it in the database.

![image](https://github.com/user-attachments/assets/16344178-04e0-442d-9b31-f59237872bff)

19 seconds took to ADF to persist the 10 thousands records into Azure SQL database.

![image](https://github.com/user-attachments/assets/a92c7158-5bd6-4f56-befd-2538c811488d)

Finally I can see the information in the database:

![image](https://github.com/user-attachments/assets/e198aa3d-738c-4bd2-9827-12e49acf783a)

For this practice I created all the resources in US-East 2, I spent a total of 0.65 USD, considering all the tests I made.

![image](https://github.com/user-attachments/assets/4c4cc31b-428c-4d2d-bed2-ee6c7db24d05)

