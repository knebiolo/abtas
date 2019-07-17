# ABTAS

Aquatic Bio-Telemetry Analysis Software (ABTAS) for use in removing false positive and overlap detections from aquatic radio telemetry projects.

# Introduction
ABTAS, Kleinschmidt�s radio telemetry analysis software is comprised a suite of Python scripts and an importable python module (abtas.py).  Each of the scripts carries out a task one may have when analyzing radio telemetry data, including: creating a standardized project database, importing raw data directly from receiver downloads, identifying and removing false positive detections, cross validating and assessing the quality of training data, producing an array of standard project statistics (no. recaptured at receiver, etc.), removing overlap between neighboring receivers with larger detection zones, and producing data appropriate for analysis with Competing Risks and Mark Recapture (Cormack Jolly Seber and Live Recapture Dead Recover) methods.  In future iterations of the software, we will replace the functional sample scripts with a Jupyter Notebook to guide users through training, classification, and data preparation.

The scripts in their current form take advantage of the multiprocessing module when necessary to make the data filtering process more efficient.  When activated, all system resources will be utilized unless otherwise directed by the end user.  **It is not recommended to create more processes than CPUs in your computer, please understand the limitations of your machine before proceeding.**

Radio telemetry projects create vast quantities of data, especially those that employ in-water beacon tags.  Therefore, it is advised to have at least 10 GB of hard disk space to store raw, intermediate, and final data products, and at least 8 GB of RAM.  To handle this data, the software creates a SQLite project database.  SQLite is an in-process library that implements a self-contained, server-less, zero configuration, transactional SQL database engine (SQLite, 2017).  More importantly, SQLite can handle simultaneous reads, so it is well-suited for write-once, read-many largescale data analysis projects that employ parallel processing across multiple cores.  To view the SQLite project database download either: [sqlite browser](http://sqlitebrowser.org/) or [sqlite studio](https://sqlitestudio.pl/index.rvt) .  Both SQLite viewers can execute SQL statements, meaning you can create a query and export to csv file from within the database viewer.  They are not that different from Microsoft Access, users familiar with databases will have an easy transition into SQLite.

Starting in 2019, Kleinschmidt Associates (primary developer) will begin hosting the code on Atlassian Bitbucket.  With internal and external collaborators, version control has become an issue and the software is no longer standard among users.  Bitbucket is a web-based version control repository hosting service, which uses the Git system of version control.  It is the preferred system for developers collaborating on proprietary code, because users must be invited to contribute.  The Git distributed version control system tracks and manages changes in source code during development.  This is important when more than 1 user of the open source software adapts it to use on their project.  

The software is written in Python 3.7.x and uses dependencies outside of the standard packages.  Please make sure you have the following modules installed and updated when running telemetry analysis: Numpy, Pandas, Networkx, Matplotlib, Sqlite3, and Statsmodels.  The software also uses a number of standard packages including: Multiprocessing, Time, Math, Os, Datetime, Operator, Threading and Collections.  

This repository includes sample scripts that guide a user through a telemetry project.  However, you could import abtas into your own proprietary scripts and data management routines.  These scripts are examples only, if you push changes to the script, the owner may not commit them.

# Project Set Up
The simple 4-line script �project_setup.py� will create a standard file structure and project database in a directory of your choosing.  **It is recommended that the directory does not contain any spaces or special characters.**  For example, if our study was of fish migration in the Connecticut River our initial directory could appear as (if saved to your desktop):
> C:\Users\UserName\Desktop\Connecticut_River_Study

When a directory has been created, insert it into the script in line 2 (highlighted below).  Then, edit line 3 to name the database.  **It is recommended to avoid using spaces in the name of the database.**  Once lines 2 and 4 have been edited, run the script.  

projct_setup.py example:
```
import abtas
proj_dir = 'J:\1210\005\Calcs\Studies\3_3_19\2019'
dbName = 'ultrasound_2019.db'
abtas.createTrainDB(proj_dir, dbName)  # create project database
```

Before proceeding to the next step, investigate the folder structure that was created.  The folders should appear as:

* Project_Name
    * Data *for storing raw data and project set up files*
	    * Training Files *for storing raw receiver files the station you are currently working up*
	* Output *for storing figures, modeling files, and scratch data*
	    * Scratch *holds intermediate files that are managed by the software* - **never delete the contents of this folder**
		* Figures *holds figures exported by the software*

**Do not alter the structure of the directory.** Both the sample scripts provided and ABTAS expect that the directory is structured exactly as created.

# Initializing the Project Database
Once the project directory has been created, we will initialize the project database by declaring parameters and importing formatted data files that describe the tags and receivers used in the study.  If one of the objectives of your study is to analyze movement or you have close-spaced receivers and overlap is an issue, then we will also need to create a Node table.  This table describes the telemetry network and helps us visualize the relationships between receivers.  Please save initial database files to the �Data� folder (see directory structure above if you forgot where it was) as comma delimited text files (.csv).  

## Master Tag Table
The tag table must match the following schema and be saved as a comma delimited text file.  Field names must appear exactly as in the first column and data types must match.  Please save the file to the �Data� folder and call it �tblMasterTag.csv�.  

| Field      | Data Type |                                      Comment                             |
|------------|-----------|--------------------------------------------------------------------------|
|FreqCode    |String     |(required) combination of radio frequency and tag code.  **must be unique**|
|PIT_ID      |Integer    |(not required) if tagged with RFID tag indicate ID                        |
|PulseRate   |Float      |(required) seconds between tag pulses                                     |
|MortRate    |Float      |(not required) if equipped, seconds between tag pulses if fish has died   |
|CapLoc      |String     |(required) capture location of fish                                       |
|RelLoc      |String     |(required) release location of fish                                       |
|TagType     |String     |(required) either 'Study' or 'Beacon                                      |
|Length      |Integer    |(not required) - mm                                                       |
|Sex         |String     |(not required) - either 'M' or 'F'                                        |
|RelDate     |DateTime   |(required) - Date and time of release                                     |

## Master Receiver Table
The receiver file must contain the following fields and should be saved as a comma delimited text file.  Please see the schema below.  As with the master tag file, please save the master receiver file to the �Data� folder.    A good name for this file is �tblMasterReceiver.csv�.  

| Field      | Data Type |                                      Comment                             |
|------------|-----------|--------------------------------------------------------------------------|
|Name        |String     |(not required) - common name of reciever location,  e.g. 'Norse Farms'    |
|RecType     |String     |(required) - acceptable inputs are either **'lotek'** or **'orion'**      |
|recID       |String     |(required) - alphanumeric ID for receiver, e.g. 'T01'                     |
|Node        |String     |(required) - alphanumeric ID for network node, e.g. 'S01'.                |

## Telemetry network
If one of your objectives is to analyze how fish move through a study area, you will need to create a table that describes relationships between receivers (single receivers or groupings) by identifying the logical pathways that exist between them.  Figure 1 depicts a telemetry network of a recent project completed by Kleinschmidt Associates.  Each point on the picture is either a single telemetry receiver or group of receivers.  These points are known as nodes and represent telemetered river reaches.  The receiver-to-node relationship is mapped in the master receiver table with the �Node� column.  The lines, or edges in Figure 1 depict the relationships between nodes.  Some are double headed while others are one way.  Some edges are one way because it is impossible for a fish to swim up through a hydroelectric turbine.  This type of graph is known as a directed acyclic graph.  For now, we only need to identify the nodes and give them arbitrary XY coordinates.  

![Example Telemetry Network](https://i.ibb.co/zNtHCwS/telem-network-sm.png)

Like the tag and receiver tables, the node table will be saved as a comma delimited text file in the �Data� folder.  The XY coordinate data is meant to produce a schematic of the telemetry study (Figure 1).  **We do not recommend the use of actual XY coordinate data (latitude, longitude) because relationships between near-adjacent nodes may be hard to view and aquatic networks are often sinuous.**  The node table has the following schema below: 

| Field      | Data Type |                                      Comment                             |
|------------|-----------|--------------------------------------------------------------------------|
|Node        |String     |(required) - alphanumeric ID for network node **must be unique**          |
|Reach       |String     |(not required) - common name of reach monitored by node, e.g. 'Cabot Tailrace'|
|X           |Integer    |(required) - arbitrary X coordinate point                                 |
|Y           |Integer    |(required) - arbitrary Y coordinate point                                 |

Note: There may be more than 1 receiver associated with a node.  For example, a fishway may have two differences, but for the purposes of the study you only have to know if a fish has entered the fishway.  It is logical to group them into a single network node.  Doing so will greatly simplify movement modeling.  The receiver to node relationship is developed in the master receiver table with the node column.  IDs must match between columns for relationships to work.

Once the initial data files have been created and stored in the �Data� folder, we will need to import them into the project database.  We will complete this task with the �project_db_ini.py� script (see below).  You will need to follow these steps:

1.	Update line 4, identify the project directory (same directory you created prior)  
2.	Update line 5, identify the project database name
3.	Update line 8, set the number of detections we will look forward and backwards from the current while creating detection histories.  **The default is 5.**  
4.	Update line 9, set the duration used when calculating the noise ratio.  **The default is 1 minute.**  
5.	If you are not assessing movement, and do not have a node table, then comment out lines 13, 19 and 20 before running the script by adding a �#� to the beginning of the line.  

example project_db_ini.py:
```
import os
import abtas
import pandas as pd
proj_dir = r'J:\1210\005\Calcs\Studies\3_3_19\2018\Test'                       # what is the project directory?
dbName = 'ultrasound_2018_test.db'                                             # whad did you call the database?
data_dir = os.path.join(proj_dir,'Data')                                       
db_dir = os.path.join(proj_dir,'Data',dbName)                                  
det = 5                                                                        # number of detections we will look forwards and backwards for in detection history
duration = 1                                                                   # duration used in noise ratio calculation
# import data to Python
tblMasterTag = pd.read_csv(os.path.join(data_dir,'tblMasterTag.csv'))
tblMasterReceiver = pd.read_csv(os.path.join(data_dir,'tblMasterReceiver.csv'))
tblNodes = pd.read_csv(os.path.join(data_dir,'tblNodes.csv'))                  # no nodes?  then comment out this line
# write data to SQLite
abtas.studyDataImport(tblMasterTag,db_dir,'tblMasterTag')
print ('tblMasterTag imported')
abtas.studyDataImport(tblMasterReceiver,db_dir,'tblMasterReceiver')
print ('tblMasterReceiver imported')
abtas.studyDataImport(tblNodes,db_dir,'tblNodes')                              # no nodes? then comment out this line
print ('tblNodes imported')                                                    # no nodes? then comment out this line
abtas.setAlgorithmParameters(det,duration,db_dir)
print ('tblAlgParams data entry complete, begin importing data and training')
```

# False Positive Removal
Radio telemetry receivers record four types of detections based upon their binary nature; true positives, true negatives, false positives and false negatives (Beeman and Perry, 2012). True positives and true negatives are valid data points that indicate the presence or absence of a tagged fish. A false positive is a detection of a fish�s presence when it is not there, while a false negative is a non-detection of a fish that is there. False negatives arise from a variety of causes including insufficient detection areas, collisions between transmitters, interference from ambient noise, or weak signals (Beeman & Perry, 2012). Inclusion of false negatives may negatively bias statistics as there is no way to know if a fish�s absence from a receiver was because it truly wasn�t there or if it was not recaptured by the receiver. While the probability of false negatives can be quantified from sample data as the probability of detection, quantifying the rate of false positives (type I error) is more problematic (Beeman & Perry, 2012). Inclusion of false positives in a dataset can bias study results in two ways: they can favor survivability through a project by including fish that weren�t there or increase measures of delay when a fish has already passed. There are no statistical approaches that can reduce bias associated with false positives, therefore they must be identified and removed *a priori*. ABTAS identifies and removes false positive detections with a Na�ve Bayes classifier and removes recaptures resulting from overlapping detections zones with an algorithm inspired by nested Russian dolls. 







