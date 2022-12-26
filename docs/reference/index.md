# Overview

YETL allows you to define a datafeed using easy to maintain yaml and sql ddl configuration that takes care of the typical mundane requirements of a datafeed whilst providing an api to implement what's special about them using the python DSL and SQL of a data engine (e.g. spark). It also allows you to configure environments including a local environment giving you complete control to use the best software engineering practices to implement datafeeds.

## The Anatomy of YETL'ing Datafeeds!

The components of a YETL project are:

- [Environments](environments.md)
- [Projects](projects.md)
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

The best way to easily create a new spark yetl project is to use the starter [template](https://github.com/sibytes/yetl.tutorial/tree/getting-started-step-1) from the tutorial and initialise it with the [cli](cli.md) init command. The [cli](cli.md) is also used to build pipelines from templates. 

Also see the [getting-started](../tutorial/gettingstarted.md) tutorial to work through the creation and setup of yetl project.