
..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode
 
==========================================
Rule-X: A multi-constraint policy solver
==========================================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/congress/+spec/rule-x

Current Congress only focuses the resource verification and configuration, 
lacking the capability of multi-constraint solving. 
Rule-X aims to help congress handle complex constraints and policies to do policy decision and execution.
It works as an abstract module combining with the policy engine of Congress to 
provide solution support in multi-constraint domains. 
As connected to DSE, Rule-X gets the multi-constraint policy description and the data table as input 
and output the optimal solutions using a simple mathematical and logical resolver. 
We believe that with Rule-X as the policy solver, 
Congress will be smarter and more automatic at the services deployment.

 
Problem description
===================

Rule-X provides a multi-constraint policy solver for management decision and 
commands in Congress because the current Congress version can only do 
simple policy decision without complex constraint solving capability. For 
instance, once the policy violation is recognized, Congress could disconnect 
the VM from the offending network. Actually, another way to eliminate the 
violation is migrating the VM to other network. Unfortunately, current 
Congress version is incapable of finding a solution like that due to its lack of 
smart decision. In interactive enforcement, Congress even needs a human to 
help eliminate violation, which is inefficient for the automatic deployment 
network. Rule-X provides Congress with multi-constraint solving capability by 
giving the result of policy computation and the guides how services operate 
optimally. 

For the policies imported in Congress, multi-constraint policies are sent to 
Rule-X solver to handle and other policies like policies validation are left to the 
current policy engine.

Rule-X works on the DSE to get the policy description and the data table as 
input and output the result to policy engine after solving. The architecture of 
Rule-X in Congress is depicted in Fig.1.

The workflow of the Rule-X in Congress is as follows.

* Rule-X gets policies from API Application and constraints and network information from data source driver.

* Get optimal solutions after solving.
 
One use case relevant to multi-constraint solving is  tenants network nodes deployment in Neutron, which also needs the source data from Nova.
Policies are 

* Network nodes of different Tenant are deployed in different network. 
* Different Network nodes cannot share the same host.

Proposed change
===============

Rule-X will be implemented as a policy module that solves multi-constraint 
problems. Once a policy is input into DSE, Rule-X checks whether the policy is 
multi-constraint policy. If the policy is not the multi-constraint policy, Rule-X 
leaves it to policy engine to handle. Otherwise, Rule-X solver compiles the 
policy and constraints, and outputs the result after complex computing. 
The list of changes we have planned:
The main changes are as follows. 
* Policy classification of multi-constraint policy or not.
* A Rule-X driver implementation. This provides the driver that enables the Rule-X connects to DSE as a plug. 
* Rule-X solver implementation that supports multi-constraint solving.
* Interface for policy distribution from API application to Rule-X.
* Interface for Data source distribution from data source to Rule-X
This feature will easily integrate with Congress architecture as it is a pluggable solver. 


Alternatives
------------

None


Policy
------

The Congress datalog syntax cannot well support above mentioned multi-constraint policy.  It should be extend to including policy description. Therefore, a declarative language is introduced as the programming language, whose BNF is as follows：

class::= “class”  <className>   “{”  <class parameter > “}”

policy   ::=     “policy”  [policyName]  “{” <rule> “}”

rule     ::=     “rule”  [ruleName]  “{”  <variable> <expression>  “}”

variable  ::=     “var”   “{” <objective> “}”

objective  ::=     <objectiveName>  “:”  <className>  “;”

expression::=     “expr”   “{” <expr>  “;”  “}”

plus     ::=     <symb1>  “+”  <symb2>

intersection::=    <set1>  “&”  <set2> 

not equal  ::=    <value1>     “!=”   < value2>

equal     ::=    < value1>     “=”   < value2>

more     ::=    < value1>  “>”  < value2>

moreEqual::=    < value1>  “>=”  < value2>

less      ::=    < value1>  “<”  < value2>

lessEqual  ::=    <symb1>  “<=”  < value2>

and      ::=    <epxr1>   “&&”  <expr2>

or       ::=    <epxr1>    “||”   <expr2>

action    ::=   “action”   “{” <action>  “;”  “}”

select    ::=    “select”  < objectiveName >*

In network path selection scenario as an example, finding two disjoint paths with low delay is expressed as follows.

class Path {  

     delay: float;

     hop:int;

     bandwidth:int; 

     links:set; 

     nodes:set; 

 }
 
 policy{

      rule{

        var{a: Path; d: Path; }

         expr{d.delay+a.delay<30 && d.links&a.links=EMPTY;}

     action{select a d;}

      }       
        
} 

For tenants network nodes deployment in Neutron, which also needs service data of Nova.
Policies are 
Network nodes of different Tenant are deployed in different network. 
Different Network nodes cannot share the same host.  

class Tenant {  

    networkID;

     host;

 }

 policy{

      rule{

        var{A: Tenant; B: Tenant;  }

         expr{ A.networkID & B.networkID =EMPTY &&  A.host & B.host=EMPTY.}

      action{select A B;}

      }               

}
 It needs to be noted that the policy language of Rule-X is not finalized. Rule-X will cooperate with language expert of Congress to work it out. 



Policy actions
--------------
The action system belongs to the work of define multi-constraint policy language and should be in line with Congress datalog language. It is for further study. 


Data sources
------------
Rule-X gets data sources from data source driver with DSE.


Data model impact
-----------------
None.

REST API impact
---------------
None.

Security impact
---------------
None.

Notifications impact
--------------------
None.

Other end user impact
---------------------
None.

Performance impact
------------------
With Rule-X, Congress, as the possible controller and policy engine to OpenStack, will have more power capability of constraint solving.

Other deployer impact
---------------------
None.

Developer impact
----------------
None.

Implementation
==============
Assignee(s)
-----------
Primary assignee:

<Vincent Lee>

Other contributors:

<Oliver Huang>

<John Strassner>

<Yiyong Zha>

<Bingyi Guo>


Work items
----------
* define the interfaces of Rule-X to other components
* multi-constraint policy description and translation from datalog.

Dependencies
============
* Subscribe data source from the data source driver. 
* Require a component that can check the multi-constraint policies out of all policies.

Testing
=======
Some sample input multi-constraint policies will be created and will be handled by Rule-X. 

Documentation impact
====================
 All Rule-X details will be documented.e.

References
==========
None.
