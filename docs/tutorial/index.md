# Tutorial

Yetl is a python Data Engineering framework for Spark and DeltaLake; it therefore assumes some basic knowledge of Data Engineering, Python, PySpark, Spark and DeltaLake and recognises the need and benefits of using a data engineering framework.

The content can be obtained by cloning the [yetl.tutorial](https://github.com/sibytes/yetl.tutorial) repo:

```sh
git clone https://github.com/sibytes/yetl.tutorial.git
```

The repo is branched at each step of the tutorial. Branches are named the same as step titles within the tutorial and mark the code state reached at the end of that step. So by checking out a specific step will place the code as it should be at the end of that tutorial step. For example the following will checkout the code as should be at the beginning of step 1 for the getting-started tutorial.

```sh
git checkout getting-started-step-1
```

At each step of the tutorial you can simply checkout the next step of the tutorial to ensure you're starting the next step with everything as required. Alternatively if you're more confident with git and the topics then checkout the starting position and just work your way through the tutorial. The remote repo is locked so commits cannot be pushed.


Tutorials for learning about yetl:

- Getting Started - A basic tutorial to demonstrate it's features and to get started, tags `step-1` to `step-?` **Work In Progess**.

## Pre-Requisites

The following pre-requisites are required for local environments for tutorials about working locally:

- Python 3.9
- Spark 3.3.0 - see [documentation]() and the many articles on the internet to install spark locally
- Java 11 - [openjdk](https://openjdk.org/install/) is a good option to install java since there are no licensing issues or authentication required
- VSCode - This tutorial assumes that the code editor is vscode on a local linux or mac environment. Any code editor and/or windows can be used but you may need to adjust some steps in this walk through.
- Databricks - Some tutorials will cover how code can be pulled to databricks and executed. These tutorials will assume some familiarity with databricks and access to databricks environment. They will also require the tutorial [data](https://github.com/sibytes/yetl.tutorial/tree/main/data) to be present on the databricks lake storage either in DBFS or mounted S3 or Azure DataLake storage; more specfically at this `/mnt/datalake/yetl_data` path configured in the [tutorial environment configuration](https://github.com/sibytes/yetl.tutorial/blob/main/config/environment/dbx_dev.yaml) `./config/environment/dbx_dev.yaml`. You can of course change this path as you please as long as you update the configfured path.
