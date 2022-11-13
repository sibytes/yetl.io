# Getting Started

Clone the tutorial.

```sh
git clone https://github.com/sibytes/yetl.tutorial.git
```

Checkout the beginning of the tutorial.

```sh
git checkout getting-started-step-0
```

Explore the project. Currently there's not there at all, mostly a simply directory structure containing some date partitioned test data to simulate our landing data partitioned by timeslice dates.

## Installation

With the project home dir create a virtual python environment and install the required libraries. This will install `yetl-framework` and the python dependencies required for this tutorial.

```sh
python -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

## Step 1 - Create a Yetl Project

Using the Yetl cli create a new project:

```sh
python -m yetl init demo 
```

This will create the following config directory:

![Config](../assets/Step 2 - 1.png)

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


## Step 2 - Create Table Manifest

Using the cli run the table manifest creation on a sample of the source data. This will scan the files and create a table manifest. It's uses a regex parameter to pull out the table name from the filenames:

```sh
python -m yetl create-table-manifest \
"demo" \
"./config/project"  \
File \
"./data/landing/demo" \
--filename "*" \
--extract-regex "^[a-zA-Z_]+[a-zA-Z]+"
```

This will create a `project/demo` folder in the `./config` directory and the table manifest from the data files it scanned. The project folder is where we'll create our pipeline templates that we will build into pipelines.


![Project](../assets/Step 3 - 1.png)

## Step 3 - Create Pipeline Template

In this step will create a pipeline template for loading landing data from `./data/landing/demo` into a set of raw (bronze) deltalake tables. This template uses jinja and will be used to generate all the required pipeline configurations in the table manifest.

Create the file using the UI or the command:
```sh
touch ./config/project/demo/landing_to_raw.yaml
```

Copy and paste the following configuration into `landing_to_raw.yaml`:

```yaml
dataflow:

  demo_landing:
    {{demo_tables_table_name}}:
      type: Reader
      properties:
        yetl.schema.corruptRecord: false
        yetl.schema.createIfNotExists: true
        yetl.metadata.timeslice: timeslice_file_date_format
        yetl.metadata.filepathFilename: true
      path_date_format: "%Y%m%d"
      file_date_format: "%Y%m%d"
      format: csv
      path: "landing/demo/{{timeslice_path_date_format}}/{{demo_tables_table_name}}_{{timeslice_file_date_format}}.csv"
      read:
        auto: true
        options:
          mode: PERMISSIVE
          inferSchema: false
          header: true

  demo_raw:
    {{demo_tables_table_name}}:
      type: DeltaWriter

      partitioned_by:
        - _partition_key

      ddl: "{{root}}"
      properties:
        yetl.metadata.datasetId: true
        yetl.schema.createIfNotExists: true
        delta.appendOnly: false
        delta.checkpoint.writeStatsAsJson: true
        delta.autoOptimize.autoCompact: true       
        delta.autoOptimize.optimizeWrite: true     
        delta.compatibility.symlinkFormatManifest.enabled: false
        delta.dataSkippingNumIndexedCols: -1
        delta.logRetentionDuration: interval 30 days
        delta.deletedFileRetentionDuration: interval 1 week
        delta.enableChangeDataFeed: true
        delta.minReaderVersion: 1
        delta.minWriterVersion: 2
        delta.randomizeFilePrefixes: false
        delta.randomPrefixLength: 2
      
      format: delta
      path: delta_lake/demo_raw/{{demo_tables_table_name}}
      write:
        mode: append
        options:
          mergeSchema: true
```


## Step 4 - Build Pipeline Configuration

In this step we will use the pipeline template `./config/project/demo/landing_to_raw.yaml` to create a pipeline configuration for each table in the table manifest at `./config/project/demo/demo_tables.yml`

Build the pipeline configurations by executing:
```sh
python -m yetl build \
demo \
demo_tables.yml \
landing_to_raw.yaml \
./config
```

## Step 5 - Code Pipeline Transformation

In this step we'll use python and yetl to code a function that loads our tables.

Create a directory called `src` and a python file in that directory called `demo_landing_to_raw.py`.

```sh
mkdir ./src
touch ./src/demo_landing_to_raw.py
```

Add the following code to `demo_landing_raw.py`

```python
from yetl.flow import (
    yetl_flow,
    IDataflow,
    IContext,
    Timeslice,
    TimesliceUtcNow,
    Save,
)
from pyspark.sql.functions import *
from typing import Type

_PROJECT = "demo"
_PIPELINE_NAME = "landing_to_raw"

@yetl_flow(project=_PROJECT, pipeline_name=_PIPELINE_NAME)
def landing_to_raw(
    table: str,
    context: IContext,
    dataflow: IDataflow,
    timeslice: Timeslice = TimesliceUtcNow(),
    save: Type[Save] = None,
) -> dict:
    """Load raw delta tables"""

    source_table = f"{_PROJECT}_landing.{table}"
    df = dataflow.source_df(source_table)

    df = df.withColumn(
        "_partition_key", date_format("_timeslice", "yyyyMMdd").cast("integer")
    )

    destination_table = f"{_PROJECT}_raw.{table}"
    dataflow.destination_df(destination_table, df, save=save)

    context.log.info(f"Loaded table {destination_table}")
```