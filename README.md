# Udacity Data Engineering Capstone Project

_________________

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
[![Imports: isort](https://img.shields.io/badge/%20imports-isort-%231674b1?style=flat&labelColor=ef8336)](https://pycqa.github.io/isort/)

_________________

## Contents

1. [Project Summary](#project-summary)  
2. [Data Sources](#data-sources)  
3. [Data Model](#data-model)   
4. [ETL Pipeline](#etl-pipeline)
5. [Other Scenarios](#other-scenarios)
6. [Structure of the project](#structure-of-the-project)

## Project Summary

This project aims to create an ETL pipeline that takes data from 7 sources, processes them and uploads them to a data warehouse. The data warehouse facilitates the analysis of the US immigration phenomenon using Business Intelligence applications. With the help of the data stored in it, it is possible to identify:
- if there is any correlation between the phenomenon of global warming and the origin of immigrants
- if there is an increase/decrease in immigrants from certain states
- which are the main states where immigrants go
- what is the age distribution of immigrants
- etc.

This repository is the result of completing the [Data Engineering Nanodegree](https://www.udacity.com/course/data-engineer-nanodegree--nd027) on Udacity. So the code was tested in Project Workspace on Udacity.

## Data Sources

As mentioned in the previous section, 7 data sources are used in this project. 4 of them are suggested by Udacity Provided Project and 3 of them are taken from various web pages. A small description of each of them can be found below:
- [I94 Immigration Data](https://www.trade.gov/national-travel-and-tourism-office): This dataset contains data about immigrants from the US National Tourism and Trade Office. In addition to the actual data, this dataset also comes with a file in which the codes used are described. For a better understanding of the meaning of the names of the columns in this dataset, I recommend the following projects: [project A](https://notebooks.githubusercontent.com/view/ipynb?browser=chrome&color_mode=auto&commit=41a3047f65ae172a11302e9446151d33dcc86033&device=unknown&enc_url=68747470733a2f2f7261772e67697468756275736572636f6e74656e742e636f6d2f4d6f64696e6777612f446174612d456e67696e656572696e672d43617073746f6e652d50726f6a6563742f343161333034376636356165313732613131333032653934343631353164333364636338363033332f43617073746f6e6525323050726f6a6563742532305375626d697373696f6e2e6970796e62&logged_in=false&nwo=Modingwa%2FData-Engineering-Capstone-Project&path=Capstone+Project+Submission.ipynb&platform=android&repository_id=261688897&repository_type=Repository&version=99) and [project B](https://www.1week4.com/it/machine-learning/udacity-data-engineering-capstone-project/#1.3.2-The-I94-dataset).
- [World Temperature Data](https://www.kaggle.com/datasets/berkeleyearth/climate-change-earth-surface-temperature-data): This dataset came from Kaggle. It provides historical information about monthly average temperatures in different cities around the world.
- [U.S. City Demographic Data](https://public.opendatasoft.com/explore/dataset/us-cities-demographics/export/): This dataset contains information on the demographics of all US cities with a population greater than 63 000.
- [Airport Code Table](https://datahub.io/core/airport-codes#data): Provides information about airports around the world.
- [Country Codes](https://countrycode.org/): This site provides the name and 2-letter code of all countries in the world.
- [US States Codes](https://www23.statcan.gc.ca/imdb/p3VD.pl?Function=getVD&TVD=53971):  This site provides the name and 2-letter code of all US states.
- [Continent Codes](https://www.php.net/manual/en/function.geoip-continent-code-by-name.php): This site provides the name and 2-letter code of all continents.

## Data Model

<p align="center">
  <img width="712" height="618" src="docs/model_schema.png">
</p>  

The schema was created using [dbdesigner](https://app.dbdesigner.net/). The SQL code to create the tables is in [docs/immigration_db_postgres_create.sql](docs/immigration_db_postgres_create.sql). As can be seen from the image, a star schema has been used as a way to model the data because the ultimate goal of the data is to analyze it using Business Intelligence applications. A brief description of the tables is reproduced in the following:  
- **country_temperature_evolution**: is a dimension table whose data source is the World Temperature Data dataset. It stores the average monthly temperatures of each country from 1743 to 2013.
- **demographic**: is a dimension table whose data source is the U.S. City Demographic Data dataset. It contains population data for each US state.
- **world_airports**: is a dimension table whose data sources are the Airport Code Table and Continent Codes datasets. It contains data about all airports in the world.
- **us_states**: is a dimension table whose data source is the US States Codes dataset. It contains the name and 2-letter code of all US states.
- **visa**: is a table of dimensions whose data source is the I94 Immigration Data dataset and its description file. It contains all valid visa information.
- **applicant_origin_country**: is a dimension table whose data source is the description file in the I94 Immigration Data dataset. It contains a 3-digit code and the name of each country from which an immigrant could come.
- **status_flag**: is a dimension table whose data source the I94 Immigration Data dataset. It contains the one-letter status for different stages that the immigrant went through.
- **admission_port**: is a dimension table whose data source is the description file in the I94 Immigration Data dataset. It contains the code and information about the admission port through which the immigrant passed.
- **arrival_mode**: is a table of dimensions whose data source is the I94 Immigration Data dataset and its description file. It contains information about how the immigrant arrived in the US.
- **date**: is a dimension table whose data source the I94 Immigration Data dataset. It contains all possible dates from the columns in the source dataset.
- **immigran_application**: is the fact table in the data model. It has as a data source both the I94 Immigration Data dataset and the *visa*, *status_flag* and *arrival_mode* tables from which it takes the id columns. This table contains information on the application submitted by the immigrants.

More details related to the columns in the tables can be found in [docs/data_dictionary.md](docs/data_dictionary.md).

## ETL Pipeline

<p align="center">
  <img width="712" height="586" src="docs/etl_design.png">
</p> 

As can be seen in the previous image, the pipeline takes the source data from 3 different places, processes the data and saves it locally in parquet format. Once all the data has been processed, they are uploaded to [Amazon S3](https://aws.amazon.com/s3/). Each table is processed using the following pattern:
1. Raw data is read.
2. The data is transformed.
3. The correctness of the processed data is checked.
4. The processed data are saved in parquet format in the output folder.

The main tools used in this project are:
- [Apache Spark](https://spark.apache.org/docs/latest/api/python/): was chosen for data processing, regardless of their size.
- [Pandas](https://pandas.pydata.org/): was chosen for the ease with which HTML tables are read.
- [Amazon S3](https://aws.amazon.com/s3/): it was chosen because it is highly scalable, reliable, fast and inexpensive data storage.
  
To run the pipeline, the next steps have to be followed:
1. Complete **dl.cfg** configuration file. It is recommended that the KEY and SECRET fields have the values of an IAM User that has only [AmazonS3FullAccess policy](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-iam-awsmanpol.html) attached. For the S3 field, it is necessary to pass the S3 bucket name. Be careful not to put the entered values between single or double quotes.
2. Run the command from the project root.
```
python -m etl
```

## Other Scenarios

#### The data was increased by 100x.

For such a scenario, I would consider using an [Amazon EMR](https://aws.amazon.com/emr/) to run the ETL, and upload the data directly to the [Amazon S3](https://aws.amazon.com/s3/). Besides this, I would partition the tables. For example, I would partition the *country_temperature_evolution* table according to the country.

#### The data populates a dashboard that must be updated on a daily basis by 7am every day.

For this situation, the ETL can be refactor to work with [Apache Airflow](https://airflow.apache.org/), because it would be much easier to automate the execution of the pipeline.

#### The database needed to be accessed by 100+ people.

If the database starts to be used intensively, I would consider moving the data to [Amazon Redshift](https://aws.amazon.com/redshift/).

## Structure of the project

    ├── docs                                # Contains files about data model and ETL pipeline.
    ├── processed_data                      # The folder where the processed data is stored.
    ├── raw_data                            # The folder where the raw data is stored.
    ├── utils                               # Contains files with functions used in the ETL pipeline.
    ├── data_exploration.ipynb              # Jupyter notebook used for data exploration.
    ├── dl.cfg                              # The credentials and config used to manage the AWS resources.
    ├── etl.py                              # Code for the ETL pipeline.
    ├── LICENSE                             # Contains information about the project license
    ├── pyproject.toml                      # Contains the configurations of the linting tools used
    ├── README.md
    └── requirements.txt                    # Contains the list of libraries used
