# Yet (another) ETL Framework

Expressive, agile, fun for Data Engineers using Python!

```
pip install yetl-framework
```

## What is YETL?



YETL is a data engineering configuration framework for building modern cloud datalake houses with [Apache Spark][apache_spark], [Databricks][databricks], [DeltaLake][delta_lake]; hence the name Yet (another) ETL Framework. In time could support many more DSL frameworks and table formats e.g. Pandas and Iceberg

Using yetl a data engineer can define configuration to take data from a source dataset to a destination dataset and just code the transform in between. It takes care of the mundane allowing the engineer to focus only on the value end of data flow in a fun and expressive way.

Summary:

- Uses a generic decorator pattern i.e. the framework can change, improve and extend but your existing pipelines will require NO changes
- It defines data flows between source datasets and destination datasets
- It takes care of the mundane best practice using configuration that's easy to maintain and build with no death by configuration!
- No compromises on engineering practices allowing you to left shift the specifics of data flows in tight dev test loops.
- It supports landing formats onboarded into a timesliced structured datalake
- It supports Delta Lake tables
- You can develop, test locally or remotely
- It provides Timeslice native type to inject into loads to run incremental loads, partial bulk loads and full reloads
- It provides easy to extend save write types that can be used from configuration or injected into loads
- Can be used with any orchestrator or worklow tool


## What is it really?

The best way to see what it is, is to look at a simple example.

Define your tables:

```yaml
version: 1.3.0

audit_control:
  delta_lake:
    raw_dbx_patterns_control:
      header_footer:
        sql: ../sql/{{database}}/{{table}}.sql
        depends_on:
          - raw.raw_dbx_patterns.*
      raw_audit:
        sql: ../sql/{{database}}/{{table}}.sql
        depends_on:
          - raw.raw_dbx_patterns.*
          - audit_control.raw_dbx_patterns_control.header_footer

landing:
  read:
    landing_dbx_patterns:
      customer_details_1: null
      customer_details_2: null

raw:
  delta_lake:
    raw_dbx_patterns:
      customers:
        ids: id
        depends_on:
          - landing.landing_dbx_patterns.customer_details_1
          - landing.landing_dbx_patterns.customer_details_2
        warning_thresholds:
          invalid_ratio: 0.1
          invalid_rows: 0
          max_rows: 100
          min_rows: 5
        exception_thresholds:
          invalid_ratio: 0.2
          invalid_rows: 2
          max_rows: 1000
          min_rows: 0
        custom_properties:
          process_group: 1
```

Define you load configuration:

```yaml
version: 1.3.0
tables: ./tables.yaml

audit_control:
  delta_lake:
    # delta table properties can be set at stage level or table level
    delta_properties:
        delta.appendOnly: true
        delta.autoOptimize.autoCompact: true
        delta.autoOptimize.optimizeWrite: true
    managed: false
    create_table: true
    container: datalake
    location: /mnt/{{container}}/data/raw
    checkpoint_location: "/mnt/{{container}}/checkpoint/{{checkpoint}}"
    path: "{{database}}/{{table}}"
    options:
      checkpointLocation: default

landing:
  read:
    trigger: customerdetailscomplete-{{filename_date_format}}*.flg
    trigger_type: file
    container: datalake
    location: "/mnt/{{container}}/data/landing/dbx_patterns/{{table}}/{{path_date_format}}"
    filename: "{{table}}-{{filename_date_format}}*.csv"
    filename_date_format: "%Y%m%d"
    path_date_format: "%Y%m%d"
    format: cloudFiles
    spark_schema: ../schema/{{table.lower()}}.yaml
    options:
      # autoloader
      cloudFiles.format: csv
      cloudFiles.schemaLocation:  /mnt/{{container}}/checkpoint/{{checkpoint}}
      cloudFiles.useIncrementalListing: auto
      # schema
      inferSchema: false
      enforceSchema: true
      columnNameOfCorruptRecord: _corrupt_record
      # csv
      header: false
      mode: PERMISSIVE
      encoding: windows-1252
      delimiter: ","
      escape: '"'
      nullValue: ""
      quote: '"'
      emptyValue: ""
    
raw:
  delta_lake:
    # delta table properties can be set at stage level or table level
    delta_properties:
      delta.appendOnly: true
      delta.autoOptimize.autoCompact: true    
      delta.autoOptimize.optimizeWrite: true  
      delta.enableChangeDataFeed: false
    managed: false
    create_table: true
    container: datalake
    location: /mnt/{{container}}/data/raw
    path: "{{database}}/{{table}}"
    checkpoint_location: "/mnt/{{container}}/checkpoint/{{checkpoint}}"
    options:
      mergeSchema: true

base:
  delta_lake:
    container: datalake
    location: /mnt/{{container}}/data/base
    path: "{{database}}/{{table}}"
    options: null
```

Import the config objects into you pipeline:

```python
from yetl import Config, StageType

pipeline = "auto_load_schema"
project = "test_project"
config = Config(
    project=project, pipeline=pipeline
)
table_mapping = config.get_table_mapping(
    stage=StageType.raw, table="customers"
)

print(table_mapping)
```

Use even less code and use the decorator pattern:

```python
@yetl_flow(
        project="test_project", 
        stage=StageType.raw
)
def auto_load_schema(table_mapping:TableMapping):

    # << ADD YOUR PIPELINE LOGIC HERE - USING TABLE MAPPING CONFIG >>
    return table_mapping # return whatever you want here.


result = auto_load_schema(table="customers")
```

## Example Project

[databricks-patterns](https://github.com/sibytes/databricks-patterns)

This is an example project that loads file sources from landing. The files are semi structured and have header and footer information with table pf data in between the meets a specific schema.

The data is loaded in the respective tables in raw, the headers and footer and shredded into a header and footer audit table. Finally all the audit information is brought to together and loaded into the audit table that can used to raise alerts or exceptions based on row counts and schema errors.

