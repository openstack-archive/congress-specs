
===========================================
Abstraction policy definitions in Horizon
===========================================


Congress aims to provide an extensible open-source framework for governance and regulatory compliance across any cloud services. Users can define policies in congress by pure Datalog. But Datalog, working with data-tables, is a very abstract and complex description for policies. Users may have some problems to understand and deploy policies by Datalog. So this specification describes an abstraction for policies, and mappings it to Datalog. And the Ultimate goal is to introduce a more simple and intuitive DSL (Domain Specific Language) to express policies in Congress.

Problem description
====================

Datalog is a not institutional, even difficult for users to express users’ policy who are not familiar with it. And it may cause some misinterpretation when translate the real intent to Datalog because of the complex logic. 

Proposed change
===============

Congress makes the whole cloud compliance by defining violation state and action for violation.

                Congress Policy ::= violation condition “do” action for violation 
           
So, policy abstraction is to abstract violation state and corresponding action to make the policy more intuitional and easy to use. This specification will provide an abstraction for all policies in congress, and show it in an abstraction form in Horizon.

By analyzing typical scenarios, violation state mainly can be divided into two parts. One part is the constraint of objects’ attributes, and another is the constraint of relationship between several objects’ attributions. All the objects and constraints are not just a simple set of data source tables, but they can be divided into some categories according to their functions and connected by some relations. So users just need to choose objects they care about without warring about which tables they are in. 

                violation condition ::= {condition}
                
                condition::=object attribute constraint (value | object attribute )
                
                object attribute::=object “.” attribute
        
For any violation state, congress will take some actions, such as monitoring, proactive and reactive. Though congress just takes monitoring for violation, changing cloud state to make the cloud compliance is also an important function for congress. So, policy abstraction will provide some optional reactive actions for different objects to resolve violation.

                action for violation ::= (“monitor”| “proactive”| “reactive”) data

So in the abstraction policy UI, user will input policy name, select objects, define violation condition, action and related data. Among these, element “name” defines the sign of a policy; element “objects” defines all objects which are concerned by this policy; element “violation condition” defines the expression of object’s attributes which can produce violation; element “action” defines the action needs to take for this policy; element “data” defines the information gotten or needed when execute the action. 

Based on the policy expressed by Datalog, admin will decide which parameters or actions can be exposed. Users can write their policy just through several drop-down lists or some simple inputs, and these parameters will express users’ policy. 

For the purpose of express users’ intent easily and intuitionally, our ultimate goal is to introduce a DSL in congress, which has a unified model has the ability to improve users’ communication in a specific domain. Users can express their intent policy by congress DSL easily and clearly.

Alternatives
------------

N/A

Policy
-------

There is one example to express typical policy by abstraction form in Horizon. 

Example: every network connected to a VM must either be public or owned by someone in the same group as the VM.

In this case, the state of violation is “the network connected VM is not public, and its owner is not same_group with VM’s”. When the violation happens, system will inform admin the vm’s id. So this policy can be expressed by abstraction form as bellow.

    *name:policy_1;*
    
    *objects: vm, network;*
    
    *violation conditions: network.share==no, not same_group( vm.tenant,network.tenant);*
    
    *actions: monitoring;*
    
    *data: vm.id;*
    
Policy Actions
--------------
Users can set action for a policy in field “action”. The action can be monitoring, proactive or some execute actions which can make the cloud compliance.

Data Source
-----------

N/A

Data model impact
-----------------

N/A

REST API impact
---------------

N/A

Security impact
---------------

Parameters inputted by users must in consistent with Datalog.

Notification impact
-------------------

N/A

Other end user impact
---------------------

End users can be able to write policies in Horizon and use some drop-down lists and some simple inputs to create a policy in Congress.

Performance impact
------------------

N/A

Implementation
===============

Assignee(s)
-----------

Primary assignee:
 Yali Zhang

Other contributors: 
 Jim Xu; Yinben Xia

Work items
-----------


- Abstraction form to write policies rules and actions for policies.

- Build mapping relationship between abstraction form and Datalog, and users can write a policy in UI other than Datalog.

- Pass information from Horizon to Congress to finish the policy creation.

Dependencies
============

N/A

Testing
=======

Typical scene has been tested.

Documentation impact
====================

N/A

References
==========

N/A

