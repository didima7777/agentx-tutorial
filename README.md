# AgentX Tutorial using Net-SNMP

Since no good tutorials are available I put together what I learnt. My approach was step by
step learning and that is what I describe here. Explorer should clone this code and checkout
step by step. Or an advanced explorer could skip some steps. Your choice. To help understand
what is going on, I checkin the code more often so explorer could compare what was
auto-generated and what was modification to make things work.

#### Notes
1. I used **net-snmp 5.7.3** for this tutorial.
1. All code is in flat directory structure in start. Only during the last stage I will move
things around to src and include directories to make it cleaner.
1. We need a MIB first. I created a sample mib **AGENTX-TUTORIAL-MIB** under _experimental_
branch (.1.3.6.1.3) so my MIB branch is **.1.3.6.1.3.9999**.
1. I also created *CMakeLists.txt* for anyone wanting to use cmake for compiling.

#### Step 1 - Simple MIB with two Read-Only Scalars
##### MIB
In step 1 I implement only two read-only scalars (one integer and one string) since they are
easiest to follow. MIB structure is as below.

```
vishalj@dreams:~/workspace/agentx-tutorial⟫ snmptranslate -Tp -IR agentxTutorial
+--agentxTutorial(9999)
   |
   +--myScalars(1)
   |  |
   |  +-- -R-- String    myROString(1)
   |  |        Textual Convention: DisplayString
   |  |        Size: 0..255
   |  +-- -R-- Integer32 myROInteger(2)
   |           Range: 0..2147483647
   |
   +--myConformenceGroups(99)
      |
      +--myScalarGroup(1)
```
I used smilint to ensure that mib structure is correct. You can verify it as below and it
should not give any errors. 
```
vishalj@dreams:~/workspace/agentx-tutorial⟫ smilint -l 4 -s ./AGENTX-TUTORIAL-MIB
```
##### Compiling the MIB / Generating Code
Using `mib2c` we can compile this MIB into 'C' code. Following three steps show how to 
create 'C' code, makefile and subagent code respectively.
```Shell
mib2c -c mib2c.scalar.conf agentxTutorial
mib2c -c mfd-makefile.m2m agentxTutorial
mib2c -c subagent.m2c agentxTutorial
```
Here is shell output.
```
vishalj@dreams:~/workspace/agentx-tutorial⟫ mib2c -c mib2c.scalar.conf agentxTutorial
writing to agentxTutorial.h
writing to agentxTutorial.c
running indent on agentxTutorial.c
running indent on agentxTutorial.h
vishalj@dreams:~/workspace/agentx-tutorial⟫ mib2c -c mfd-makefile.m2m agentxTutorial
writing to agentxTutorial_Makefile
vishalj@dreams:~/workspace/agentx-tutorial⟫ mib2c -c subagent.m2c agentxTutorial                                                                                                              
writing to agentxTutorial_subagent.c
running indent on agentxTutorial_subagent.c
vishalj@dreams:~/workspace/agentx-tutorial⟫ 
```
##### Checkin! (git tag: step1_scalars_autogenerated)
I checkin the code now so explorer can see what was generated by mib2c. Nothing would
compile anything yet.

##### Modify the code to make it work
1. Lets introduce two variables that we are going to monitor and name them as `myROStringVar` and `myROIntVar` to match
our MIB. I added them in subagent source file `agentxTutorial_subagent.c`.
1. Then I added an alarm handler which will update their values every 5 seconds. `myRWString` will hold the unixtime
when `myROIntVar` changed.
1. Finally in `agentxTutorial.c` I updated the placeholders to copy the values from application variables into MIB
variables.

##### Compile the code
use `make -f agentxTutorial_makefile` or use `cmake` as follows.
```
vishalj@dreams:~/workspace/agentx-tutorial⟫ mkdir build
vishalj@dreams:~/workspace/agentx-tutorial⟫ cd build/
vishalj@dreams:~/workspace/agentx-tutorial/build⟫ cmake ../
-- The C compiler identification is GNU 6.2.0
-- The CXX compiler identification is GNU 6.2.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/vishalj/workspace/agentx-tutorial/build
vishalj@dreams:~/workspace/agentx-tutorial/build⟫  make
[ 33%] Building C object CMakeFiles/agentx-tutorial.dir/agentxTutorial.c.o
[ 66%] Building C object CMakeFiles/agentx-tutorial.dir/agentxTutorial_subagent.c.o
[100%] Linking C executable agentx-tutorial
[100%] Built target agentx-tutorial
vishalj@dreams:~/workspace/agentx-tutorial/build⟫
```
##### Run the code
1. On one terminal run it as below. Press `CTRL C` to kill.
```
vishalj@dreams:~/workspace/agentx-tutorial/build⟫ ./agentx-tutorial -f
NET-SNMP version 5.7.3 AgentX subagent connected

```
2. On other terminal run following snmpwalk commands with some wait in between so you can see the values changing.
```
vishalj@dreams:~/workspace/agentx-tutorial/cmake-build-debug⟫ snmpwalk -On localhost agentxTutorial
.1.3.6.1.3.9999.1.1.0 = STRING: last changed = 1484447214
.1.3.6.1.3.9999.1.2.0 = INTEGER: 1
vishalj@dreams:~/workspace/agentx-tutorial/cmake-build-debug⟫ snmpwalk -On localhost agentxTutorial
.1.3.6.1.3.9999.1.1.0 = STRING: last changed = 1484447214
.1.3.6.1.3.9999.1.2.0 = INTEGER: 1
vishalj@dreams:~/workspace/agentx-tutorial/cmake-build-debug⟫ snmpwalk -On localhost agentxTutorial
.1.3.6.1.3.9999.1.1.0 = STRING: last changed = 1484447219
.1.3.6.1.3.9999.1.2.0 = INTEGER: 2
vishalj@dreams:~/workspace/agentx-tutorial/cmake-build-debug⟫ snmpwalk -On localhost agentxTutorial
.1.3.6.1.3.9999.1.1.0 = STRING: last changed = 1484447219
.1.3.6.1.3.9999.1.2.0 = INTEGER: 2
vishalj@dreams:~/workspace/agentx-tutorial/cmake-build-debug⟫ snmpwalk -On localhost agentxTutorial
.1.3.6.1.3.9999.1.1.0 = STRING: last changed = 1484447224
.1.3.6.1.3.9999.1.2.0 = INTEGER: 3
vishalj@dreams:~/workspace/agentx-tutorial/cmake-build-debug⟫ snmpwalk -On localhost agentxTutorial
.1.3.6.1.3.9999.1.1.0 = STRING: last changed = 1484447254
.1.3.6.1.3.9999.1.2.0 = INTEGER: 9
vishalj@dreams:~/workspace/agentx-tutorial/cmake-build-debug⟫ snmpwalk localhost agentxTutorial
AGENTX-TUTORIAL-MIB::myROString.0 = STRING: last changed = 1484447289
AGENTX-TUTORIAL-MIB::myROInteger.0 = INTEGER: 16
vishalj@dreams:~/workspace/agentx-tutorial/cmake-build-debug⟫
```
##### Checkin! (git tag: step1_scalars_implement)
I checkin the code. Thats it for step 1.

#### Step 2 - Simple Table using MIB FOR DUMMIES

##### Add table to our MIB
I created `myTestTable` as a simple table with two columns. So the new MIB structure looks like as below.
```
vishalj@dreams:~/workspace/agentx-tutorial⟫ snmptranslate -Tp -IR agentxTutorial
+--agentxTutorial(9999)
   |
   +--myScalars(1)
   |  |
   |  +-- -R-- String    myROString(1)
   |  |        Textual Convention: DisplayString
   |  |        Size: 0..255
   |  +-- -R-- Integer32 myROInteger(2)
   |           Range: 0..2147483647
   |
   +--myTables(2)
   |  |
   |  +--myTestTable(1)
   |     |
   |     +--myTestTableEntry(1)
   |        |  Index: myROStringCol1
   |        |
   |        +-- -R-- String    myROStringCol1(1)
   |        |        Textual Convention: DisplayString
   |        |        Size: 0..255
   |        +-- -R-- Integer32 myROIntCol2(2)
   |                 Range: 0..2147483647
   |
   +--myConformenceGroups(99)
      |
      +--myScalarGroup(1)
      +--myTableGroup(2)
vishalj@dreams:~/workspace/agentx-tutorial⟫
```
##### Generate code using MIB for Dummies configuration
Now to autogenerate the code using `mib2c`. I used MIB for dummies which is quite simple to implement.
```
vishalj@dreams:~/workspace/agentx-tutorial⟫ mib2c -c mib2c.mfd.conf myTestTable
Defaults for myTestTable...
writing to -
There are no defaults for myTestTable. Would you like to

  1) Accept hard-coded defaults
  2) Set defaults now [DEFAULT]

Select your choice : 1
writing to defaults/table-myTestTable.m2d
Starting MFD code generation...
writing to myTestTable.h
| +-> Processing table myTestTable
writing to defaults/node-myROIntCol2.m2d
writing to defaults/node-myROStringCol1.m2d
writing to myTestTable.c
writing to myTestTable_data_get.h
writing to myTestTable_data_get.c
| |   +-> Processing nonindex myROIntCol2
writing to myTestTable_data_set.h
writing to myTestTable_data_set.c
writing to myTestTable_oids.h
writing to myTestTable_enums.h
writing to myTestTable_interface.h
writing to myTestTable_interface.c
writing to myTestTable_data_access.h
writing to myTestTable_data_access.c
writing to myTestTable-README-FIRST.txt
writing to myTestTable-README-myTestTable.txt
running indent on myTestTable_interface.c
running indent on myTestTable.h
running indent on myTestTable_data_access.c
running indent on myTestTable_data_get.c
running indent on myTestTable_oids.h
running indent on myTestTable_enums.h
running indent on myTestTable.c
running indent on myTestTable_interface.h
running indent on myTestTable_data_set.c
running indent on myTestTable_data_access.h
running indent on myTestTable_data_set.h
running indent on myTestTable_data_get.h
vishalj@dreams:~/workspace/agentx-tutorial⟫ 
```
In case only table was to be implemented, makefile and subagent code could also be generated by following same steps as
described in step 1.
##### Checkin! (git tag: step2_tables_autogenerated)
I checkin the code now so it can be seen what was generated by `mib2c`. This new code will not compile yet because we did
not add it to makefile yet. And there are many TODOs which need to be resolved in order to load the data.

#####  Modify the code to make it work (at this point diff the files to find updated code)
1. We need to define our datastore first. I am using `/var/tmp/dummy.conf` which is supposed to contain two column data.
 First column will be a string and second an integer, e.g. `five 5`. Using this external data store will allow us to
 update it on the fly and see the changes reflect in `snmpwalk` or other similar commands.
```
three 3
four 4
two 2
one 1
zero 0
```
2. Now we add code to `myTestTable_data_access.c`.
    1. Update `myTestTable_init_data` to define what columns we are implementing. This allows us to have a table with lots
     of columns defined in the MIB but only implement a handful. This is helpful in implementing a bigger table step by
     step.
    2. Modify `myTestTable_container_load` and implement the code to read our external datastore (`/var/tmp/dummy.conf`)
     and load the values in internal datastore/container that is provided by Net-SNMP. We first create the index
     followed by copying the second column. This function will be called each time cache expires which is set to 60
     seconds by default. `myTestTable_data_access.h` includes a macro `MYTESTTABLE_CACHE_TIMEOUT` which can be tuned for
     lower or higher values. Value of 0 or less will cause loading of external datastore at each SNMP GET which could be
     initiated using `snmpwalk` or `snmpget` or `snmpbulkwalk` etc. So be cognizent of the fact that too short interval
     would cause extra burden of reading a file and may not be advisable based on application's use.
    3. Update `CMakeLists.txt` or `agentxTutorial_Makefile` whichever you are using to include all the new `myTable*`
     files.
3. Finally update `agentxTutorial_subagent.c` to initialize our Test Table `myTestTable`.

##### Compile the code
Use `make -f agentxTutorial_makefile` or use `cmake` as follows.
```
vishalj@dreams:~/workspace/agentx-tutorial/build⟫ cmake ../
-- The C compiler identification is GNU 6.2.0
-- The CXX compiler identification is GNU 6.2.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/vishalj/workspace/agentx-tutorial/build
vishalj@dreams:~/workspace/agentx-tutorial/build⟫ make
Scanning dependencies of target agentx-tutorial
[ 12%] Building C object CMakeFiles/agentx-tutorial.dir/myTestTable.c.o
[ 25%] Building C object CMakeFiles/agentx-tutorial.dir/myTestTable_data_access.c.o
[ 37%] Building C object CMakeFiles/agentx-tutorial.dir/myTestTable_data_get.c.o
[ 50%] Building C object CMakeFiles/agentx-tutorial.dir/myTestTable_data_set.c.o
[ 62%] Building C object CMakeFiles/agentx-tutorial.dir/myTestTable_interface.c.o
[ 75%] Building C object CMakeFiles/agentx-tutorial.dir/agentxTutorial.c.o
[ 87%] Building C object CMakeFiles/agentx-tutorial.dir/agentxTutorial_subagent.c.o
[100%] Linking C executable agentx-tutorial
[100%] Built target agentx-tutorial
vishalj@dreams:~/workspace/agentx-tutorial/build⟫
```
##### Run the code
1. Create/update `/var/tmp/dummy.conf` with following content.
```
three 3
four 4
two 2
one 1
zero 0
```
2. On one terminal run the compiled code as below. Press `CTRL C` to kill when done.
```
vishalj@dreams:~/workspace/agentx-tutorial/build⟫ ./agentx-tutorial -f
NET-SNMP version 5.7.3 AgentX subagent connected
```
3. On other terminal run following `snmpwalk` or `snmptable` commands.
```
vishalj@dreams:~/workspace/agentx-tutorial⟫ snmpwalk localhost myTestTable
AGENTX-TUTORIAL-MIB::myROStringCol1."one" = STRING: one
AGENTX-TUTORIAL-MIB::myROStringCol1."two" = STRING: two
AGENTX-TUTORIAL-MIB::myROStringCol1."four" = STRING: four
AGENTX-TUTORIAL-MIB::myROStringCol1."zero" = STRING: zero
AGENTX-TUTORIAL-MIB::myROStringCol1."three" = STRING: three
AGENTX-TUTORIAL-MIB::myROIntCol2."one" = INTEGER: 1
AGENTX-TUTORIAL-MIB::myROIntCol2."two" = INTEGER: 2
AGENTX-TUTORIAL-MIB::myROIntCol2."four" = INTEGER: 4
AGENTX-TUTORIAL-MIB::myROIntCol2."zero" = INTEGER: 0
AGENTX-TUTORIAL-MIB::myROIntCol2."three" = INTEGER: 3
vishalj@dreams:~/workspace/agentx-tutorial⟫ snmptable localhost myTestTable
SNMP table: AGENTX-TUTORIAL-MIB::myTestTable

 myROStringCol1 myROIntCol2
            one           1
            two           2
           four           4
           zero           0
          three           3
vishalj@dreams:~/workspace/agentx-tutorial⟫
```
4. Now update `/var/tmp/dummy.conf` with some new content or remove old content. Wait for about a minute and run snmpwalk
again to see the updates.
```
vishalj@dreams:~/workspace/agentx-tutorial⟫ snmpwalk localhost myTestTable
AGENTX-TUTORIAL-MIB::myROStringCol1."one" = STRING: one
AGENTX-TUTORIAL-MIB::myROStringCol1."four" = STRING: four
AGENTX-TUTORIAL-MIB::myROStringCol1."zero" = STRING: zero
AGENTX-TUTORIAL-MIB::myROStringCol1."three" = STRING: three
AGENTX-TUTORIAL-MIB::myROStringCol1."fiftysix" = STRING: fiftysix
AGENTX-TUTORIAL-MIB::myROStringCol1."twentytwo" = STRING: twentytwo
AGENTX-TUTORIAL-MIB::myROIntCol2."one" = INTEGER: 1
AGENTX-TUTORIAL-MIB::myROIntCol2."four" = INTEGER: 4
AGENTX-TUTORIAL-MIB::myROIntCol2."zero" = INTEGER: 0
AGENTX-TUTORIAL-MIB::myROIntCol2."three" = INTEGER: 3
AGENTX-TUTORIAL-MIB::myROIntCol2."fiftysix" = INTEGER: 56
AGENTX-TUTORIAL-MIB::myROIntCol2."twentytwo" = INTEGER: 22
vishalj@dreams:~/workspace/agentx-tutorial⟫ snmptable localhost myTestTable
SNMP table: AGENTX-TUTORIAL-MIB::myTestTable

 myROStringCol1 myROIntCol2
            one           1
           four           4
           zero           0
          three           3
       fiftysix          56
      twentytwo          22
vishalj@dreams:~/workspace/agentx-tutorial⟫
```
##### Checkin! (git tag: step2_tables_implement)
I checkin the code. Thats it for step 2.

#### Step 3 - Implementing a simple Trap
##### MIB
Now we add a trap definition (`myROIntHit`) to our MIB. MIB structure is as below. In step 1 I introduced `myROInteger` 
which is being incremented every 5 seconds in `AlrmHandler`. I will now introduce a logic that anytime `myROInteger`
modulo 5 is equal to 1, a trap is generated. e.g. when `myROInteger` equals 6, 11, 16, 21 and so on.

```
vishalj@dreams:~/workspace/agentx-tutorial⟫ snmptranslate -Tp -IR agentxTutorial
+--agentxTutorial(9999)
   |
   +--myNotifications(0)
   |  |
   |  +--myNotificationObjects(0)
   |  |  |
   |  |  +-- ---N Integer32 myTrapTime(1)
   |  |  +-- ---N String    myTrapData(2)
   |  |           Textual Convention: DisplayString
   |  |           Size: 0..255
   |  |
   |  +--myROIntHit(1)
   |
   +--myScalars(1)
   |  |
   |  +-- -R-- String    myROString(1)
   |  |        Textual Convention: DisplayString
   |  |        Size: 0..255
   |  +-- -R-- Integer32 myROInteger(2)
   |           Range: 0..2147483647
   |
   +--myTables(2)
   |  |
   |  +--myTestTable(1)
   |     |
   |     +--myTestTableEntry(1)
   |        |  Index: myROStringCol1
   |        |
   |        +-- -R-- String    myROStringCol1(1)
   |        |        Textual Convention: DisplayString
   |        |        Size: 0..255
   |        +-- -R-- Integer32 myROIntCol2(2)
   |                 Range: 0..2147483647
   |
   +--myConformenceGroups(99)
      |
      +--myScalarGroup(1)
      +--myTableGroup(2)
      +--myTrapObjectGroup(3)
      +--myTrapsGroup(4)
vishalj@dreams:~/workspace/agentx-tutorial⟫
```
##### Compiling the MIB / Generating Code
Now to autogenerate the code using `mib2c`. I am using -f flag to tell mib2c where to store the generated code othewise
it will end up overwriting `agentxTutorial.[ch]` files which we have already updated with out code.
```
vishalj@dreams:~/workspace/agentx-tutorial⟫ mib2c -c mib2c.notify.conf -f agentxTutorial_traps agentxTutorial
writing to agentxTutorial_traps.h
writing to agentxTutorial_traps.c
running indent on agentxTutorial_traps.h
running indent on agentxTutorial_traps.c
vishalj@dreams:~/workspace/agentx-tutorial⟫
```
##### Checkin! (git tag: step3_traps_autogenerated)
I checkin the code now so explorer can see what was generated by mib2c. Nothing would compile anything yet.
