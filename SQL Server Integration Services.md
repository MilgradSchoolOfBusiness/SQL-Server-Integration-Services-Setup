# SQL Server Integration Services

This tutorial offers an introduction into Microsoft's SQL Server Integration services (SSIS). SSIS is used for building an enterprise level data integration and data transformation solutions. The external data source may be an OLTP system, a spreadsheet or a flat file. The data source we are considering in this document is a Microsoft Excel workbook.

This document is divide into two parts

1. Installation and instruction for SSIS
2. An example to illustrate how to use SSIS to perform ETL (Extract,, Transform, Load) on the excel data source.

After finishing this tutorial, the reader will be able to:

-  Understand foundations of the SSIS module and how it supports ETL transformations.

- Apply the theoretical concepts of SSIS in designing and implementing a complete ETL workflow.

-  The tutorial uses an Excel file as the data source and a data warehouse schema implemented in MSSQL as the target artifact.

-  Apply the following kinds of ETL transformations

  - Filter null values.
  - Split a field into multiple fields.
  - Validate data quality and integrity using foreign key constraints.


**NOTE** 

This tutorial assumes SQL Server Management Studio 17.X installed on your computer.

------

## 1. Installing SQL Server Integration Services

To install SSIS, we first need to install SQL Server. SQL Server is  Microsoft's relational database management system(RDBMS). Follow the below steps for step by step installation of SQL server on your Windows machine.

Go to https://www.microsoft.com/en-us/sql-server/sql-server-downloads and download SQL Server for **Developer**. (Tip: Developer version is a full featured free edition which can be used for a non- production environment)

![](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540626796181.png)



Once the download is complete, run the .exe file and select Basic 

![1540627021509](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540698672793.png)



Select *Accept* > *Install*

![1540627136574](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540698692323.png)



Once the installation is complete, hit *close*

![1540627621608](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540698708800.png)



SQL server is now installed on your machine. We need one more tool before be begin using SSIS for our assignment. We need to install SQL Server Data Tools or SSDT which is a tool for building the relational databases, Analysis Services (AS), Integration Services (IS) packages, and Reporting services (SSRS). More information about SSDT can be found [HERE](https://docs.microsoft.com/en-us/sql/ssdt/download-sql-server-data-tools-ssdt?view=sql-server-2017).

*For most users, SQL Server Data Tools (SSDT) is installed during Visual Studio installation. Installing SSDT using the Visual Studio installer adds the base SSDT functionality, so you still need to run the SSDT standalone installer to get AS, IS, and RS tools.*

To download stand alone SSDT installer use the link  - https://go.microsoft.com/fwlink/?linkid=2024393.

Follow the below steps for step by step installation of SQL server Data Tools on your Windows machine.

- Open the .exe file and begin the installation.

- Click on the drop down below *Modify this Visual Studio 2017 instance* and select **Microsoft SQL server Data Tools for Visual Studio 2017 (SSDT)**. NOTE - This is install a separate instance of Visual Studio on your machine if you already have Visual Studio installed. This instance will be called a Microsoft SQL server Data Tools for Visual Studio 2017 (SSDT) instance. Creating this new instance is preferred for simplicity. Check only the SQL Server Integration Services checkbox since we are only dealing with SSIS. Its up to you if you want the install other components which might take more time.

  ![1540629471622](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540629471622.png)

- Once the installation is complete, in the start menu, open **Visual Studio 2017 (SSDT)**

  ![Open Visual Studio SSDT](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540701810484.png)

- You are all set to perform your first activity of ETL using SSIS as a Data integration and transformation tool.

## 2. Using SSIS to perform ETL Transformations

This section is divided into three parts -

1. A discussion of the source and target schemas

2.  A brief introduction to SSIS ETL transformation terminology.

3.  Detailed steps illustrating sample transformation of Hubspot data from an Excel spreadsheet into our

   data warehouse schema.


2.1 Source and Target Schemas

Source and Target Schemas We use Hubspot data from the Teradata Data Challenge 20162 as the data source for our data warehouse. Hubspot3 is a company having a software product by the same name. The schema as shown in below represents a consolidated view to support web analytics for the inbound marketing efforts across multiple social media channels such as Facebook, Twitter, LinkedIn. Some of the fields are null, and some of the data must be transformed to be useful. This data will be loaded into a multidimensional schema for further analysis, but it must be cleaned and transformed first. The data consists of the following fields - 

![Fields in hubspot data](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540698588212.png)

```tex
The hubspot data for this tutorial refers to the Excel file provided in the supporting files accompanying this tutorial. The name of the file is: hubspot-social-media-export-01-08-16.xls.
```

The figure below shows us our data warehouse schema. The schema has one fact (`fact_interactions`) and two dimensions (`dim_channel` and `dim_time`). The attributes *ChannelId* and *TimeNo* in the `fact_interactions` table are the foreign keys to the two dimension tables `dim_channel` and `dim_time`. 

![Our data warehouse schema](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540699145973.png)

You can create the above schema in MSSQL using the DDL script provided in the accompanied files. The name of the DDL script is: hubspot (Microsoft SQL Server Query File).

At this point, we have our input and output artifacts in place . In preparation to understand the example, we offer a brief introduction to the terminology related to the SSIS Data Integration module in the next section.



### 2.2 SSIS Data Integration Module Terminology

This section introduces and explains terms used in the SSIS Data Integration software. We discuss selected terms that are used in our example.

A data flow task is composed of transformations which are connected by *data flow paths*. Transformations have different kind of functionality based on the transformation type. For example, the Conditional split transformation  filters data based on a defined set of conditions. Similarly, the Microsoft Excel Input *source transformation* allows reading data from an Excel spreadsheet. The steps are connected by *data flow paths* that allow the data to flow from the source to the target. Figure below shows an example of a transformation in which data is read from an Excel spreadsheet (Microsoft Excel Input Source Transformation) and written to a text file (Flat file Destination Transformation).

![Simple transformation](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540701412547.png)



### 2.3 Illustrative Example

In this section we present step-by-step instructions for populating our data warehouse schema  from the Hubspot data. The first two subsections (2.3.1 and 2.3.2) below illustrate populating dimension tables while subsection 2.3.3 populates a fact table. We provide detailed steps for populating the dimension table dim_time and leave the loading of the other dimension table dim_channel as an exercise. Before discussing the dimension and fact tables, we present a list of the ETL transformations that we implement in our example.

1. Read data from the Hubspot Excel spreadsheet.

2. Parse the PublishTime column from our source data and split it into three fields: TimeDay, TimeMonth and TimeYear to allow loading data into the dim_time table.

3. Populate the fact table fields Facebook_likes, Facebook_comments and Total_Interactions based on the channel and publish time information in the original dataset.

4. Validate the date and channel information against the corresponding values from the dimension tables.

5. Populating only the valid TimeNo values in the fact table based on the foreign key from the dim_time table.

We now show the detailed steps to achieve all of the above transformations.

#### 2.3.1 Populating the time dimension table

In this section, we cover the loading of the data in the dim_time dimension table of our sample warehouse schema.  We assume that SSIS module is running and the IDE is open with a new data integration project created.

 The complete transformation for dim_time table consists of four steps: (a) connecting to the source data, (b) filtering null values, (c) splitting the publishtime field into three fields: TimeDay, TimeMonth and TimeYear, and (d) loading the data into the MSSQL table dim_time.

1. Go to IDE and open package.dtsx from SSIS Packages on the solution explorer on the right. This opens up the **SSIS Designer** as shown below.

   ![Basic project](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540702223131.png)

2. Drag the **Data flow task** from the **SSIS toolbox** on the left to the SSIS Designer on the **control Flow** tab.  A **control flow** defines a workflow of tasks to be executed, often a particular order.

   ![1540702391644](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540702391644.png)

3.  Double click on the Data Flow Task. This open the Data flow tab.

   ![1540702658714](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540702674071.png)

4. Scroll down on the SSIS toolbox to find **Excel Source** under the **Other Sources** tab.

   ![1540702833626](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540702833626.png)

5. Double click on it to configure it. Select *New > browse to the path where the spreadsheet is located on you machine > Check  "First row has column names" checkbox > OK.*

   ![1540702970158](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540702986580.png)

6. Select "Name of the excel sheet" to the appropriate one.

   ![1540703109729](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540703109729.png)

7. If you now select "Columns" on the left panel, you should be able to see this - 

   NOTE : Make sure all the columns are selected.

   ![1540703178916](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540703178916.png)

8. Hit OK for the window to close.



To make sure that your excel source is configured properly, you should not see any error icon over the Excel source transformation.

Now our Source is ready to transfer it's data. Our destination is going to be a ADO NET Destination where we will load the `dim_time` table assuming the tables are already created by you on the database specified to you.

Before loading the `dim_time` table we have to make sure the Campaign column in our excel should not contain any NULL values. To do this drag the **Conditional Split** Transformation from the SSIS toolbox to the Data Flow designer.

![1540703730235](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540703730235.png)



The below steps will illustrate how you can configure the conditional split transformation to filter NULL values from the Campaign column.

1. Click on the Excel Source transformation and drag the **Blue**  Data flow Path onto the Conditional Split transformation. (Tip: The **RED** Data Flow paths are meant for error logging purpose. It can be useful when a transformation fails)

   ![1540703931965](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540704012931.png)

2. Double click on Conditional Split > Expand "NULL Functions" on the top right panel.

   ![1540704124312](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540704124312.png)

3. Drag and drop the **ISNULL(<<expression>>)** function onto the condition filed as shown below.

   ![1540704337163](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540704337163.png)

4. Drag and drop the *publishTime* column from the left panel onto the <<expression>> placeholder. Since we need to get only NOT NULL values, prefix the ISNULL condition with "!" so that it now becomes as what is shown below.

   ![1540704582869](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540704618509.png)

5. In the above step, we can also rename the "Output Name" to something meaningful instead of "Case 1". The **Default output name** can remain untouched or renamed depending upon your naming preferences. The Default Output Name outputs all the values which doesn't meet the condition we have specified.

6. Hit OK.

We have successfully configured our conditional split to filter out the null values.  You can now start the data flow task by clicking on **Start**.

![1540705362812](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540705362812.png)



This will start the current data flow task. When the task completes you will be able to see the below screen. 

![1540705459178](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540705459178.png)



At the bottom of the screen you will be able to see that the task was completed successfully. You will also be able to see the number of rows that were fed into our conditional split transformation. Stop the task and go back to the designer.



We are yet to transfer the filtered rows to another transformation.

Below steps illustrate how you can take the output of the conditional split transformation to make three new derived columns Day, Month and Year for our `dim_time` table.

1. Drag and drop **Derived Column Transformation** on the designer. (Tip: Derived Column Transformation are usually used to create/replace a column with a new logic)

2. Create  the  data flow path from Conditional split transformation to the Derived Column Transformation. You will be able to see the **Input Output Selection** window pop open.

   ![1540706087920](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540706087920.png)

3. Select the output as Filter_Campaign and hit OK. Filter_Campaign is the output port from the conditional split transformation which will give us the NOT NULL values for the Campaign column.

   ![1540706208724](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540706208724.png)

4. Double click on the Derived Column transformation and expand the "Date/Time Functions". Drag the "DAY", "Month" and "YEAR" functions one by one onto the expressions field to create three new Derived Columns as shown below.

![1540706440387](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540706440387.png)

5. Replace each of the <<date>> placeholder with the publishedDate column from the left panel and rename the columns to Day, Month and Year respectively.

![1540706953157](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540706953157.png)

6. Hit OK.





