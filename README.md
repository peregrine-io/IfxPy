

## [IfxPy](https://openinformix.github.io/IfxPy/)
-------------------------------------------------
Informix native Python driver is a high performing data access interface suitable for highly scalable enterprise and IoT solutions to works with Informix database. The **Advanced native extension module** is the heart piece of driver which is completely written in **C language** for better efficiency and performance. The **Python Database API Specification v2.0 API** has been created on top of this native layer with Python code by focusing on application API level compatibility.

The driver has support for both Python 2.7 and Python 3x. It has been certified across all major development platforms such as ARM64 running Raspbian OS, Linux, and Windows; and no surprise, it works well on the Raspberry Pi3.
 
The development activities of the driver are powered by passion, dedication and independent thinking. You may send pull request, together we grow as an open community; relevant discussion and queries are answered by community through stackoverflow. [http://stackoverflow.com/questions/tagged/informix](http://stackoverflow.com/questions/tagged/informix)  

-------------------------
### [Installing the driver](#)
```bash
pip install ifxpy
or  # if Python 3.x then
pip3 install ifxpy
```


The **pip install support** is available for **Windows 64, Linux x86_64, Arm7** for both **Python 2.7 and Python 3.4 and up**. For all other platform you may perform a local build. The driver source code is platform neutral and it is expected to build on any platform.


#### Runtime Environment
FYI: **[Informix Client SDK](http://www-01.ibm.com/support/docview.wss?uid=swg27016673) 4.10 xC2 or above** is needed for the driver to make connection to the database. Make sure Informix Client SDK is installed and its environments are set prior to running application.  
- [Download Informix Client SDK](http://www-01.ibm.com/support/docview.wss?uid=swg27016673)

Say **INFORMIXDIR** is the location where you have installed Informix Client SDK.
##### Linux
```bash
export LD_LIBRARY_PATH=${INFORMIXDIR}/lib:${INFORMIXDIR}/lib/esql:${INFORMIXDIR}/lib/cli
```
##### Windows
```bat
SET PATH=%INFORMIXDIR%\bin;%PATH%
```

-------------------------

#### [IfxPy: (Advanced Native Extension Module)](https://github.com/OpenInformix/IfxPy/wiki)
The Advanced Native Extension Module is the heart of the driver which is completely written in C language for better efficiency and performance while maintaining cross platform support.  
Please see the **[IfxPy Wiki](https://github.com/OpenInformix/IfxPy/wiki)** (work in progress) for the documentation 

#### [IfxPyDbi: Python Database API Specification v2.0 support](http://www.python.org/dev/peps/pep-0249/)
Support for the Python DB API have been created on top of the native layer by focusing on application layer compatibility; then the application source code is generally more portable across databases.


##### Known Limitation
* Large Object Support

##### Future Roadmap
* Django
* SQLAlchemy

## Build the driver from its source code
------------------
The driver source code is platform neutral; you may build the driver on any platforms. If you face any difficulty feel free to reach out to us, we are happy to help you. The following URL has instruction to build it on Windows and Linux. 

* [Windows Build](./LocalBuildWindows.md)
* [Linux Build](./LocalBuildLinux.md)
* [Run tests after the build](RunTests.md)


-----------

## Examples
Make sure to set Informix CSDK runtime environment before running the examples. 

#### Connection to the database
The basic connectivity to Informix database.

```python
import IfxPy

ConStr = "SERVER=ids0;DATABASE=db1;HOST=127.0.0.1;SERVICE=9088;UID=informix;PWD=xxxx;"

# netstat -a | findstr  9088
conn = IfxPy.connect( ConStr, "", "")

# Do some work
# -- -- -- -- --
# -- -- -- -- --
IfxPy.close(conn)
```


#### Simple Query 
The driver APIs used in this example are from the set of **Advanced native extension module APIs**   

##### FYI: IfxPy fetch functions
* IfxPy.fetch_tuple()  
Returns a tuple, indexed by column position, representing a row in a result set. The columns are 0-indexed.  

* IfxPy.fetch_assoc()  
Returns a dictionary, indexed by column name, representing a row in a result set.  

* IfxPy.fetch_both()  
Returns a dictionary, indexed by both column name and position, representing a row in a result set.  

* IfxPy.fetch_row()  
Sets the result set pointer to the next row or requested row. Use this function to iterate through a result set.  

* free_result()  
Frees resources associated with a result set  

* free_stmt()  
Frees resources associated with the indicated statement resource  


```python
# Sample1.py
import IfxPy

def my_Sample():
    ConStr = "SERVER=ids0;DATABASE=db1;HOST=127.0.0.1;SERVICE=9088;UID=informix;PWD=xxxxx;"

    try:
        # netstat -a | findstr  9088
        conn = IfxPy.connect( ConStr, "", "")
    except Exception as e:
        print ('ERROR: Connect failed')
        print ( e )
        quit()

    SetupSqlSet = [
        "create table t1 ( c1 int, c2 char(20), c3 int, c4 int ) ;",
        "insert into t1 values( 1, 'Sunday', 101, 201 );",
        "insert into t1 values( 2, 'Monday', 102, 202 );",
        "insert into t1 values( 3, 'Tuesday', 103, 203 );",
        "insert into t1 values( 4, 'Wednesday', 104, 204 );",
        "insert into t1 values( 5, 'Thursday', 105, 2005 );",
        "insert into t1 values( 6, 'Friday', 106, 206 );",
        "insert into t1 values( 7, 'Saturday', 107, 207 );"
    ]

    try:
        sql = "drop table t1;"
        print ( sql )
        stmt = IfxPy.exec_immediate(conn, sql)
    except:
        print ('FYI: drop table failed')

    i = 0
    for sql in SetupSqlSet:
        i += 1
        print (sql)
        stmt = IfxPy.exec_immediate(conn, sql)

    # The first record executed is for create table
    i -= 1

    # Select records
    sql = "SELECT * FROM t1"
    stmt = IfxPy.exec_immediate(conn, sql)
    dictionary = IfxPy.fetch_both(stmt)

    rc = 0
    while dictionary != False:
        rc += 1
        print ("--  Record {0} --".format(rc))
        print ("c1 is : ",  dictionary[0])
        print ("c2 is : ", dictionary[1])
        print ("c3 is : ", dictionary["c3"])
        print ("c4 is : ", dictionary[3])
        print (" ")
        dictionary = IfxPy.fetch_both(stmt)

    print()
    print( "Total Record Inserted {}".format(i) )
    print( "Total Record Selected {}".format(rc) )

    # Free up memory used by result and then stmt too
    IfxPy.free_result(stmt)
    IfxPy.free_stmt (stmt)

    IfxPy.close(conn)

    print ("Done")

####### Run the sample function ######
my_Sample()

```

---
#### Param Binding of Basic Data Types

```python
# Sample2.py
import IfxPy


def my_Sample():
    ConStr = "SERVER=ids0;DATABASE=db1;HOST=127.0.0.1;SERVICE=9088;UID=informix;PWD=xxxxx;"

    try:
        # netstat -a | findstr  9088
        conn = IfxPy.connect( ConStr, "", "")
    except Exception as e:
        print ('ERROR: Connect failed')
        print ( e )
        quit()


    try:
        sql = "drop table t1;"
        print ( sql )
        stmt = IfxPy.exec_immediate(conn, sql)
    except:
        print ('FYI: drop table failed')

    sql = "create table t1 ( c1 int, c2 char(20), c3 int, c4 int ) ;"
    stmt = IfxPy.exec_immediate(conn, sql)


    sql = "INSERT INTO t1 (c1, c2, c3, c4) VALUES ( ?, ?, ?, ? )"
    stmt = IfxPy.prepare(conn, sql)

    c1 = None
    c2 = None
    c3 = None
    c4 = None
    # Create bindings for the parameter
    IfxPy.bind_param(stmt, 1, c1, IfxPy.SQL_PARAM_INPUT, IfxPy.SQL_INTEGER)
    IfxPy.bind_param(stmt, 2, c2, IfxPy.SQL_PARAM_INPUT, IfxPy.SQL_CHAR)
    IfxPy.bind_param(stmt, 3, c1, IfxPy.SQL_PARAM_INPUT, IfxPy.SQL_INTEGER)
    IfxPy.bind_param(stmt, 4, c1, IfxPy.SQL_PARAM_INPUT, IfxPy.SQL_INTEGER)

    print("Inserting Recors ......")
    i = 0
    while i < 10:
        i += 1
        c1 = 100 + i
        c2 = "Testing {0}".format(i)
        c3 = 20000 + i
        c4 = 50000 + i
        # supply new values as a tuple
        IfxPy.execute(stmt, (c1, c2, c3, c4) )


    # Try select those rows we have just inserted
    print("Selecting Recors ......")
    sql = "SELECT * FROM t1"
    stmt = IfxPy.exec_immediate(conn, sql)
    tu = IfxPy.fetch_tuple(stmt)
    rc = 0
    while tu != False:
        rc += 1
        print( tu )
        tu = IfxPy.fetch_tuple(stmt)

    print()
    print( "Total Record Inserted {}".format(i) )
    print( "Total Record Selected {}".format(rc) )

    # Free up memory used by result and then stmt too
    IfxPy.free_result(stmt)
    IfxPy.free_stmt (stmt)

    # close the connection
    IfxPy.close(conn)

    print ("Done")

####### Run the sample function ######
my_Sample()


```

---
#### Param Binding of Collection and User Defined Data Types

```python
# Sample3.py
import IfxPy


def my_Sample():
    ConStr = "SERVER=ids0;DATABASE=db1;HOST=127.0.0.1;SERVICE=9088;UID=informix;PWD=xxxxx;"

    try:
        # netstat -a | findstr  9088
        conn = IfxPy.connect( ConStr, "", "")
    except Exception as e:
        print ('ERROR: Connect failed')
        print ( e )
        quit()
    
    try:
        sql = "drop table rc_create;"
        print ( sql )
        stmt = IfxPy.exec_immediate(conn, sql)
    except:
        print ('FYI: drop table failed')

    sql = "DROP ROW TYPE if exists details RESTRICT;"
    IfxPy.exec_immediate(conn, sql)

    sql = "DROP ROW TYPE if exists udt_t1 RESTRICT;"
    IfxPy.exec_immediate(conn, sql)

    sql = "CREATE ROW TYPE details(name varchar(15), addr varchar(15), zip varchar(15) );"
    stmt = IfxPy.exec_immediate(conn, sql)

    sql = "CREATE ROW TYPE udt_t1(name varchar(20), zip int);"
    stmt = IfxPy.exec_immediate(conn, sql)

    sql = "CREATE TABLE udt_tab1 (c1 int, c2 SET(CHAR(10)NOT NULL), c3 MULTISET(int not null), c4 LIST(int not null), c5 details, c6 udt_t1 );"
    stmt = IfxPy.exec_immediate(conn, sql)

    sql = "INSERT INTO udt_tab1 VALUES (?, ?, ?, ?, ?, ?);"
    stmt = IfxPy.prepare(conn, sql)

    c1 = None
    c2 = None
    c3 = None
    c4 = None
    c5 = None
    c6 = None

    print("Bind the params");
    IfxPy.bind_param(stmt, 1, c1, IfxPy.SQL_PARAM_INPUT, IfxPy.SQL_INTEGER)
    IfxPy.bind_param(stmt, 2, c2, IfxPy.SQL_PARAM_INPUT, IfxPy.SQL_CHAR,  IfxPy.SQL_INFX_RC_COLLECTION)
    IfxPy.bind_param(stmt, 3, c3, IfxPy.SQL_PARAM_INPUT, IfxPy.SQL_CHAR, IfxPy.SQL_INFX_RC_COLLECTION)
    IfxPy.bind_param(stmt, 4, c4, IfxPy.SQL_PARAM_INPUT, IfxPy.SQL_CHAR, IfxPy.SQL_INFX_RC_COLLECTION)
    IfxPy.bind_param(stmt, 5, c5, IfxPy.SQL_PARAM_INPUT, IfxPy.SQL_CHAR, IfxPy.SQL_INFX_UDT_FIXED)
    IfxPy.bind_param(stmt, 6, c6, IfxPy.SQL_PARAM_INPUT, IfxPy.SQL_CHAR, IfxPy.SQL_INFX_UDT_VARYING)
    i = 0
    while i < 3:
        i += 1
        c1 = 100+i
        c2 = "SET{'test', 'test1'}"
        c3 = "MULTISET{1,2,3}"
        c4 = "LIST{10, 20}"
        c5 = "ROW('Pune', 'City', '411061')"
        c6 = "ROW('Mumbai', 11111)"
        IfxPy.execute(stmt, (c1, c2, c3, c4, c5, c6));


    print("Selecting Records ......")
    sql = "SELECT * FROM udt_tab1"
    stmt = IfxPy.exec_immediate(conn, sql)
    tu = IfxPy.fetch_tuple(stmt)
    rc = 0
    while tu != False:
        rc += 1
        print( tu)
        tu = IfxPy.fetch_tuple(stmt)

    print()
    print( "Total Record Inserted {}".format(i) )
    print( "Total Record Selected {}".format(rc) )

    # Free up memory used by result and then stmt too
    IfxPy.free_result(stmt)
    IfxPy.free_stmt (stmt)

    # close the connection
    IfxPy.close(conn)

    print ("Done")

####### Run the sample function ######
my_Sample()


```

---
#### Sample program for Smart Trigger support in Informix Python Driver

```python
# test_SmartTrigger.py
import IfxPy
from ctypes import *
import threading
import os

def printme1(outValue1):
   "This prints a passed string into this function1"
   print ("\nTest for callback function, value = ", outValue1)
   return

def printme2(outValue2):
   "This prints a passed string into this function2"
   print ("\nTest for callback function, value = ", outValue2)
   return
 
def task1():
    print("Task 1 assigned to thread: {}".format(threading.current_thread().name))
    print("ID of process running task 1: {}".format(os.getpid()))
    temp4 = IfxPy.register_smart_trigger_loop(conn, printme1, temp, "t1", "informix", "sheshdb", "select * from t1;", "label1", False, False)
    #temp4 = IfxPy.register_smart_trigger_loop(conn, printme1, temp, "बस", "informix", "sheshdb_utf8", "select * from बस;", "label1", False, False)
 
def task2():
    print("Task 2 assigned to thread: {}".format(threading.current_thread().name))
    print("ID of process running task 2: {}".format(os.getpid()))
    #temp5 = IfxPy.register_smart_trigger_loop(conn, printme2, temp, "t2", "informix", "sheshdb", "select * from t2;", "label2", False, False)
    temp5 = IfxPy.register_smart_trigger_loop(conn, printme2, temp, "t2", "informix", "sheshdb_utf8", "select * from t2;", "label2", False, False)
 
# if __name__ == "__main__":
             
# ConStr = "SERVER=ol_informix1410;DATABASE=sheshdb_utf8;HOST=127.0.0.1;SERVICE=1067;UID=informix;PWD=xxxx;DB_LOCALE=en_us.utf8;CLIENT_LOCALE=en_us.UTF8;"
ConStr = "SERVER=ol_informix1410;DATABASE=sheshdb;HOST=127.0.0.1;SERVICE=1067;UID=informix;PWD=xxx;"

conn = IfxPy.connect( ConStr, "", "")

temp = IfxPy.open_smart_trigger(conn, "Unique1", False, 5, 1, 0)
print ("\nFile descriptor = ", temp)
 
# creating threads
t1 = threading.Thread(target=task1, name='t1')
t2 = threading.Thread(target=task2, name='t2')  
 
# starting threads
t1.start()
t2.start()
 
# wait until all threads finish
t1.join()
t2.join()

IfxPy.close(conn)
print ("Done")


```


### [Python Database API Specification v2.0](http://www.python.org/dev/peps/pep-0249/)
-------------------------------------
The Advanced Native Extension Module is the heart of the driver which is completely written in C language for better efficiency and performance. The Python Database API Specification v2.0 APIs have been created on top of this native layer by focusing on application layer compatibility. Then the application source code is generally more portable across databases.


#### Example by using Python DB API
The driver APIs used in this example are from the set of **Python Database API Specification v2.0 APIs**

```python

import IfxPyDbi as dbapi2

ConStr = "SERVER=ids0;DATABASE=db1;HOST=127.0.0.1;SERVICE=9088;UID=informix;PWD=xxxx;"

conn = dbapi2.connect( ConStr, "", "")
cur = conn.cursor()

try:
    stmt = cur.execute('drop table t1;')
except:
    print ('FYI: drop table failed')

cur.execute('create table t1 ( c1 int, c2 char(20), c3 int, c4 int ) ')

cur.execute("insert into t1 values( 1, 'Sunday', 101, 201 )")
cur.execute("insert into t1 values( 2, 'Monday', 102, 202 )")
cur.execute("insert into t1 values( 3, 'Tuesday', 103, 203 )")
cur.execute("insert into t1 values( 4, 'Wednesday', 104, 204 )")
cur.execute("insert into t1 values( 5, 'Thursday', 105, 2005 )")
cur.execute("insert into t1 values( 6, 'Friday', 106, 206 )")
cur.execute("insert into t1 values( 7, 'Saturday', 107, 207 )")

conn.commit ()

cur.execute ("SELECT * FROM t1")
rows = cur.fetchall()
for i, row in enumerate(rows):
    print ("Row", i, "value = ", row)

cur.close()

conn.close()
print ("Done")

```

#### DB API Example: inserts many records at a time
See the working sample of this module **DbAPI_Sample_executemany.py** in the Examples folder.  

```python
#-- -- -- -- --
#-- -- -- -- --
cur.execute('create table t1 ( c1 int, c2 char(20), c3 int, c4 int ) ')
days = [ (1, 'Sunday',  101, 201),
         (2, 'Monday',  102, 202),
         (3, 'Tuesday', 103, 203),
         (4, 'Wednesday', 104, 204),
         (5, 'Thursday', 105, 205),
         (6, 'Friday', 106, 206),
         (7, 'Saturday', 1027, 207),
       ]
cur.executemany('INSERT INTO t1 VALUES (?,?,?,?)', days)
conn.commit ()
#-- -- -- -- --
#-- -- -- -- --
```

#### DB API Example: params
See the working sample of this module **DbAPI_Sample_Params.py** in the Examples folder.  
```python
#-- -- -- -- --
#-- -- -- -- --
c1_val = 4
cur.execute("UPDATE t1 SET c2 = 'Wednesday!' WHERE c1 = (?)", (c1_val,))
conn.commit()

c3_val = 1004
cur.execute("UPDATE t1 SET c3 = (?) WHERE c1 = 4", (c3_val,))
conn.commit()

cur.execute("select * FROM t1 WHERE c1 = ?", (c1_val,))
row = cur.fetchone()
print ( "value = ", row)
#-- -- -- -- --
#-- -- -- -- --
```


### [Advanced native extension module APIs](https://github.com/OpenInformix/IfxPy/wiki)
-------------------------------------------
* IfxPy.connect:  
Connect to the database.  

* IfxPy.exec_immediate:  
Prepares and executes an SQL statement.  

* IfxPy.prepare:  
Prepares an SQL statement.    

* IfxPy.bind_param:  
Binds a Python variable to an SQL statement parameter.  

* IfxPy.execute:  
Executes an SQL statement that was prepared by * IfxPy.prepare().  

* IfxPy.fetch_tuple:  
Returns an tuple  

* IfxPy.fetch_assoc:  
Returns a dictionary  

* IfxPy.fetch_both:  
Returns a dictionary  

* IfxPy.fetch_row:  
Sets the result set pointer to the next row or requested row.  

* IfxPy.result:  
Returns a single column from a row in the result set.  

* IfxPy.active:  
Checks if the specified connection resource is active.  

* IfxPy.autocommit:  
Returns or sets the AUTOCOMMIT state for a database connection.  

* IfxPy.callproc:  
Returns a tuple containing OUT/INOUT variable value.  

* IfxPy.check_function_support:  
return true if fuction is supported otherwise return false.  

* IfxPy.close:  
Close a database connection.  

* IfxPy.conn_error:  
Returns a string containing the SQLSTATE returned by the last connection attempt.  

* IfxPy.conn_errormsg:  
Returns an error message and SQLCODE value representing the reason the last database connection attempt failed. 

* IfxPy.conn_warn:  
Returns a warning string containing the SQLSTATE returned by the last connection attempt.  

* IfxPy.client_info:  
Returns a read-only object with information about the IDS database client.  

* IfxPy.column_privileges:  
Returns a result set listing the columns and associated privileges for a table.  

* IfxPy.columns:  
Returns a result set listing the columns and associated metadata for a table.  

* IfxPy.commit:  
Commits a transaction.  

* IfxPy.cursor_type:  
Returns the cursor type used by a statement resource.  

* IfxPy.execute_many:  
Execute SQL with multiple rows.

* IfxPy.field_display_size:  
Returns the maximum number of bytes required to display a column.  

* IfxPy.field_name:  
Returns the name of the column in the result set.  

* IfxPy.field_nullable:  
Returns indicated column can contain nulls or not.  

* IfxPy.field_num:  
Returns the position of the named column in a result set.  

* IfxPy.field_precision:  
Returns the precision of the indicated column in a result set.  

* IfxPy.field_scale:  
Returns the scale of the indicated column in a result set.  

* IfxPy.field_type:  
Returns the data type of the indicated column in a result set.  

* IfxPy.field_width:  
Returns the width of the indicated column in a result set.  

* IfxPy.foreign_keys:  
Returns a result set listing the foreign keys for a table.  

* IfxPy.free_result:  
Frees resources associated with a result set.  

* IfxPy.free_stmt:  
Frees resources associated with the indicated statement resource.  

* IfxPy.get_option:  
Gets the specified option in the resource.  

* IfxPy.num_fields:  
Returns the number of fields contained in a result set.  

* IfxPy.num_rows:  
Returns the number of rows affected by an SQL statement.  

* IfxPy.get_num_result:  
Returns the number of rows in a current open non-dynamic scrollable cursor.  

* IfxPy.primary_keys:  
Returns a result set listing primary keys for a table.  

* IfxPy.procedure_columns:  
Returns a result set listing the parameters for one or more stored procedures.  

* IfxPy.procedures:  
Returns a result set listing the stored procedures registered in a database.  

* IfxPy.rollback:  Rolls back a transaction.  

* IfxPy.server_info:  
Returns an object with properties that describe the IDS database server.  

* IfxPy.get_db_info:  
Returns an object with properties that describe the IDS database server according to the option passed.  

* IfxPy.set_option:  
Sets the specified option in the resource.  

* IfxPy.special_columns:  
Returns a result set listing the unique row identifier columns for a table.  

* IfxPy.statistics:  
Returns a result set listing the index and statistics for a table.  

* IfxPy.stmt_error:  
Returns a string containing the SQLSTATE returned by an SQL statement.  

* IfxPy.stmt_warn:  
Returns a warning string containing the SQLSTATE returned by last SQL statement.  

* IfxPy.stmt_errormsg:  
Returns a string containing the last SQL statement error message.  

* IfxPy.table_privileges:  
Returns a result set listing the tables and associated privileges in a database. 

* IfxPy.tables:  
Returns a result set listing the tables and associated metadata in a database.  

* IfxPy.get_last_serial_value:  
Returns last serial value inserted for identity column.

* IfxPy.open_smart_trigger:  
Open Smart Trigger session

* IfxPy.get_smart_trigger_session_id:  
Get already opened Smart Trigger session ID

* IfxPy.join_smart_trigger_session:  
Join already opened Smart Trigger session ID

* IfxPy.register_smart_trigger_loop:  
Open Smart Trigger session with loop handled by Informix Python driver

* IfxPy.register_smart_trigger_no_loop:  
Open Smart Trigger session with loop handled by Informix Python application

* IfxPy.delete_smart_trigger_session:  
Delete opened Smart Trigger session
