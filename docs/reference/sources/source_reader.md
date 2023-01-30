# Source Reader

The source reader is the simple batch reader that you can execute in spark. Yetl framework pushes the yaml configuration into the following idoim for reading data in spark:

```python
df = spark.read.format(format).options(**options).load(path)
```

How the SQL reader also has a number of features to make life easy loading from a data lake:

- Slice date partition format parameterisation into the file path
- Using wildcards to load multiple slice date periods
- Handle schema exceptions and load them into a deltalake table
- Set exception and warning thresholds for schema exceptions
- Set exception and warning thresholds for the min and max number of expected records
- Automatically infer the 1st load, create a spark schema and save it into the schema repo
- Add the file, path and or filepath as columns to the data frame
- Adds the slice date into the dataframe of the relative data
- Configure the corrupt record on Permissive loads

## Example

Here is an example of source reader declared in dataflow yaml declaration:

```yaml
dataflow:

  # database name that can be reference in the rest of the
  # configuration by using {{ database_name }}
  demo_landing:
    # table name that be reference in the rest of the
    # configuration by using {{ table_name }}
    customer_details:
      type: Reader     

      properties:
        ## properties and their default values

        # Schema properties
        yetl.schema.createIfNotExists: true
        yetl.schema.corruptRecordName: _corrupt_record
        yetl.schema.corruptRecord: true

        # metadata lineage that can be added into the data set
        yetl.metadata.filepathFilename: true
        yetl.metadata.datasetId: true
        # this will parse the timeslice metadata from the file name (see the path below)
        yetl.metadata.timeslice: timeslice_file_date_format
        yetl.metadata.filepath: false
        yetl.metadata.filename: false
        yetl.metadata.contextId: false
        yetl.metadata.dataflowId: false
  
      # these formats configure how the timeslice jinja variables are
      # formatted for the Timeslice parameter when the data flow is executed
      # 2 are provided because the timestamp in the filename and path can
      # can be different
      # This is used to format the timeslice before replacing {{ timeslice_path_date_format }}
      path_date_format: "%Y%m%d"
      # This is used to format the timeslice before replacing {{ timeslice_file_date_format }}
      file_date_format: "%Y%m%d"
      # the file format that is loaded
      format: csv
      path: "landing/demo/{{ timeslice_path_date_format }}/customer_details_{{ timeslice_file_date_format }}.csv"
      read:
        # toggles on or off the atomated features e.g. schema exception, handling and loading exceptions to a delta table.
        auto: true
        # The spark DSL read options
        options:
          mode: PERMISSIVE
          inferSchema: false
          header: true

      # Configures it to handle exceptions and to which delta table to load them
      # it uses jinja variables so it can be generic across dataflows
      exceptions:
          path: "delta_lake/{{ database_name }}/{{ table_name }}_exceptions"
          database: "{{ database_name }}"
          table: "{{ table_name }}_exceptions"
      # configures thresholds for that if exceeded will raise a warning or exceptions
      thresholds:
        # errors will cause the dataflow to fail and not proceed
        error:
          # how schema error rows are allowed
          exception_count: 0
          # how many schema error rows are allowed as % of the total rows
          exception_percent: 5
          # max rows expected
          max_rows: 1000
          # min rows expected
          min_rows: 1
        # warnings will cause the result set to contain warnings
        # but will not stop the dataflow from proceeding
        warning:
          exception_count: 0
          exception_percent: 5
          max_rows: 1000
          min_rows: 1
```

Yetl will always gracefully handle data flow errors and succeed; however it will give details of warnings and errors in the result set. Errors will have typically stopped the data flow proceeding, warnings however will not. This should allow the calling orchestrator code to get the detailed information and handle it appropriatly thus supporting any orchestrator setting.

Here is an exmaple of the error and warning section of the results set for settings above. Notice the errors and warnings are configure the same so give the same exceptions; because error thresholds the dataflow would have not proceeded. However they can be set independently allowing more elegant handling. 

```json
    "warning": {
        "count": 2,
        "warnings": [
            {
                "ThresholdWarning": "exception_count threshold exceeded 1 > 0"
            },
            {
                "ThresholdWarning": "exception_percent threshold exceeded 9.090909090909092 > 5"
            }
        ]
    },
    "error": {
        "count": 2,
        "errors": [
            {
                "ThresholdException": "exception_count threshold exceeded 1 > 0"
            },
            {
                "ThresholdException": "exception_percent threshold exceeded 9.090909090909092 > 5"
            }
        ]
    }
```