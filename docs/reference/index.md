# Overview

YETL allows you to define a datafeed using easy to maintain yaml and sql ddl configuration that takes care of the typical mundane requirements of a datafeed whilst providing an api to implement what's special about them using the python DSL and SQL of a data engine (e.g. spark). It also allows you to configure environments including a local envinorment giving you complete control to use the best software engineering practices to implement datafeeds.

## The Anatomy of YETL'ing Datafeeds!

The components of a YETL project are:

- [Environments](environments.md)
- Projects
- Pipelines
- Schema Repositories
- Logging


A source code project can have many YETL datafeed projects. A YETL datafeed project can have many dataflow pipelines:

```
Source Code Project < Datafeed Projects < datafeed pipelines
```

Configuration resides at root of the yetl source code project folder. By default this is `./config` or it can defined using a environment variable:

```sh
YETL_ROOT=./config
```

