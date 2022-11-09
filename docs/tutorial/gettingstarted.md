# Getting Started


## Step 1 - Installation

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


## Step 3 - Create Table Manifest

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

## Step 4 - Create Pipeline Template

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
      table:
        properties:
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
      exceptions:
          path: "delta_lake/demo_landing/{{table_name}}_exceptions"
          database: demo_landing
          table: "{{table_name}}_exceptions"
  

  demo_raw:
    {{demo_tables_table_name}}:
      type: DeltaWriter
      table:

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