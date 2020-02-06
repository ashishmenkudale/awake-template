---
title: Databricks unit testing framework suitable for CI/CD workflows with Junit xmls, coverage reports – (Python) 
subtitle: Awake is Built to Be Blazing Fast
category:
  - DevOps
author: Ashish Menkudale
date: 2020-01-31T04:27:56.800Z
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

### Folder structure

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

### Why this folder structure

The intended CI flow, will be:

  1. Initial desired stages....
  2. Git checkout repo on devops agent
  3. Devops agent to databricks (workspace folder on workspace, dbfs folder on dbfs) through cli
  4. Databricks job submit to trigger the 'Trigger' notebook which calls individual test_notebooks
  5. Read XML / coverage reports
  6. further desired stages....

### 2.	Python Notebooks

This part is little tricky, expecially in main() of unittest class. 

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
dbutils use is limited to scopes. There are not any dbutils.notbooks.run commands or widgets use.

### Test script

Corresponding notebook which triggers unit testing

```python
# Databricks notebook source
import sys
sys.path.append('/dbfs/where/you/put/your/module1/notebook/directory/')

# COMMAND ----------

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
    #runner = unittest.TextTestRunner()
    runner = xmlrunner.XMLTestRunner(output='/dbfs/testreport.xml')
    runner.run(suite)
    
    cov.stop()
    cov.save()
    cov.html_report(directory='/dbfs/covhtml')
    
```

## Pretty Stinkin' Fast, I'd Say

I've taken a number of steps to try and make Awake as fast and snappy as possible for the end user and I think you'll find it's been handled fairly well. Last I ran one of the posts through Page Speed Insights I got a 99 score for desktop and 89 for mobile. [Give it a try for yourself!](https://developers.google.com/speed/pagespeed/insights/?url=https%3A%2F%2Fawake-template.netlify.com%2Fpost-markup-and-formatting%2F&tab=desktop)

![Page speed insights score 99!!](/uploads/page-speed-insights.jpg)
