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

|argument|type|default|required|
|-|-|-|-|
|project|text|none|yes|
|metadata_file|text|none|yes|
|template_file|text|none|yes|
|build_dir|text|none|yes|

Example:

From [getting-started](../tutorial/gettingstarted.md)

```sh
python -m yetl build \
demo \
demo_tables.yml \
landing_to_raw.yaml \
./config
```
