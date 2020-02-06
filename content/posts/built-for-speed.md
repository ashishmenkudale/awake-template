---
title: Databricks unit testing framework
subtitle: suitable for CI/CD workflows with Junit xmls, coverage reports – (Python)
category:
  - DevOps
author: Ashish Menkudale
date: January 30, 2020
featureImage: /uploads/marc-olivier-jodoin-nqoinj-ttqm-unsplash.jpg
---

Databricks has blessed Data Science community with a convenient and robust infrastructure for data analysis. Spinning up clusters, spark backbone, language interoperability, nice IDE and many more essential features.

It’s more of notebooky interface is a great alternative for conventional Jupyter users. And Data Scientists - mostly who are not orthodox python hard coders, love this interface.

## Problem statement

When it comes to productionizing models developed in databricks, these workflows in notebooks presents a little different problem for devops / build engineers. The conventional ways of unittesting python modules, generating Junit compatible xml reports, or coverage reports for product owners through command shell do not work as is in this notebook workflow. Also, data scientists working in databricks tend to use ‘dbutils’ dependencies – the databricks custom utility which provides secrets, notebook workflows, widgets etc. delighters as part of their routine model development. 

In this framework incorporating unit testing in a standard CI/CD workflow can easily become tricky.

## What this post is about

This post demonstrates a simple setup for unittesting python notebooks in databricks which is compatible for any CI/CD workflow. This is a middle ground for regular python unittest modules’ framework and databricks notebooks. 

This strategy is my personal preference. This might not be optimal solution; feedback/comments are welcome. Meanwhile, here’s how it works.

### 1. Folder structure

The first place to start is folder structure for repo. At high level, the folder structure should contain at least two folders,

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

Workspace folder will have all the notebooks, dbfs will have all the intermediate files that are to be placed on dbfs.

The notebooks in modules should be purely data science models or scripts which can be executed independently. In these notebooks, databricks’s dbutils dependency should be limited to accessing scopes and secrets. These are the notebooks, for which we will have unittesting done through notebooks in test folder.

Utilities folder can have notebooks which orchestrates execution of modules in any desired sequence. These notebooks can have dbutils.notebook.run commands. 

Tests folder will have unittesting scripts and one trigger notebook to trigger all test_Notebooks individually.

### 2. Why this folder structure

The intended CI flow, will be:

  1. Initial desired stages....
  2. Git checkout repo on devops agent
  3. Devops agent connects with databricks through cli
  	- workspace folder on workspace
	- dbfs folder on dbfs
	- workspace folder on dbfs
  4. Databricks job submit to trigger the 'Trigger' notebook which calls individual test_notebooks
  5. Read XML / coverage reports
  6. further desired stages....
  
Yes, workspace (codebase) we have kept it on dbfs as well. More specifically, we need all the notebooks in the modules on dbfs. The intention is to import notebooks in these modules as a stand alone , independent python modules inside testing notebooks to suit unittest setup.

The path where we keep our codebase on dbfs, we will append that path through sys.append.path() within testing notebook, so that, python knows from where to import these modules. (section 4, first 2 commands)

### 3.	Python Notebooks

In conventional python way, we would have a unittest framework ending with a main(), which executes bunch of tests defined within class. And through command shell, using pytest, this test script will be triggered. Also through command shell, Junit xmls can be generated and with pytest-cov extention, coverage report can be generated.

Because I prefer developing unit testing in the notebook itself, the above option of calling test scripts through command shell was invalid. pytest does not support databricks notebooks (it supports jupyter/ipython notebooks through nbval extentions)

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
dbutils use is limited to scopes. There are not any dbutils.notbooks.run commands or widgets use. dbutils.notebook should be kept in orchestration notebooks, not in core modules.

### 4. Test script

Corresponding notebook which triggers unit testing. The trick is in main() of unittest class. 

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

class Testcustomsparktest(unittest.TestCase):
  
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
       
    suite =  unittest.TestLoader().loadTestsFromTestCase(Testcustomsparktest)
    runner = xmlrunner.XMLTestRunner(output='/dbfs/testreport.xml')
    runner.run(suite)
    
    cov.stop()
    cov.save()
    cov.html_report(directory='/dbfs/covhtml')
    
```

Explaining main() of unittest: 

1. Junit xml
I could not find xmlrunner within unittest module which was generating Junit compatible xmls. There's this [xmlrunner](https://github.com/xmlrunner/unittest-xml-reporting) package I've used which provides xmlrunner. I am defining test suite explicitely with unittest.TestLoader() by passing the class itself. with runner and suite defined, we are triggering unittesting and generating Junit xml.

2. Coverage report
Using coverage.py, we have initiated cov object. We have to explicitly start and stop the execution to note the time. and at the end path for storing html report on coverage.


### 5. Explanation for few decisions

We could have kept module_notebooks on workspace itself and triggered unittesting scripts. For this, we will have to use %run magic to run the module_notebook at the start of testing notebooks. I prefer to keep module notebooks on dbfs, it serves another purpose if letting us compile a python module if required using setup tools. (details in another post)

The testing notebooks corresponding to different modules and one trigger notebook for all of them, provides independence of selecting which testing notebooks to run and which not to run. Also, this increases complexity when it comes to the requirement of generating one single xml for all testing scripts and not one xml per testing script. The solution can be either keep on extending single test suite for all test_notebooks or different test suits generating different xmls and at the end with xml parser compiling/merging different xmls into one.
