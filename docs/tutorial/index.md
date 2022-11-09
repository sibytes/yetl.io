# Tutorial

Yetl is a python Data Engineering framework for Spark and DeltaLake; it therefore assumes some basic knowledge of Data Engineering, Python, PySpark, Spark and DeltaLake and recognise the need and benefits of using a data engineering framework.

The content can obtained by cloning the [yetl.tutorial](https://github.com/sibytes/yetl.tutorial) repo:

```sh
git clone https://github.com/sibytes/yetl.tutorial.git
```

The repo is tagged at each step of the tutorial. Tags are named the same as step titles within the tutorial and mark the code state reached at the end of that step. So by checking out a specific step will place the code as it should be at the end of that tutorial step. For example the following will checkout the code as should be at the end of step 0. 

```sh
git checkout tags/step-0
```

At each step of the tutorial you can simply checkout the step you've finished working on to ensure you starting the next step with everything as required. Alternatively if you're more confident with git and the topics then checkout the tag into a working branch and make your own commits. The remote repo is locked so commits cannot be pushed.


Tutorials for learning about yetl:

- Getting Started - A basic tutorial to demonstrate it's features and to get started, tags `step-1` to `step-?` **Work In Progess**.

## Pre-Requisites

The following pre-requisites are required for this tutorial:

- Python 3.9
- Spark 3.3.0 - see [documentation]() and the many articles on the internet to install spark locally
- Java 11 - [openjdk](https://openjdk.org/install/) is a good option to install java since there are no licensing issues or authentication required

This tutorial assumes that the code editor is vscode on a local linux or mac environment. Any code editor can be used but you may need to adjust some steps in this walk through.
