# Tutorial

A basic tutorial to demonstrate it's features and to get started.

## Pre-Requisites

The following pre-requisites are required for this tutorial:

- Python 3.9
- Spark 3.3.0
- Java 11

This tutorial assumes that the code editor is vscode on a local linux or mac environment. Any code editor can be used but you may need to adjust some steps in this walk through.

## Step 1 - Install Python Dependencies

With the project home dir create a virtual python environment and install the required libraries. This will install `yetl-framework` and the python dependencies required for this tutorial.

```sh
pip -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

## Step 2 - Create a Yetl Project

Using the Yetl cli create a new project:

```sh
python -m yetl init demo 
```

This will create the following config directory:

![Config](./assets/Step 2 - 1.png)

The directories and files in config are as follows:

- **environment** - Contains global level settings for yetl for the respective environment e.g. the root of the datalake where data resides. This is how yetl can be configured to run the same code locally and on databricks
- **logging.yaml** - Contains the python logging configuration
- **schema** - This is the basic schema repo that comes with yetl that stores SQL and Spark schema's organised into files on disk
- **demo** - This is where the pipeline configuration and SQL will reside for the project demo when yetl builds the configuration.

Yetl init will also create a `.venv` file containing the following:

```
YETL_ROOT=./config
YETL_ENVIRONMENT=local
```

The `.env` can be used with vscode or your IDE to create environment variables. These environment variables are required by Yetl so that it know what environment it is and where the configuration resides.


## step 3 - Create Table Manifest

Using the cli run the table manifest creation on a sample of the source data. This will scan the files and create a table manifest. It's uses a regex parameter to pull out the table name from the filenames:

```sh
python -m yetl create-table-manifest \
"demo" \
"./config/project"  \
File "/Users/shaunryan/AzureDevOps/yetl.tutorial/data/landing/demo" -\
-filename "*" \
--extract-regex "^[a-zA-Z_]+[a-zA-Z]+"
```

This will create a `project/demo` folder in the `./config` directory and the table manifest from the data files it scanned. The project folder is where we'll create our pipeline templates that we will build into pipelines.

![Project](./assets/Step 3 - 1.png)