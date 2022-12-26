# Command Line Interface

Yetl provides a command line interface tool. It can be used to:

- initialise a yetl project
- scrape datasource metadata
- build pipeline configurations from a jinja templates

Please see the [getting-started](../tutorial/gettingstarted.md) to walk through the development workflows using the cli. Yetl is built using [typer](https://typer.tiangolo.com/).

Yetl cli is self documented that can be accessed using the help command:

```sh
python -m yetl --help
```

## Commands

Yetl has the following commands:

|command|description|
|-|-|
| init | Initialise the configuration directory with the required structure and start config files |
| create-table-manifest | Create manifest configuration file containing the names of tables we want create yetl data pipelines on |
| build | Use table manifest file and the pipeline jinja template to build a pipeline configuration for each table |

### init

Description: Use manifest config file and the pipeline jinja template to build a pipeline configuration for each table

Usage:

```sh
python -m yetl init [OPTIONS] project 
```

Arguments:

|argument|type|default|required|
|-|-|-|-|
|project|text|none|yes|


Options:

|option|type|default|
|-|-|-|
|--home-dir|text|.|
|--config-folder|text|config|
|--overwrite or --no-overwrite||no-overwrite|

Example:

From [getting-started](../tutorial/gettingstarted.md)

```sh
python -m yetl init demo 
```

### create-table-manifest

Description: Use manifest config file and the pipeline jinja template to build a pipeline configuration for each table

Usage:

```sh
python -m yetl create-table-manifest [OPTIONS] project build_dir source_type source_dir
```

Arguments:

|argument|type|default|required|
|-|-|-|-|
|project|text|none|yes|
|build_dir|text|none|yes|
|source_type|text|none|yes|
|source_dir|text|none|yes|

Options:

|option|type|default|
|-|-|-|
|--filename|text|*|
|--extract-regex|text|none|

Example:

From [getting-started](../tutorial/gettingstarted.md)

```sh
python -m yetl create-table-manifest \
"demo" \
"./config/project"  \
File \
"./data/landing/demo" \
--filename "*" \
--extract-regex "^[a-zA-Z_]+[a-zA-Z]+"
```

### build

Description: Use table manifest file and the pipeline jinja template to build a pipeline configuration for each table

Usage:

```sh
python -m yetl build project metadata_file template_file build_dir
```

Arguments:

|argument|type|default|required|description|
|-|-|-|-|-|
|project|text|none|yes| name of the project to build the pipeline into |
|metadata_file|text|none|yes| the name of the table manifest configuration file located at $build_dir/project/$project/$metadata_file |
|template_file|text|none|yes| the name of the jinja pipeline yaml template file located at $build_dir/project/$project/$template_file |
|build_dir|text|none|yes| the yetl configuration home directory where the pipelines will be built $build_dir/$project/pipelines/$tablename_$template_file |

Example:

From [getting-started](../tutorial/gettingstarted.md)

```sh
python -m yetl build \
demo \
demo_tables.yml \
landing_to_raw.yaml \
./config
```

This command will use the files at:

- metadata_file = [./config/project/demo/demo_tables.yml](https://github.com/sibytes/yetl.tutorial/blob/main/config/project/demo/demo_tables.yml)
- template_file = [./config/project/demo/landing_to_raw.yaml](https://github.com/sibytes/yetl.tutorial/blob/main/config/project/demo/landing_to_raw.yaml)

To build a pipeline configuration for each table in `demo_tables.yml` using the template file `landing_to_raw.yaml` and store it in the build location [./config](https://github.com/sibytes/yetl.tutorial/tree/main/config) in subdirectory named after the project [.config/demo/pipelines](https://github.com/sibytes/yetl.tutorial/tree/main/config/demo/pipelines). 

For example if there are 2 tables:

- customer_details
- customer_preferences

It will build 2 pipelines files

- [./config/demo/pipelines/customer_details_landing_to_raw.yaml](https://github.com/sibytes/yetl.tutorial/blob/main/config/demo/pipelines/customer_details_landing_to_raw.yaml)
- [./config/demo/pipelines/customer_preferences_landing_to_raw.yaml](https://github.com/sibytes/yetl.tutorial/blob/main/config/demo/pipelines/customer_preferences_landing_to_raw.yaml)