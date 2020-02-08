---
title: Databricks unit testing framework
subtitle: suitable for CI/CD workflows generating Junit xmls, coverage reports – (Python)
category:
  - DevOps
author: Ashish Menkudale
date: 2020-03-01T19:59:59.000Z
featureImage: /uploads/marc-olivier-jodoin-nqoinj-ttqm-unsplash.jpg
---

Databricks has blessed Data Science community with a convenient and robust infrastructure for data analysis. Spinning up clusters, spark backbone, language interoperability, nice IDE, and many more delighters have made life easier.

It’s more of notebooky interface is a great alternative for conventional Jupyter users. And Data Scientists - mostly who are not orthodox python hard coders, love this interface.

## Problem statement

When it comes to productionizing models developed in databricks notebooks, these workflows in notebooks present a little different problem for devops / build engineers. The conventional ways of unittesting python modules, generating Junit compatible xml reports, or coverage reports through command shell do not work as is in this new workflow. Also, Data Scientists working in databricks tend to use ‘dbutils’ dependencies – the databricks custom utility which provides secrets, notebook workflows, widgets etc. delighters as part of their routine model/project development. 

Within these development cycles in databricks, incorporating unit testing in a standard CI/CD workflow can easily become tricky.

## What this post is about

This post is about a simple setup for unittesting python modules / notebooks in databricks. This setup is compatible for typical CI/CD workflow. This is a middle ground for regular python 'unittest' module's framework and databricks notebooks. Similar strategy can be applied for Jupyter notebook workflow on local system as well.

This strategy is my personal preference. This might not be an optimal solution; feedback/comments are welcome. Meanwhile, here’s how it works.

### 1. Folder structure

The first place to start is a folder structure for repo. At high level, the folder structure should contain at least two folders, workspace and dbfs

  -	Workspace
      -	Module1
          -	Notebook1
          -	Notebook2
      -	Module2
          -	Notebook1
          -	Notebook2
      -	Utilities
          -	Orchestration Notebook 1
          -	Orchestration Notebook 2
      -	Setup
          -	Notebook to setup env
      -	Schedulers
      -	Tests
          -	Trigger
          -	test_Notebook1
          -	test_notebook2 
  -	dbfs
      -	property _files
      -	intermediate_tables

Workspace folder contains all the modules / notebooks. dbfs folder contains all the intermediate files which are to be placed on dbfs.

The notebooks in module folders should be purely data science models or scripts which can be executed independently. In these notebooks, databricks’s dbutils dependency should be limited to accessing scopes and secrets. These are the notebooks, for which we will have unittesting triggered through notebooks in the 'test' folder.

Utilities folder can have notebooks which orchestrates execution of modules in any desired sequence. These notebooks can have dbutils.notebook.run commands. 

Tests folder will have unittesting scripts and one trigger notebook to trigger all test_Notebooks individually.

### 2. Why this folder structure

The intended CI flow, will be:

  1. Initial desired stages....
  2. Git checkout repo on devops agent
  3. Devops agent connects with databricks through cli
  	-- workspace folder on workspace
	-- dbfs folder on dbfs
	-- workspace folder on dbfs
  4. Databricks job submit to trigger the 'Trigger' notebook which calls individual test_notebooks
  5. Read XML / coverage reports
  6. further desired stages....
  
Yes, we have kept workspace (codebase) on dbfs as well. More specifically, we need all the notebooks in the modules on dbfs. The intention is to have an option of importing notebooks in these modules as stand alone, independent python modules inside testing notebooks to suit unittest setup.

We will append the path where we kept our codebase on dbfs through sys.append.path() within testing notebook. This enables python to import these modules / Notebooks. (section 4, first 2 commands)

### 3.	Python Notebooks

In conventional python way, we would have a unittest framework, where our test class inherits unittest.Testcase ending with a main(). This main() calls bunch of tests defined within the class. And through command shell, using pytest, this test script will be triggered. Also through command shell, Junit xmls can be generated with pytest --junitxml=path command. Similarly with pytest-cov extention, coverage report can be generated.

Because I prefer developing unit testing in the notebook itself, the above option of calling test scripts through command shell is no longer available. pytest does not support databricks notebooks (it supports jupyter/ipython notebooks through nbval extentions)

To understand the proposed way, let's first see how a typical python module notebook should look like.

Below is template for Notebook1 from Module1.

```python
# Databricks notebook source
#python imports
import sys
import os
from datetime import datetime, timedelta
import pandas as pd
import numpy as np

# COMMAND ----------

#pyspark imports
import pyspark.sql.functions as F
from pyspark.sql.window import Window
from pyspark.sql.types import DateType, StructType, StructField, StringType


# COMMAND ----------

# if this notebook is intended to be run on other env. e.g. jenkins agent or other VM,
# spark session need to be explicitely created.
spark = SparkSession.builder.appName('some').enableHiveSupport().getOrCreate()

# COMMAND ----------

# explicitely creating dbutils
# if this notebooks need to be executed through databricks' default python, 
# this function can access dbutils from pyspark.dbutils.
# in other scenarios we can pass dbutils as an argument to our module class

def get_dbutils(self, spark = None):
    try:
        if spark == None:
            spark = spark

        from pyspark.dbutils import DBUtils
        dbutils = DBUtils(spark)
    except ImportError:
        import IPython
        dbutils = IPython.get_ipython().user_ns["dbutils"]
    return dbutils
  
dbutils = get_dbutils(spark = spark)

# COMMAND ----------

#module
class calculator:

	def __init__(self, x = 10, y = 8, dbutils = None, spark = None):
		self.x = x
		self.y = y
		self.dbutils = dbutils
		self.spark = spark
			
	def print_dbutils(self):
		mysecret = self.dbutils.secrets.get(scope="myscope", key="mysecret")
	
	def use_spark(self):
		print(self.spark.range(100).count())
		
	def add(self, x = None, y = None):
		"""add function"""
		if x == None:
			x = self.x
		if y == None:
			y = self.y			
		return x+y

	def subtract(self, x = None, y = None):
		"""subtract function"""
		if x == None:
			x = self.x
		if y == None:
			y = self.y	
		return x-y

	def multiply(self, x = None, y = None):
		"""multiply function"""
		if x == None:
			x = self.x
		if y == None:
			y = self.y			
		return x*y

	def devide(self, x = None, y = None):
		"""devide function"""
		if x == None:
			x = self.x
		if y == None:
			y = self.y			
		if y == 0:
			raise ValueError('cannot devide by zero')
		else:
			return x/y
			
# a provision to execute this script if called from command line just in case, not required in our flow
if __name__ == '__main__':
    inst = calculator(x = 1, y = 2, dbutils = dbutils, spark = spark)
    inst.use_spark()
    inst.print_dbutils()

```

Things to notice:
dbutils use is limited to scopes. There are not any dbutils.notbooks.run commands or widgets being used. dbutils.notebook related commands should be kept in orchestration notebooks, not in core modules.

### 4. Test script

Corresponding test_notebook which has unittest setup. The trick is in main() of unittest class. 

```python
# Databricks notebook source
import sys
sys.path.append('/dbfs/where/you/put/your/module1/notebook/directory/')

# COMMAND ----------

##this import will be successful because we appended path.
import module1

# COMMAND ----------

import unittest
import xmlrunner
import coverage

class Testmodule1(unittest.TestCase):
  
  @classmethod
  def setUpClass(cls):
    cls.calculator_inst = customsparktest.calculator(x = 100, y = 200, spark = spark, dbutils = dbutils)

  def setUp(self):
    print("this is setup for every method")
    pass

  def test_add(self):
    self.assertEqual(self.calculator_inst.add(10,5), 15, )

  def test_subtract(self):
    self.assertEqual(self.calculator_inst.subtract(10,5), 5)
    self.assertNotEqual(self.calculator_inst.subtract(10,2), 4)

  def test_multiply(self):
    self.assertEqual(self.calculator_inst.multiply(10,5), 50)

  def tearDown(self):
    print("teardown for every method")
    pass

  @classmethod
  def tearDownClass(cls):
    print("this is teardown class")
    pass
    
  def main():    
    cov = coverage.Coverage()
    cov.start()
       
    suite =  unittest.TestLoader().loadTestsFromTestCase(Testmodule1)
    runner = xmlrunner.XMLTestRunner(output='/dbfs/testreport.xml')
    runner.run(suite)
    
    cov.stop()
    cov.save()
    cov.html_report(directory='/dbfs/covhtml')
    
```

Explaining main() of unittest: 

The objective is to generate Junit compatible xml and generate a coverage report when we call this test_notebook.

1. Junit xml
I could not find xmlrunner within unittest module which could generate Junit compatible xmls. Here I've used [xmlrunner](https://github.com/xmlrunner/unittest-xml-reporting) package which provides xmlrunner object. I am defining test suite explicitely with unittest.TestLoader() by passing the class itself. With runner and suite defined, we are triggering unittesting and generating Junit xml.

2. Coverage report
Using coverage package, we have initiated 'cov' object. We have to explicitly start and stop the execution. At the end, path for storing html report on coverage is provided.


### 5. Explanation for few decisions

We could have kept module_notebooks on workspace itself and triggered unittesting scripts. For this, we will have to use %run magic to run the module_notebook at the start of testing notebooks. I prefer to keep module notebooks on dbfs, it serves another purpose in case we have to compile a python module using setup tools. (details in another post)

The testing notebooks corresponding to different modules and one trigger notebook to invoke all testing notebooks provides independence of selecting which testing notebooks to run and which not to run. However, this increases complexity when it comes to the requirement of generating one single xml for all testing scripts and not one xml per testing script. The solution can be either extending single test suite for all test_notebooks or different test suits generating different xmls which at the end are compiled/merged with xml parser into one single xml.

In the CI/CD workflow, we will submit databricks job to run the test_notebooks individually, or we can run one trigger notebook which calls all test_notebooks.
