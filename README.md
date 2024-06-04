# <div align="center">ANALYSIS OF AIRLINES PERFORMANCES🛬</div>
## <div align="center">![Intro](images/iStock-498532108-916x517-1.jpg)

## Introduction
This report provides an analysis of flight delays and cancellations for an airline. The goal is to understand the key factors causing these issues and suggest ways to improve. By examining various data points, we aim to provide clear insights and actionable recommendations to enhance flight punctuality and reliability. First, we cover the process of uploading a data file to HDFS, followed by analyzing the data using Pig, and finally, visualizing the results with Python packages.

## Problem Statement
Frequent flight delays and cancellations affect the airline's operations and passenger satisfaction. Identifying the primary reasons behind these disruptions and finding solutions to minimize them is crucial for improving service quality and operational efficiency.

## Objectives:
1) Identify the main causes of flight delays and cancellations.

2) Determine the optimal times of the day, days of the week, and months of the year to minimize delays and cancellations.

3) Provide actionable recommendations to improve flight punctuality and reliability.

## Dataset 📖
The source of the data is from [Airline On Time Data](https://www.kaggle.com/datasets/wenxingdi/data-expo-2009-airline-on-time-data/data?select=1993.csv)
### **There are 4 main datasets that we will use:**
- 2008.csv 
- plane-data 
- carriers 
- airports

### **The main questions of interest in the dataset:**
- What are the optimal times of day, days of the week, and times of the year for minimizing flight delays?
- What are the primary factors contributing to flight delays?
- What factors predominantly lead to flight cancellations?
- Which flight experiences the most frequent and significant delays and cancellations?

We will answer all the questions using Pig

## Methodology
**Uploading Datasets to Hadoop File System**

Here are the steps to upload the datasets (2008, plane-data, carriers, and airports) to the Hadoop File System:

**Step 1: Transfer File to Virtual Machine**

**Command Prompt:**
>![Command Prompt](images/commandprompt.png)

Explanation:

In this step, the pscp command (PuTTY Secure Copy) is used to transfer the 2008.csv file from your local Windows machine (specified by the user file path) to the home directory of the user maria_dev on the virtual local machine. The file is now accessible in the virtual machine but not yet in the Hadoop file system.

**Putty:**
>![Putty](images/putty.png)

Explanation:

Here, the hdfs dfs -put command is used to move the 2008.csv file from the local file system of the virtual machine (where it resides in /home/maria_dev/) to the Hadoop Distributed File System (HDFS). The file is uploaded to the directory /user/maria_dev/flight_data/ in HDFS, making it available for processing by Hadoop.

**Learning Data Manipulation Using Pig**

1. **Data Loading:**
   
- Load datasets containing flight records, airport information, carrier details, and plane data into PIG.
