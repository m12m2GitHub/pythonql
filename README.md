# pythonql
PythonQL is an extension to Python that allows language-integrated queries against relational, XML and JSON data, as well as Python's collections.


Python has pretty advanced comprehensions that cover a large chunk of SQL, to the point where PonyORM was able to build a whole ORM system based on comprehensions. However, group by mechanisms, outerjoins and support for semi-structured data are not handled well at all.


We propose the following extensions to Python (that are implemeneted in this demo preprocessor and query executor):

 - Path expressions. When working with nested data that has varied structure, path expressions are extremely useful. We have modeled our path expression on XPath, however we use a much simplified verison:

  - Child step:  ```for x in data ./ _``` or ```for x in data ./ expr ``` where expr must evaluate to string 
  - Descendants step: ```for x in data .// _``` or ```for x in data ../ expr``` where expr must evaluate to string

Therefore we can write path expression in the query language (and elsewhere in Python expressions) like this:
```
  for x in data ./ "hotels" .// "room"
```

 - Try-except expressions. Python has try-except statement, but in many cases when working with dirty or semi-structured data, we need to be able to use an expression inside an iterator or the query. So we introduced a try-except expressions:
 
```
   try int(x) except 0 for x in values 
```

 - Tuple constructor. Tuples that have named columns are very useful in querying, however Pythons native namedtuple is not very convenient. We have extended Python's tuple constructor syntax:
 ```
   (id as employee_id, sum(x) as total_salary)
 ```

 - Query expressions:
Our query syntax is a strict superset of Pythons comprehensions, we extend the comprehensions to do much more powerful queries
than they are capable of at the time of writing.
```
 [ select (prod,len(p)) 
   for p in sales 
   let prod = p.prod 
   group by prod ]
```

At the same time our queries look similar to SQL, but are MORE FLEXIBLE and of course most of the expressions in the queries are
in pure Python. A lot of functionality is CLEANER than in SQL, like the window queries, subqueries in general, etc. 

As in Python, our query expressions can return generators, list, sets and maps.

## Documentation

A short tutorial on PythonQL is available here: https://github.com/pythonql/pythonql/wiki/PythonQL-Intro-and-Tutorial


## Examples

We have an in-progress website dedicated to various scenarios with lots of queries where PythonQL is shown to be especially useful and versatile: 

www.pythonql.org

Our website http://www.pythonql.org/ will document and code a number of scenarios where PythonQL is especially useful for solving:

Below is a small example PythonQL program.



```Python
#coding: pythonql
#
# This example illustrates the try-catch business in PythonQL.
# Basically, some data might be dirty, but you still want to be able to write a simple query

from collections import namedtuple
ord = namedtuple('Order', ['cust_id','prod_id','price'])
cust = namedtuple('Cust', ['cust_id','cust_name'])

ords = [ ord(1,1,"16.54"),
         ord(1,2,"18.95"),
         ord(1,5,"8.96"),
         ord(2,1,"????"),
         ord(2,2,"20.00") ]

custs = [ cust(1,"John"), cust(2,"Dave"), cust(3,"Boris") ]

# Basic SQL query, but with some data cleaning
res = [select (name, sum(price) as sum)
        for o in ords
        let price = try float(o.price)  except 0
        for c in custs
        where c.cust_id == o.cust_id
        group by c.cust_id as id, c.cust_name as name]

print (res)
```

## Installing pythonql:

Run ```pip install pythonql``` to install pythonql for Python2.7, or ```pip install pythonql3``` for Python 3.x. 

## Running pythonql:

PythonQL is implemented as a special encoding in a normal python script. When this encoding is specified, the
pythonql preprocessor is run, which converts the pythonql syntax to pure python.

So you should have a line in the beginning of your script:
```
#coding: pythonql

result = [ select y for x in [1,2,3] let y = x**2 ]
```

## Uninstalling pythonql:

PythonQL installs a special file in your library to enable the pythonql encoding.
If you decide to uninstall pythonql, run ```pip uninstall pythonql``` (or pythonql3) and then delete 
pythonql.pth file from your Python library.

## Help/Bugs/Suggestions:

We have a Google group running, where you can ask any questions, report bugs or suggest improvements:
https://groups.google.com/forum/#!forum/pythonql
