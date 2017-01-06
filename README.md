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
Using mib2c we can compile this MIB into 'C' code. Following three steps show how to 
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