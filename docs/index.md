# Yet (another) ETL Framework

Expressive, agile, fun for Data Engineers using Python!

```
pip install yetl-framework
```

## What is YETL?



YETL is a data engineering framework for building modern cloud datalake houses with [Apache Spark][apache_spark], [Databricks][databricks], [DeltaLake][delta_lake]; hence the name Yet (another) ETL Framework. In time could support many more DSL frameworks and table formats e.g. Pandas and Iceberg

Using yetl a data engineer can define configuration to take data from a source dataset to a destination dataset and just code the transform in between. It takes care of the mundane allowing the engineer to focus only on the value end of data flow in a fun and expressive way rather than filling the day with dread.

Summary:

- Uses a generic decorator pattern i.e. the framework can change, improve and extend but your existing pipelines will require NO changes
- It defines data flows between source datasets and destination datasets
- It takes care of the mundane best practice using configuration that's easy to maintain and build with no death by configuration!
- It's built for engineers and how they work from automatically creating the initial schema's, the development process, TDD, integration testing and deployment
- No compromises on engineering practices allowing you to left shift the specifics of data flows in tight dev test loops.
- It supports landing formats onboarded into a timesliced structured datalake
- It supports Delta Lake tables
- You can develop, test locally or remotely
- It can be developed and/or deployed onto Databricks with no code changes
- It provides Timeslice native type to inject into loads to run incremental loads, partial bulk loads and full reloads
- It provides easy to extend save write types that can be used from configuration or injected into loads
- Can be used with any orchestrator or worklow tool
- Can be used with any data expectations python API


## What is it really?

The best way to see what it is, is to look at a simple example. There are many ways to define a data flow, and that's what great about yetl, it doesn't take that away from you nor does it take away the great DSL API's provided by [spark][apache_spark]

This is a simple data flow that loads from 2 landed data files, typically you wouldn't integrate data from landing to raw but this shows that multiple sources and destinations is supported. It also loads a specific table however this doesn't have to be the case there are generic patterns we can use for many tables using a low code footprint but we can still inject specific transforms for all tables or some tables. You can even define your own save semantics and inject it into the load. In other words it can be as expressive as you want it to be.

```python
@yetl_flow(project="demo")
def landing_to_raw(
    context: IContext,
    dataflow: IDataflow,
    timeslice: Timeslice = TimesliceUtcNow(),
    save: Type[Save] = None,
) -> dict:
    """Load the demo customer data as is into a raw delta hive registered table.

        The config for this dataflow has 2 landing sources 
        that are joined and written to delta table
    """

    df_cust = dataflow.source_df(
        f"{context.project}_landing.customer"
    )
    df_prefs = dataflow.source_df(
        f"{context.project}_landing.customer_preferences"
    )

    context.log.info("Joining customers with customer_preferences")
    df = df_cust.join(df_prefs, "id", "inner")
    df = df.withColumn(
        "_partition_key", date_format("_timeslice", "yyyyMMdd").cast("integer")
    )

    dataflow.destination_df(f"{context.project}_raw.customer", df, save=save)
```

To run an incremental load

```py
timeslice = Timeslice(2021, 1, 1)
results = landing_to_raw(timeslice=timeslice)
```

To run a bulk load, it's this simple!

```python
timeslice = Timeslice("*")
results = landing_to_raw(
    timeslice=timeslice, 
    save=OverwriteSave
)
```

To reload a year!

```python
timeslice = Timeslice(2021, "*", "*")
results = landing_to_raw(
    timeslice=timeslice, 
    save=OverwriteSave
)
```

## Features

When the data flow runs above what did it do? Well what it does depends on how it's configured. With this **TODO** configuration this is what it does:

- Automatically mapped through the timeslice to path location of the source data in the landing data store
- If there wasn't a spark schema present in the schema repository, it creates one for you automatically; yes it has a schema repo!
- Applies schema validation exception and warning thresholds
- Loads schema excepted data into deltalake exception tables that are also automatically created
- Loads the valid data into source dataframes
- Adds data lineage metadata to the dataframe
- Creates the deltalake destination table if it deoesn't exist
- Creates or uses a deltalake table SQL DLL schema definition in the schema repo
- Creates or updates the configured deltalake table properties
- Creates or updates deltalake constraint properties
- Creates a partition definition on the deltalake table
- Writes the data to the destination deltalake table
- Automerges schema changes into the deltalake table if configured & supported
- Optimises and Z-Orders the destination deltalake table based on the Z-Order columns and partitions affected by the write
- Outputs a full datalineage and audit of operations that were performed in a hierarchical uniquely identified structure of context > dataflow > dataset
- The incremental load uses the default save which is the configured save
- Partitial bulk or full reloads can override the default save by injecting a different Save type that implements a different patter
- If it fails the dataflow will still succeed, but provide a full audit of what it did with errors so it can logged and/or handled accordingly
- Just a python library no external databases or moving parts required, but can be extended to use datastore if required


## Is it safe to use?

This is currently an experimental release, whilst it is tested and stable to a degree in terms of the dataflow interface. We're currently in a period of hardening and cleaning the internals.

As of now we're developing and testing with:

 - pyspark 3.3.1
 - delta-spark 2.1.1



[apache_spark]: https://spark.apache.org/
[delta_lake]: https://delta.io/
[databricks]: https://databricks.com


## Development Workflows

Development workflows are important and it's what makes a framework great or terrible. Does it make life easier or is a wrestle to apply your tradecraft? Does it fill the day with dread or is it expressive, agile & fun!

This is an example of how a yetl development workflow can go, loosly in order since all experienced engineers know that micro-iterations is how good quality software is really crafted:

- Use the cli to create a new project template in vscode
- Get some small or representative test datasets and copy them into the vscode project at the configured local lake path
- Use the cli to scan the datafiles and create a table manifest that you can tweak and curate
- Create a template configuration for a inferred pipeline that generates all the schema's, a simple few lines of properties in 1 config file
- Use the cli to build the pipeline configurations for all the tables in the manifest
- Run the pipeline that will automatically generate spark schema's and deltatable SQL definitions for everything and load the data
- Use a data profiler on the loaded tables
- Now incrementally start refining schema's, date formats, expectations, partitions, z-ordering, build unit tests etc using whatever great python libraries you want
- Switch off the schema inferrence in 1 config simply rebuild the pipeline config and commit the spark and delta schema's to the repo
- At convenient points deploy onto a cloud environment like databricks and start refining against a bigger sample of data

