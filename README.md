# SQL Server Integration Services

This tutorial offers an introduction into Microsoft's SQL Server Integration services (SSIS). SSIS is used for building an enterprise level data integration and data transformation solutions. The external data source may be an OLTP system, a spreadsheet or a flat file. The data source we are considering in this document is a Microsoft Excel workbook.

This document is divide into two parts

1. Installation and instruction for SSIS
2. An example to illustrate how to use SSIS to perform ETL (Extract,, Transform, Load) on the excel data source.

After finishing this tutorial, the reader will be able to:

- Understand foundations of the SSIS module and how it supports ETL transformations.
- Apply the theoretical concepts of SSIS in designing and implementing a complete ETL workflow.
- The tutorial uses an Excel file as the data source and a data warehouse schema implemented in MSSQL as the target artifact.
- Apply the following kinds of ETL transformations
- Filter null values.
- Split a field into multiple fields.
- Validate data quality and integrity using foreign key constraints.

**NOTE** 

This tutorial assumes SQL Server Management Studio 17.X installed on your computer.

------

## Installing SQL Server Integration Services

To install SSIS, we first need to install SQL Server. SQL Server is  Microsoft's relational database management system(RDBMS). Follow the below steps for step by step installation of SQL server on your Windows machine.

Go to https://www.microsoft.com/en-us/sql-server/sql-server-downloads and download SQL Server for **Developer**. (Tip: Developer version is a full featured free edition which can be used for a non- production environment)

![](/SC/1.JPG)



Once the download is complete, run the .exe file and select Basic 

![1540627021509](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540627021509.png)



Select *Accept* > *Install*

![1540627136574](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540627136574.png)



Once the installation is complete, hit *close*

![1540627621608](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540627621608.png)



SQL server is now installed on your machine. We need one more tool before be begin using SSIS for our assignment. We need to install SQL Server Data Tools or SSDT which is a tool for building the relational databases, Analysis Services (AS), Integration Services (IS) packages, and Reporting services (SSRS). More information about SSDT can be found [HERE](https://docs.microsoft.com/en-us/sql/ssdt/download-sql-server-data-tools-ssdt?view=sql-server-2017).

*For most users, SQL Server Data Tools (SSDT) is installed during Visual Studio installation. Installing SSDT using the Visual Studio installer adds the base SSDT functionality, so you still need to run the SSDT standalone installer to get AS, IS, and RS tools.*

To download stand alone SSDT installer use the link  - https://go.microsoft.com/fwlink/?linkid=2024393.

Follow the below steps for step by step installation of SQL server Data Tools on your Windows machine.

- Open the .exe file and begin the installation.

- Click on the drop down below *Modify this Visual Studio 2017 instance* and select **Microsoft SQL server Data Tools for Visual Studio 2017 (SSDT)**. NOTE - This is install a separate instance of Visual Studio on your machine if you already have Visual Studio installed. This instance will be called a Microsoft SQL server Data Tools for Visual Studio 2017 (SSDT) instance. Creating this new instance is preferred for simplicity. Check only the SQL Server Integration Services checkbox since we are only dealing with SSIS. Its up to you if you want the install other components which might take more time.

  ![1540629471622](C:\Users\karth\AppData\Roaming\Typora\typora-user-images\1540629471622.png)

- Once the installation is complete, you are all set to perform your first activity of ETL using SSIS as a Data integration and transformation tool.

















