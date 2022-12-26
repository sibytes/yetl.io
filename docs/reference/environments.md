# Overview

The environment configuration is located in yaml files at `$YETL_ROOT/envirioment` that us `./config/environment` by default. The name of the file indicates the name of the environment. The environment can be set by setting the `YETL_ENVIRONMENT` environment variable. For example setting the environment variable as follows uses the configuration in the yaml file `./config/local.yaml`

```sh
YETL_ENVIRONMENT=local
```

In .vscode the configuration set up for a source code project and local environment can be achieved using the following files:

`./.env`

```sh
YETL_ROOT=./config
YETL_ENVIRONMENT=local
```

`./config/environment/local.yaml` 
```yaml
datalake: "{{cwd}}/data"

engine:
  spark:
    logging_level: ERROR
    config:
      spark.master: local
      # yetl uses table properties so this must be set as a table
      # property or globally like here in the spark context
      spark.databricks.delta.allowArbitraryProperties.enabled: true
      spark.jars.packages: io.delta:delta-core_2.12:2.1.1
      spark.sql.extensions: io.delta.sql.DeltaSparkSessionExtension
      spark.sql.catalog.spark_catalog: org.apache.spark.sql.delta.catalog.DeltaCatalog
      spark.databricks.delta.merge.repartitionBeforeWrite.enabled: true

pipeline_repo:
  pipeline_file:
    pipeline_root: "./config/{{project}}/pipelines"
    sql_root: "./config/{{project}}/sql"

spark_schema_repo:
  spark_schema_file:
    spark_schema_root: ./config/schema/spark

deltalake_schema_repo:
  deltalake_sql_file:
    deltalake_schema_root: ./config/schema/deltalake
```

The configuration file sets the location of a number services that yetl uses to configure and execute datafeed pipelines. 

These services are:

## datalake

The data lake path that hosts the data.

The path supports the following jinja variables that are replaced at runtime:

|variable|description|
|-|-|
| {{ cwd }} | Replaced with the current working directory at runtime thus allowing you to store development and test data sets in the project itself where ever you choose. Note this is usually inappropriate and not an obvious location for cloud environments since storage and compute is often separated|

## engine

Sets the data processing engine and the configuration for it. The 1st key in the `engine` configuration sets the type of data processing engine that will be used. At this time only spark configured for deltalake is supported.


### spark

The configuration for spark accepts the following configuration properties.

|property|description|
|-|-|
| logging_level | The current spark logging configuration |
| config | Key value pairs passed to spark session creation |


Example

``` yaml
engine:
  spark: # determines the engine that will be used
    logging_level: ERROR
    config:
      spark.master: local
      # yetl uses table properties so this must be set as a table
      # property or globally like here in the spark context
      spark.databricks.delta.allowArbitraryProperties.enabled: true
      spark.jars.packages: io.delta:delta-core_2.12:2.1.1
      spark.sql.extensions: io.delta.sql.DeltaSparkSessionExtension
      spark.sql.catalog.spark_catalog: org.apache.spark.sql.delta.catalog.DeltaCatalog
      spark.databricks.delta.merge.repartitionBeforeWrite.enabled: true
```

## pipeline_repo

The `pipeline_repo` configures where the built pipeline configuration is stored that the yetl uses when running data flow pipelines.

The 1st key in the `pipeline_repo` configures the type of repo that is used to store the pipeline configuration. Yetl currently provides the following types of pipeline repo's:

- `pipeline_file` - stores the pipeline configuration in yaml files in the project file system.

### pipeline_file

|property|description|
|-|-|
| pipeline_root | File path to where the runtime yaml configuration files are held for datafeed pipelines |
| sql_root | File path to where the runtime SQL statement files are held for datafeed pipelines |


Both paths support the following jinja variables that are replaced at runtime:

|variable|description|
|-|-|
| {{ project }} | The name of the datafeed project, since each source code project can have multiple yetl datafeed projects |

To keep the project intuitively organised wihin the source code project it's recommended to keep them in location configured in the `YETL_ROOT` environment variable which is `./config` by default.

Example

```yaml
pipeline_repo:
  pipeline_file:
    pipeline_root: "./config/{{project}}/pipelines"
    sql_root: "./config/{{project}}/sql"
```

## spark_schema_repo

Currently since the only supported data engine processing is spark then yetl also requires you to configure a spark schema repo. The spark schema repo is where yetl will create, store and load spark schema's for loading data using schema on read.

The 1st key in the `spark_schema_repo` configures the type of repo that is used to store the spark schema. Yetl currently provides the following types of repo's:

- `spark_schema_file` - stores spark schema as yaml files in the project file system.

### spark_schema_file

|property|description|
|-|-|
| spark_schema_root | The root path to where the runtime yaml spark schema files are saved to / or loaded from when schema on read loading files using the [Reader]() dataset|

Schema defintitions use the native form of spark schema and are stored in a sub-directory that matches the name of the database and in a filename that matches the name of the table. See the [Reader]() dataset for more details.

To keep the project intuitively organised wihin the source code project it's recommended to keep them in location configured in the `YETL_ROOT` environment variable which is `./config` by default.

Example

```yaml
spark_schema_repo:
  spark_schema_file:
    spark_schema_root: ./config/schema/spark
```

## deltalake_schema_repo

The deltalake schema repo is where yetl will create, store and load deltalake schema's for creating and maintaining deltalake tables.

The 1st key in the `deltalake_schema_repo` configures the type of repo that is used to store the table defintions for deltalake tables. Yetl currently provides the following types of repo's:

- `deltalake_sql_file` - stores deltalake tables using SQL create table files in the project file system.

### deltalake_sql_file

|property|description|
|-|-|
| deltalake_schema_root | The root path to where the runtime create table SQL files are saved to / or loaded from when loading data into deltalake tables using the [DeltaWriter]() dataset|

Schema defintitions use the deltalake SQL language and are stored in a sub-directory that matches the name of the database and in a filename that matches the name of the table. See the [DeltaWriter]() dataset for more details.

To keep the project intuitively organised wihin the source code project it's recommended to keep them in location configured in the `YETL_ROOT` environment variable which is `./config` by default.

Example

```yaml
deltalake_schema_repo:
  deltalake_sql_file:
    deltalake_schema_root: ./config/schema/deltalake
```
