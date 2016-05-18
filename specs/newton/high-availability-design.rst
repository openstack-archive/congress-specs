..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================================================
Support high availability, high throughput deployment
=====================================================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/congress/+spec/high-availability-design

Some applications require Congress to be highly available (HA). Some
applications require a Congress policy engine to handle a high volume of
queries (high throughput - HT). This proposal describes how we can support
several deployment schemes that address several HA and HT requirements.


Problem description
===================

This spec aims to address three main problems:

1. Congress is not currently able to provide high query throughput because
   all queries are handled by a single, single-threaded policy engine instance.
2. Congress is not currently able to failover quickly when its policy engine
   becomes unavailable.
3. If the policy engine and a push datasource driver both crash, Congress is
   not currently able to restore the latest data state upon restart or
   failover.


Proposed change
===============

Implement the required code changes and create deployment guides for the
following reference deployments.

Warm standby for all Congress components in single process
----------------------------------------------------------

- Downtime: ~1 minute (start a new Congress instance and ingest data from
  scratch)
- Reliability: action executions may be lost during downtime.
- Performance considerations: uniprocessing query throughput
- Code changes: minimal

Active-active PE replication, DSDs warm-standby
----------------------------------------------------

Run N instances of Congress policy engine in active-active configuration. One
datasource driver per physical datasource published data on oslo-messaging to
all policy engines.

::

  +-------------------------------------+      +--------------+
  |       Load Balancer (eg. HAProxy)   | <----+ Push client  |
  +----+-------------+-------------+----+      +--------------+
       |             |             |
  PE   |        PE   |        PE   |        all+DSDs node
  +---------+   +---------+   +---------+   +-----------------+
  | +-----+ |   | +-----+ |   | +-----+ |   | +-----+ +-----+ |
  | | API | |   | | API | |   | | API | |   | | DSD | | DSD | |
  | +-----+ |   | +-----+ |   | +-----+ |   | +-----+ +-----+ |
  | +-----+ |   | +-----+ |   | +-----+ |   | +-----+ +-----+ |
  | | PE  | |   | | PE  | |   | | PE  | |   | | DSD | | DSD | |
  | +-----+ |   | +-----+ |   | +-----+ |   | +-----+ +-----+ |
  +---------+   +---------+   +---------+   +--------+--------+
       |             |             |                 |
       |             |             |                 |
       +--+----------+-------------+--------+--------+
          |                                 |
          |                                 |
  +-------+----+   +------------------------+-----------------+
  |  Oslo Msg  |   | DBs (policy, config, push data, exec log)|
  +------------+   +------------------------------------------+


- Downtime: < 1s for queries, ~2s for reactive enforcement
- Deployment considerations:

  - Cluster manager (eg. Pacemaker + Corosync) can be used to manage warm
    standby
  - Does not require global leader election
- Performance considerations:

  - Multi-process, multi-node query throughput
  - No redundant data-pulling load on datasources
  - DSDs node separate from PE, allowing high load DSDs to operate more
    smoothly and avoid affecting PE performance.
  - PE nodes are symmetric in configuration, making it easy to load balance
    evenly.

- Code changes:

  - New synchronizer and harness to support two different node types:
    API+PE node and all-DSDs node

Details
###########################################################################

- Datasource drivers (DSDs):

  - One datasource driver per physical datasource.
  - All DSDs run in a single DSE node (process)
  - Push DSDs: optionally persist data in push data DB, so a new snapshot can
    be obtained whenever needed.

- Policy engine (PE):

  - Replicate policy engine in active-active configuration.
  - Policy synchronized across PE instances via Policy DB
  - Every instance subscribes to the same data on oslo-messaging.
  - Reactive enforcement:
    All PE instances initiate reactive policy actions, but each DSD locally
    selects a "leader" to "listen to". The DSD ignores execution requests
    initiated by all other PE instances.

    - Every PE instance computes the required reactive enforcement actions and
      initiate the corresponding execution requests over oslo-messaging.
    - Each DSD locally picks PE instance as leader (say the first instance the
      DSD hears from in the asymmetric node deployment, or the PE instance on
      the same node as the DSD in a symmetric node deployment) and executes
      only requests from that PE.
    - If heartbeat contact is lost with the leader, the DSD selects a new
      leader.
    - Each PE instance is unaware of whether it is a "leader"

- API:

  - Each node has an active API service
  - Each API service routes requests for PE to its associated intranode PE
  - Requests for any other service(eg. get data source status) are routed to
    DSE2, which will be fielded by some active instance of the service on some
    node
  - Details:
    - in API models, replace every invoke_rpc with a conditional:

      - if the target is policy engine, target same-node PE
        eg.::
        self.invoke_rpc(
        caller, 'get_row_data', args, server=self.node.pe.rpc_server)

      - otherwise, invoke_rpc stays as is, routing to DSE2
        eg.::
        self.invoke_rpc(caller, 'get_row_data', args)

- Load balancer:

  - Layer 7 load balancer (e.g. HAProxy) distributes incoming API calls among
    the nodes (each running API service).
  - load balancer optionally configured to use sticky session to pin each API
    caller to a particular node. This configuration avoids the experience of
    going back in time.

- External components (load balancer, DBs, and oslo messaging bus) can be made
  highly available using standard solutions (e.g. clustered LB, Galera MySQL
  cluster, HA rabbitMQ)


Dealing with missed actions during failover
###########################################################################

When a leader fails (global or local), it takes time for the failure to be
detected and a new leader anointed. During the failover, reactive enforcement
actions expected to be triggered would be missed. Four proposed approaches
are discussed below.

- Tolerate missed actions during failover: for same applications, it may be
  acceptable to miss actions during failover.

- Re-execute recent actions after failover

  - Each PE instance remembers its recent action requests (including the
    requests a follower PE computed but did not send)
  - On failover, the DSD requests the recent action requests from the new
    leader and executes them (within a certain recency window)
  - Duplicate execution expected on failover.
- Re-execute recent unmatched actions after failover (possible future work)

  - We can reduce the number of duplicate executions on failover by attempting
    to match a new leader's recent action requests with the already executed
    requests, and only additionally executing those unmatched.
  - DSD logs all recent successfully executed action requests in DB
  - Request matching can be done by a combination of the following information:

    - the action requested
    - the timestamp of the request
    - the sequence number of the data update that triggered the action
    - the full derivation tree that triggers the action
  - Matching is imperfect, but still helpful

- Log and replay data updates (possible future work)

  - Log every data update from every data source, and let a new leader replay
    the updates where the previous leader left off to generate the needed
    action requests.
  - The logging can be directly supported by transport or by additional DB

    - kafka naturally supports this model
    - hard to do directly with oslo-messaging + RabbitMQ

- Leaderless de-duplication (possible future work)

  - If a very good matching method is implemented for re-execution for recent
    unmatched actions after failover, it is possible to go one stop further
    and simply operate in this mode full time.
  - Each incoming action request is matched against all recently executed
    action requests.

    - Discard if matched.
    - Execute if unmatched.
  - Eliminates the need for selecting leader (global or local) and improves
    failover speed

We propose to focus first on supporting the first two options
(deployers' choice). The more complex options may be implemented and supported
in future work.

Alternatives
------------

We first discuss the main decision points before detailing several alternative
deployments.

For active-active replication of the PE, here are the main decision points:

A. node configurations

  - Options:

    1. single node-type (every node has API+PE+DSDs).
    2. two node-types (API+PE nodes, all-DSDs node). [proposed target]
    3. many node-types (API+PE nodes, all DSDs in separate nodes).

  - Discussions: The many node-types configuration is most flexible and has
    the best support for high-load DSDs, but it also requires the most work to
    dev and to deploy.
    We propose to target the two node-types configuration because it gives
    reasonable support for high-load DSDs while keeping both the development
    and the deployment complexities low.

B. global vs local leader for action execution

  - Options:

    1. global leader: Pacemaker anoints a global leader among PE instances;
       only the leader sends action-execution requests.
    2. local leader: every PE instance sends action-execution requests, but
       each receiving DSD locally picks a "leader" to listen to.
       [proposed target]

  - Discussions: Because there is a single active DSD for a given data source,
    it is a natural spot to locally choose a "leader" among the PE instances
    sending reactive enforcement action execution requests.
    We propose to target the local leader style because it avoids the
    development and deployment complexities associated with global leader
    election.
    Furthermore, because all PE instances perform reactive enforcement and send
    action execution requests, the redundancy opens up the possibility for
    zero disruption to reactive enforcement when a PE intance fails.

C. DSD redundancy

  - Options:

    1. warm standby: only one set of DSDs running at a given time; backup
       instances ready to launch.
    2. hot standby: multiple instances running, but only one set is active.
    3. active-active: multiple instances active.

  - Discussions:

    - For pull DSDs, we propose to target warm standby seems most appropriate
      because warm startup time is low (seconds) relative to frequency of data
      pulls.
    - For push DSDs, warm standby is generally sufficient except for use cases
      that demand sub-second latency even during a failover. Those use cases
      would require active-active replication of the push DSDs. But even with
      active-active replication of push DSDs, other unsolved issues in
      action-execution prevent us from delivering sub-second end-to-end latency
      (push data to triggered action executed) during failover (see leaderless
      de-duplication approach for sub-second action execution failover).
      Since we cannot yet realize the benefit of active-active replication of
      push DSDs, we propose to target a warm-standby deployment, leaving
      active-active replication as potential future work.

Active-active PE replication, DSDs hot-standby, all components in one node
###########################################################################

Run N instances of single-process Congress.

One instance is selected as leader by Pacemaker. Only the leader has active
datasource drivers (which pull data, accept push data, and accept RPC calls
from the API service), but all instances subscribes to and processes data on
oslo-messaging. Queries are load balanced among instances.

::

  +-----------------------------------------------------------------------+
  |                     Load Balancer (eg. HAProxy)                       |
  +----+------------------------+------------------------+----------------+
       |                        |                        |
       |           leader       |         follower       |         follower
  +---------------------+  +---------------------+  +---------------------+
  | +-----+ +---------+ |  | +-----+ +---------+ |  | +-----+ +---------+ |
  | | API | |DSD (on) | |  | | API | |DSD (off)| |  | | API | |DSD (off)| |
  | +-----+ +---------+ |  | +-----+ +---------+ |  | +-----+ +---------+ |
  | +-----+ +---------+ |  | +-----+ +---------+ |  | +-----+ +---------+ |
  | | PE  | |DSD (on) | |  | | PE  | |DSD (off)| |  | | PE  | |DSD (off)| |
  | +-----+ +---------+ |  | +-----+ +---------+ |  | +-----+ +---------+ |
  +---------------------+  +---------------------+  +---------------------+
       |                        |                        |
       |                        |                        |
       +---------+--------------+-------------------+----+
                 |                                  |
                 |                                  |
  +--------------+-----------+ +--------------------+---------------------+
  |         Oslo Msg         | | DBs (policy, config, push data, exec log)|
  +--------------------------+ +------------------------------------------+



- Downtime: < 1s for queries, ~2s for reactive policy
- Deployment considerations:

  - Requires cluster manager (Pacemaker) and cluster messaging (Corosync)
  - Relatively simple Pacemaker deployment because every node is identical
  - Requires leader election (handled by Pacemaker+Corosync)
  - Easy to start new DSD (make API call, all instances sync via DB)
- Performance considerations:

  - [Pro] Multi-process query throughput
  - [Pro] No redundant data-pulling load on datasources
  - [Con] If some data source drivers have high demand (say telemetry data),
    performance may suffer when deployed in the same python process as other
    Congress components.
  - [Con] Because the leader has the added load of active DSDs, PE performance
    may be reduced, making it harder to evenly load balance across instances.
- Code changes:

  - Add API call to designate a node as leader or follower
  - Custom resource agent that allows Pacemaker to start, stop, promote, and
    demote Congress instances

Details
++++++++++++++

- Pull datasource drivers (pull DSD):

  - One active datasource driver per physical datasource.
  - Only leader node has active DSDs (active polling loop and active
    RPC server)
  - On node failure, new leader node activates DSDs.

- Push datasource drivers (push DSD):

  - One active datasource driver per physical datasource.
  - Only leader node has active DSDs (active RPC server)
  - On node failure, new leader node activates DSDs.
  - Persist data in push data DB, so a new snapshot can be obtained.

- Policy engine (PE):

  - Replicate policy engine in active-active configuration.
  - Policy synchronized across PE instances via Policy DB
  - Every instance subscribes to the same data on oslo-messaging.
  - Reactive enforcement: See later section "Reactive enforcement architecture
    for active-active deployments"

- API:

  - Add new API calls for designating the receiving node as leader or follower.
    The call must complete all tasks before returning (eg. start/stop RPC)
  - Each node has an active API service
  - Each API service routes requests for PE to its associated intranode PE,
    bypassing DSE2.
  - Requests for any other service(eg. get data source status) are routed to
    DSE2, which will be fielded by some active instance of the service on some
    node
  - Details:
    - in API models, replace every invoke_rpc with a conditional:

      - if the target is policy engine, target same-node PE
        eg.::
        self.invoke_rpc(
        caller, 'get_row_data', args,
        server=self.node.pe.rpc_server)

      - otherwise, invoke_rpc stays as is, routing to DSE2
        eg.::
        self.invoke_rpc(caller, 'get_row_data', args)

- Load balancer:

  - Layer 7 load balancer (e.g. HAProxy) distributes incoming API calls among
    the nodes (each running API service).
  - load balancer optionally configured to use sticky session to pin each API
    caller to a particular node. This configuration avoids the experience of
    going back in time.

- Each DseNode is monitored and managed by a cluster manager (eg. Pacemaker)
- External components (load balancer, DBs, and oslo messaging bus) can be made
  highly available using standard solutions (e.g. clustered LB, Galera MySQL
  cluster, HA rabbitMQ)

- Global leader election with Pacemaker:

  - A resource agent contains the scripts that tells a Congress instance it is
    a leader or follower.
  - Pacemaker decides which Congress instance to promote to leader (master).
  - Pacemaker promotes (demotes) the appropriate Congress instance to leader
    (follower) via the resource agent.
  - Fencing:

    - If the leader node stops responding, and a new node is promoted to
      leader, it is possible that the unresponsive node is still doing work
      (eg. listening on oslo-messaging, issuing action requests).
    - It is generally not a catastrophe if for a time there is more than one
      Congress node doing the work of a leader. (Potential effects may include:
      duplicate action requests and redundant data source pulls)
    - Pacemaker can be configured with strict fencing and STONITH for
      deployments that require it. (deployers' choice)
      http://clusterlabs.org/doc/en-US/Pacemaker/1.1/html-single/Pacemaker_Explained/#_what_is_stonith

  - In case of network partitions:

    - Pacemaker can be configured to stop each node that is not part of a
      cluster reaching quorum, or to allow each partition to continue
      operating. (deployers' choice)
      http://clusterlabs.org/doc/en-US/Pacemaker/1.0/html/Pacemaker_Explained/s-cluster-options.html

Active-active PE replication, DSDs warm-standby, each DSD in its own node
###########################################################################

Run N instances of Congress policy engine in active-active configuration. One
datasource driver per physical datasource published data on oslo-messaging to
all policy engines.

::

  +-------------------------------------+
  |       Load Balancer (eg. HAProxy)   |
  +----+-------------+-------------+----+
      |             |             |
      |             |             |
  +---------+   +---------+   +---------+
  | +-----+ |   | +-----+ |   | +-----+ |
  | | API | |   | | API | |   | | API | |
  | +-----+ |   | +-----+ |   | +-----+ |
  | +-----+ |   | +-----+ |   | +-----+ |
  | | PE  | |   | | PE  | |   | | PE  | |
  | +-----+ |   | +-----+ |   | +-----+ |
  +---------+   +---------+   +---------+
      |             |             |
      |             |             |
      +------------------------------------------+-----------------+
      |             |             |              |                 |
      |             |             |              |                 |
  +----+-------------+-------------+---+  +-------+--------+  +-----+-----+
  |              Oslo Msg              |  |  Push Data DB  |  |    DBs    |
  +----+-------------+-------------+---+  ++---------------+  +-----------+
      |             |             |       |      (DBs may be combined)
  +----------+   +----------+     +----------+
  | +------+ |   | +------+ |     | +------+ |
  | | Poll | |   | | Poll | |     | | Push | |
  | | Drv  | |   | | Drv  | |     | | Drv  | |
  | | DS 1 | |   | | DS 2 | |     | | DS 3 | |
  | +------+ |   | +------+ |     | +------+ |
  +----------+   +----------+     +----------+
      |             |                 |
  +---+--+      +---+--+          +---+--+
  |      |      |      |          |      |
  | DS 1 |      | DS 2 |          | DS 3 |
  |      |      |      |          |      |
  +------+      +------+          +------+

- Downtime: < 1s for queries, ~2s for reactive policy
- Deployment considerations:

  - Requires cluster manager (Pacemaker) and cluster messaging (Corosync)
  - More complex Pacemaker deployment because there are many different
    kinds of nodes
  - Does not require global leader election (but that's not a big saving if
    we're running Pacemaker+Corosync anyway)
- Performance considerations:

  - [Pro] Multi-process query throughput
  - [Pro] No redundant data-pulling load on datasources
  - [Pro] Each DSD can run in its own node, allowing high load DSDs to operate
    more smoothly and avoid affecting PE performance.
  - [Pro] PE nodes are symmetric in configuration, making it easy to load
    balance evenly.

- Code changes:

  - New synchronizer and harness and DB schema to support per node
    configuration

Hot standby for all Congress components in single process
###########################################################################

Run N instances of single-process Congress, as proposed in:
https://blueprints.launchpad.net/congress/+spec/basic-high-availability

A floating IP points to the primary instance which handles all queries and
requests, failing over when primary instance is down.
All instances ingest and process data to stay up to date.

::

  +---------------+
  |  Floating IP  | - - - - - - + - - - - - - - - - - - -+
  +----+----------+             |                        |
       |
       |                        |                        |
  +---------------------+  +---------------------+  +---------------------+
  | +-----+ +---------+ |  | +-----+ +---------+ |  | +-----+ +---------+ |
  | | API | |   DSD   | |  | | API | |   DSD   | |  | | API | |   DSD   | |
  | +-----+ +---------+ |  | +-----+ +---------+ |  | +-----+ +---------+ |
  | +-----+ +---------+ |  | +-----+ +---------+ |  | +-----+ +---------+ |
  | | PE  | |   DSD   | |  | | PE  | |   DSD   | |  | | PE  | |   DSD   | |
  | +-----+ +---------+ |  | +-----+ +---------+ |  | +-----+ +---------+ |
  | +-----------------+ |  | +-----------------+ |  | +-----------------+ |
  | |     Oslo Msg    | |  | |     Oslo Msg    | |  | |     Oslo Msg    | |
  | +-----------------+ |  | +-----------------+ |  | +-----------------+ |
  +---------------------+  +---------------------+  +---------------------+
             |                        |                      |
             |                        |                      |
  +----------+------------------------+----------------------+------------+
  |                             Databases                                 |
  +-----------------------------------------------------------------------+


- Downtime: < 1s for queries, ~2s for reactive policy
- Feature limitations:

  - Limited support for action execution: each action execution is triggered
    N times)
  - Limited support for push drivers: push updates only primary instance
    (optional DB-sync to non-primary instances)

- Deployment considerations:

  - Very easy to deploy. No need for cluster manager. Just start N independent
    instances of Congress (in-process messaging) and setup floating IP.
- Performance considerations:

  - Performance characteristics similar to single-instance Congress
  - [Con] uniprocessing query throughput (optional load balancer can be added
    to balance queries between instances)
  - [Con] Extra load on data sources from replicated data source drivers
    pulling same data N times
- Code changes:

  - (optional) DB-sync of pushed data to non-primary instances


Policy
------

Not applicable

Policy actions
--------------

Not applicable


Data sources
------------

Not applicable


Data model impact
-----------------

No impact


REST API impact
---------------

No impact


Security impact
---------------

No major security impact identified compared to a non-HA distributed
deployment.

Notifications impact
--------------------

No impact

Other end user impact
---------------------

Proposed changes generally transparent to end user. Some exceptions:

- Different PE instances may be out-of-sync in their data and policies
  (eventual consistency). The issue is generally made transparent to the end
  user by making each user sticky to a particular PE instance. But if
  a PE instance goes down, the end user reaches a different instance and may
  experience out-of-sync artifacts.

Performance impact
------------------

- In single node deployment, there is generally no performance impact.
- Increased latency due to network communication required by multi-node
  deployment
- Increased reactive enforcement latency if action executions are persistently
  logged to facilitate smoother failover
- PE replication can achieve greater query throughput

Other deployer impact
---------------------

- New config settings:

  - set DSE node type to one of the following:

    - PE+API node
    - a DSDs node
    - all-in-one node (backward compatible default)

  - set reactive enforcement failover strategy:

    - do not attempt to recover missed actions (backward compatible default)
    - after failover, repeat recent action requests
    - after failover, repeat recent action requests not matched to logged
      executions

- Proposed changes have no impact on existing single-node deployments.
  100% backward compatibility expected.
- Changes only have effect if deployer chooses to set up a multi-node
  deployment with the appropriate configs.

Developer impact
----------------

No major impact identified.


Implementation
==============

Assignee(s)
-----------

Work to be assigned and tracked via launchpad.


Work items
----------

Items with order dependency:

1. API routing.
   Change API routing to support active-active PE replication, routing PE-bound
   RPCs to the PE instance on the same node as the receiving API server.
   Changes expected to be confined to congress/api/*
2. Reactive enforcement.
   Change datasource_driver.py:ExecutionDriver class to handle action execution
   requests from replicated PE-nodes (locally choose leader to follow)
3. (low priority) Missed actions mitigation.

   - Implement changes to mitigate missed actions during DSD failover
   - Implement changes to mitigate missed actions during PE failover

Items without dependency:

- Push data persistence.
  Change datasource_driver.py:PushedDataSourceDriver class to support
  persistence of pushed data. Corresponding DB changes also needed.
- (potential) Leaderless de-duplication of action execution requests.
- HA guide.
  Create HA guide sketching the properties and trade-offs of each different
  deployment types.
- Deployment guide.
  Create deployment guide for active-active PE replication


Dependencies
============

- Requires Congress to support distributed deployment (for example with policy
  engine and datasource drivers on separate DseNodes.) Distributed deployment
  has been addressed by several implemented blueprints. The following patch is
  expected to be the final piece required.
  https://review.openstack.org/#/c/307693/
- This spec does not introduce any new code of library dependencies.
- In line with OpenStack recommendation
  (http://docs.openstack.org/ha-guide/controller-ha-pacemaker.html), some
  reference deployments use open source software outside of OpenStack:
  - HAProxy: http://www.haproxy.org
  - Pacemaker: http://clusterlabs.org/wiki/Pacemaker
  - Corosync: http://corosync.github.io/corosync/


Testing
=======

We propose to add tempest tests for the following scenarios:
- Up to (N - 1) PE-nodes killed
- Previously killed PE-nodes rejoin.
- Kill and restart DSDs-node, possibly at the same time PE-nodes are killed.

Split brain scenarios can be manually tested.


Documentation impact
====================

Deployment guide to be added for each supported reference deployment. No impact
on existing documentation.


References
==========

- IRC discussion on major design decisions (#topic HA design):
  http://eavesdrop.openstack.org/meetings/congressteammeeting/2016/congressteammeeting.2016-06-09-00.04.log.txt
- Notes from summit session:
  https://etherpad.openstack.org/p/newton-congress-availability
- OpenStack HA guide:
  http://docs.openstack.org/ha-guide/controller-ha-pacemaker.html
- HAProxy documentation:
  http://www.haproxy.org/#docs
- Pacemaker documentation:

  - directory: http://clusterlabs.org/doc/
  - Cluster from scratch:
    http://clusterlabs.org/doc/en-US/Pacemaker/1.1-pcs/html/Clusters_from_Scratch/index.html
  - Configuration explained:
    http://clusterlabs.org/doc/en-US/Pacemaker/1.1-pcs/html/Pacemaker_Explained/index.html

- OCF resource agents:
  http://www.linux-ha.org/wiki/OCF_Resource_Agents