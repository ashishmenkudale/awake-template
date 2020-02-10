---
title: Python - R interoperability in Azure HDInsights with Reticulate
subtitle: Running pyspark scripts through R
category:
  - DevOps
author: Ashish Menkudale
date: 2020-03-01T19:59:59.000Z
featureImage: /uploads/marc-olivier-jodoin-nqoinj-ttqm-unsplash.jpg
---

When it comes to favorite language for Data Science, both python and R communities present strong arguments. Overall, python has upperhand in infrastructure setup while R has upper hand in core statistical functionality. Although things have changed a lot recently.

If big data is in the picture, pyspark blows the compitition out of the park. SparkR and SparklyR do not have comparable functionality for data manipulation. If only there could be a solution where a project can take advantage of infrastructure setup in python/pyspark and data science models developed in R, wouldn't it be ideal?

[Reticulate](https://rstudio.github.io/reticulate/) lets us use python through R while [rpy2](https://rpy2.readthedocs.io/en/latest/index.html) lets us access R functionality through python. 

## Problem statement

For one of the big data project, I had to setup an R - python interoperable framework in Azure Hdinsights platform. The objective was to use python / pyspark for faster data query , and data transformations. And once the data ETL was complete, the core algorithms in R were to be executed. Reticulate installation and setup is easy. The challange arises when it comes down to using reticulate to run python scripts which had pyspark / hive context dependency.

## What this post is about

Right of the bat, there were issues as pyspark and hadoop configuration on HDInsights cluster is in python 2. And the framework, I wanted to setup was in python 3. Things started to fail when I was trying to executing pyspark script. This post is about the core issues and how to configure HDInsights cluster to make it suitable for python3 - R interoperability.

### 1. Reticulate setup

Installing reticulate on Rserver running on Hdinsights was easy. Through developer level access for the cluster, package can be installed through bash script with sudo su command. Note: if we do ```install.packages('Reticulate')``` the package will be installed locally for the user. The package needs to be installed at root level.

Once package s installed, we can verify it with simply, 

```R
library(reticulate)
reticulate::py_config()
```
Output:
```
python:         /usr/bin/python
libpython:      /usr/lib/python2.7/config-x86_64-linux-gnu/libpython2.7.so
pythonhome:     /usr:/usr
version:        2.7.12 (default, Dec  4 2017, 14:50:18)  [GCC 5.4.0 20160609]
numpy:           [NOT FOUND]

python versions found: 
 /usr/bin/python
 /usr/bin/python3
 ```


### 2. Calling simple python3 scripts through Reticulate

Here's wow reticulate can be used in simple cases,

python script called calc.py

```python
class calculator:

	def __init__(self, x = 10, y = 8):
		self.x = x
		self.y = y
			
	def add(self, x = None, y = None):
		"""add function"""
		if x == None:
			x = self.x
		if y == None:
			y = self.y
		
		print("this is hello from python")	
		return x + y
```

we will call this script in R

```R
library(reticulate)
reticulate::source_python('calc.py')
 
inst = calculator(x = 14, y = 16)
inst$add()

#Output
#this is hello from python
#[1] 30

```


### 3.	When pyspark is in picture

Everything so far works fine. However, when python script requires spark dependency, issues start.



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
