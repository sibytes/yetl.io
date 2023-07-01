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

NOTE:

- Lists are entered using character return between items in an Excel cell.
- Dicts are entered using a : between key value pairs and character returns betweenitmes in an Excel cell.


|merge_column.column                  | required | type | description |
|-|-|-|-|
| stage	                              |     y     | [audit_control, landing, raw, base, curated] | The architecural layer of the data lake house you want the DB in |
| table_type	                      |     y     | [read, delta_lake] | What type of table to create, read is a spark.read, delta_lake is a delta table. | 
| catalog	                          |     n     | str | name of the catalog. Although you can set it here in the condif the api allows passing it as a parameter also. |   
| database	                          |     y     | str | name of the database |       
| table	                              |     y     | str | name of the table |       
| sql	                              |     n     | [y, n] | whether or not to include a default link to a SQL ddl file for creating the table. |   
| id	                              |     n     | str, List[str] | a column name or list of column names that is the primary key of the table. |   
| depends_on	                      |     n     | List[str] | list of other tables that the table is loaded from thus creating a mapping. It required the yetl index which is `stage.database.table` you can also use  `stage.database.*` for exmaple if you want to reference all the tables in a database.|   
| deltalake.delta_properties          |     n     | Dict[str,str] | key value pairs of databricks delta properties |       
| deltalake.identity                  |     n     | [y, n] | whether or not to include an indentity on the table when a delta table is created implicitly |
| deltalake.partition_by              |     n     | str, List[str] | column or list of columns to partition the table by |   
| deltalake.delta_constraints         |     n     | Dict[str,str] | key value pairs of delta table constraints |   
| deltalake.z_order_by                |     n     | str, List[str] | column or list of columns to z-order the table by |   
| deltalake.vacuum                    |     n     | int | vaccum threshold in days for a delta table |
| warning_thresholds.invalid_ratio    |     n     | float | ratio of invalid to valid rows threshold that can be used to raise a warning |
| warning_thresholds.invalid_rows     |     n     | int | number of invalid rows threshold that can be used to raise a warning |
| warning_thresholds.max_rows         |     n     | int | max number of rows thresholds that can be used to raise a warning |
| warning_thresholds.mins_rows        |     n     | int | min number of rows thresholds that can be used to raise a warning |
| error_thresholds.invalid_ratio      |     n     | float | ratio of invalid to valid rows threshold that can be used to raise an exception |
| error_thresholds.invalid_rows       |     n     | int | number of invalid rows threshold that can be used to raise an exception |
| error_thresholds.max_rows           |     n     | int | max number of rows thresholds that can be used to raise an exception |
| error_thresholds.mins_rows          |     n     | int | min number of rows thresholds that can be used to raise an exception |
| custom_properties.process_group     |     n     | any | customer properties can be what ever you want. Yetl is smart enough to build them into the API |
| custom_properties.rentention_days   |     n     | any | |
| custom_properties.anything_you_want |     n     | any | |


Create the tables.yaml file by executing:

```
python -m yetl import-tables ./my_project/pipelines/tables.xlsx ./my_project/pipelines/tables.yaml
```

## Create Pipeline Metadata

In the `./my_project/pipelines` folder create a yaml file that contains the metadata specifying how to load the tables defined in `./my_project/pipelines/tables.yaml`. You can call them whatever you want and you can create more than one. Perhaps one that batch loads and another that event stream loads. The yetl api will allow you to parameterise which pipeline metadata you want to use.

Please see the pipeline reference documentation for details. Here is an [example](https://github.com/sibytes/databricks-patterns/blob/main/header_footer/pipelines/autoloader.yaml).

## Create Spark Schema

## Develop & Test Pipeline

## Deploy Product


