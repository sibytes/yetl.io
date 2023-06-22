# On-Board Datafeed

These are the outline steps to on-board a new datafeed:

1. Load Landing Data
2. Create Yetl Project
3. Create Table Metadata
4. Create Pipeline Metadata
5. Create Spark Schema
6. Develop & Test Pipeline
7. Deploy Product


## Load Landing Data

This project is about loading data from cloud blob into databricks deltalake tables. Data would normally be orchestrated into landing using some other tool for example Azure Data Factory or AWS Data Pipeline or some other tool that specialises in securely wholesale copying datasets into the cloud or between cloud services.

You may already have this orchestration in place in which case the data will in your landing blob location already or you mock it by getting a sample of data and manually copying it into the location it will be landed to.

For test driven development you can include a small vanilla hand crafted data set into the project itself that can automatically be copied into place from the workspace files as way of creating repeatable end to end integration tests that can staged, executed and torn down with simple commands.

## Create Yetl Project

Create a directory, setup virtual python environment and install yetl.

```sh
mkdir my_project
cd my_project
python -m venv venv
source venv/bin/activate
pip install yetl-framework
```

Create yetl project scaffolding.

```sh
python -m yetl init my_project 
```

## Create Table Metadata

Fill out the spreadsheet template with landing and deltalake architecture that you want to load.
The excel file has to be in a specific format other the import will not work. Use this [example](https://github.com/sibytes/databricks-patterns/blob/main/header_footer/pipelines/tables.xlsx) as a template


|merge_column.column                  | required | default | type | description |
|-|-|-|-|-|
| stage	                              |          |         |      |                                                                                      |
| table_type	                      |          |         |      |                                                                                      |   
| database	                          |          |         |      |                                                                                      |       
| table	                              |          |         |      |                                                                                      |       
| sql	                              |          |         |      |                                                                                      |   
| id	                              |          |         |      |                                                                                      |   
| depends_on	                      |          |         |      |                                                                                      |   
| deltalake.delta_properties          |          |         |      |                                                                                      |       
| deltalake.identity                  |          |         |      |                                                                                      |
| deltalake.partition_by              |          |         |      |                                                                                      |   
| deltalake.delta_constraints         |          |         |      |                                                                                      |   
| deltalake.z_order_by                |          |         |      |                                                                                      |
| deltalake.vacuum                    |          |         |      |                                                                                      |
| warning_thresholds.invalid_ratio    |          |         |      |                                                                                      |
| warning_thresholds.invalid_rows     |          |         |      |                                                                                      |
| warning_thresholds.max_rows         |          |         |      |                                                                                      |
| warning_thresholds.mins_rows        |          |         |      |                                                                                      |
| error_thresholds.invalid_ratio      |          |         |      |                                                                                      |
| error_thresholds.invalid_rows       |          |         |      |                                                                                      |
| error_thresholds.max_rows           |          |         |      |                                                                                      |
| error_thresholds.mins_rows          |          |         |      |                                                                                      |
| custom_properties.process_group     |          |         |      |                                                                                      |
| custom_properties.rentention_days   |          |         |      |                                                                                      |
| custom_properties.anything_you_want |          |         |      |                                                                                      |


Create the tables.yaml file by executing:

```
python -m yetl import-tables ./my_project/pipelines/tables.xlsx ./my_project/pipelines/tables.yaml
```

## Create Pipeline Metadata

## Create Spark Schema

## Develop & Test Pipeline

## Deploy Product


