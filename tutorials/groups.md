## Distributed Coordination (Groups)

The distributed coordination service provides common services for coordination among actors that run on a
network of nodes. The distributed coordination services includes (1) group membership, (2) leader election,
(3) distributed consensus, and (4) time-synchronized coordinated action. Expected groups are specified in
the modeling file to identify the kind of group desired and the messages the groups will send. Formation of
the groups will occur in the component code created by the application developer to allow full control of
entering and exiting a defined group.

### Group Definition in Model File (.riaps)

Within the RIAPS Model file (.riaps), define the groups that components can join. Applications can have
multiple different group definitions or multiple instances of the same group definition. The
[Group](https://github.com/RIAPS/riaps-pycom/tree/master/tests/Group) application is an example of
a RIAPS application that tests the communication within groups. This example application defines a
**group** keyword with a group name of 'TheGroup' that is expected to send a messages between the joined
nodes using the defined 'Msg' message. The **group** keyword should be defined after the **message**
definitions and prior to component definitions in the model file. A group may or may not have a leader.
Groups with no leader perform simple in-group publish/subscribe messaging. This keeps the messaging overhead
low. If a leader is desired, then add **with leader** to the group definition. Time stamping can be setup to
determine when a message is sent to the group and when this message is received by the a group member by
adding a **timed** keyword to the end of the group definition.

```
Leaderless group
================
message Msg;
group TheGroup using Msg;

Group with a leader
===================
group TheGroup with leader using Msg;
```

### Group Formation and Communication

The RIAPS Framework provides an API and services to help components of one or more applications to
form logical groups during operation. The components are able to send messages to all the group members. Components not joined in the group will not see these messages.

The model file provided the definition of the available group types. One or more instances of a group type can be initiated with the component code. For the example code, the developer chose to make each group instance a one letter name and provide multiple names in each actor setup by passing in a list of instance names ("'gs='12'"). How the group instance(s) are named and determined is at the discretion of the application developer.

Components are started with an activation state which will invoke the **handleActivate** function. This is a function which should be included in the application specific component code and will be the location to call the **joinGroup** function to create the desired group instances. The 'groupName' is the name of the group defined in the model file and the developer provides an instance name ('instName'). When utilizing the message scheduling features, the group message can include a 'groupPriority' with values between 1 and 256 to indicate how to prioritize the these messages along with the other component port messages.

Syntax:
```
def handleActivate(self):
    group = joinGroup(groupName, instName, groupPriority=256)
```

To send messages to other members of a group, indicate the group instance and call **send_pyobj** with Python object as a message. The RIAPS framework will distribute the message to all other members of the group. Members receive the message in the **handleGroupMessage** function. This function must immediately call a **recv_pyobj** to obtain the group message.

Syntax:
```
g.send_pyobj(msg)
where g is the group instance name
```
```
def handleGroupMessage(self,group):
    msg = group.recv_pyobj()
```

>Note: there also send()/recv() functions to transfer a bytes object as a message.

When new members enter and leave a group instance, the existing members of the group will be notified of their arrival through the **handleMemberJoined** function and exit through the **handleMemberLeft** function. This provides a place for components to track the membership and execute application specific activities when membership changes.

Syntax:
```
handleMemberJoined(group, memberId)
handleMemberLeft(group, memberId)
```

When components are in the process of being destroyed, a **handleDeactivate** function is called. The developer should utilize this event handler to explicitly leave the group (**leaveGroup**).  

Syntax:
```
def handleDectivate():
    leaveGroup(group)
```

### Leader Elections
Once a group is formed, a leader can be elected from one of the members. Choosing a leader is a process
where a single component becomes designated as an organizer of tasks (or decision maker) among several
distributed components within a group. The election methods used is based on the [Raft](https://raft.github.io/)
election algorithm. The election process is handled by the RIAPS framework without interaction from the component code.
Once a leader is elected, there of component level event handlers available assist the application code customizing the
activity when a leader is available, reacting to requests from followers and any messaging to the followers.

The leader election is an optional feature within RIAPS groups. The feature can be enabled when the group
is created using the **with leader** keywords. If the option is turned on, the RIAPS Framework starts the
Raft leader election algorithm in the background when at least two members have joined an instance of a group.

One key element of Raft algorithm is the member state. Each component can be in one of the following states:
(1) Follower, (2) Candidate or (3) Leader. A new group member is always a *Follower*. If election is required,
then the *Followers* are promoted to *Candidates* and the voting starts. If a member collects the majority of the
votes, then its status changes to *Leader*. The *Leader* periodically sends heartbeats to the group members. If the
heartbeat stops, a new election is triggered by the framework.

There are two timeout settings configured within the RIAPS framework which control elections: (i) election timeout
and (ii) heartbeat timeout. The election timeout is the amount of time a *Follower* waits until becoming a *Candidate*
(the election timeout is randomized between a lower and upper value). The heartbeat timeout is the maximum interval
of the heartbeats by the *Leader*.

When a new election term begins and the *Follower* becomes a *Candidate*. The *Candidate* votes for itself and
sends out a request for votes to the other group members.   Once a *Candidate* has a majority of votes, it becomes
the *Leader*. All of the leader election process is handled in the background by the RIAPS framework.

All group members will receive a notification when the leader has been elected (**handleLeaderElected**) and when a
leader exits (**handleLeaderExited**). Developers can utilize these event handlers to take application specific actions. To
determine if the group has a leader, the component can call **hasLeader** where a true response indicates a leader is present.
To determine if a member is a leader, use **isLeader()** which returns true for the group leader.

Syntax:
```
handleLeaderElected(group, leaderId)
handleLeaderExited(group, leaderId)
hasLeader()
isLeader()
```

Group members can send messages to the leader using either **sendToLeader_pyobj** or **sendToLeader**. Group members following a
leader can use the **handleMessageFromLeader** event handler to receive messages from the leader. The member must immediately call
**recv**/**recv_pyobj** on the group instance to obtain the message. To determine the group instance name associated with the
message, call the **getGroupName** function.

Syntax:
```
g.sendToLeader_pyobj(msg)
where g is the group instance name
```
```
def handleMessageFromLeader(self,group):
    msg = group.recv_pyobj()
    msgGroupName = group.getGroupName()
```

Leaders of the group have additional responsibility to receive messages from group followers and send messages out to each follower,
depending on what activity is being handle (i.e. monitoring, voting, broadcasting). The **handleMessageToLeader event handler
provides the indication a message is available. As with other message handling events, must immediately call **recv** or
**recv_pyobj** to obtain the message. To determine the identity of the group, utilize the **identity** parameter of the group object.
To send a message back to the members, utilize the **sendToMember** or **sendToMember_pyobj** and provide the identity of the
member that should receive the message. If the identity is not provided, the last value is used.


Syntax:
```
g.sendToLeader_pyobj(msg)
where g is the group instance name
```
```
def handleMessageToLeader(self,group):
    msg = group.recv_pyobj()
    identity = group.identity
```
```
    group.sendToMember_pyobj(responseMsg,identity)
```

### Consensus Voting

Group members form agreement on a set of data values using a consensus vote. During consensus, N components
work to agree on a value, where the participating components are members of a group instance. A valid leader
must be present to start and complete the consensus process, otherwise an error message is returned. There
are four stages to consensus voting:

1) Request a vote: Components propose a value to the leader. This value must be considered by all
   group members and they can agree or disagree.
2) Vote request from leader: When a value is proposed to the leader, the leader forwards the proposal to the
   group members. The components get a notification about the request for a vote with the proposed value.
3) Send Vote to leader: Each group member considers the value and agrees or disagrees with it. The decisions are sent back
   to the leader from each group member.
4) Leader announces agreement: Based on the collected decisions, the leader announces if and when a consensus
   is achieved.

To start a consensus voting process, any group member can request a vote using either **requestVote** or **requestVote_pyobj**
with a topic. This message will be forwarded to the group leader. By default the voting is a majority vote, but a consensus vote can
be requested by adding 'Poll.CONSENSUS' in the requestVote call. For time sensitive votes, a 'timeout' value can be provided with
this request. A generated id string identifier will be returned for successful requests.

Syntax:
```
rfcId = g.requestVote_pyobj("some topic") # Majority vote

or

rfcId = g.requestVote_pyobj("some topic",Poll.CONSENSUS,timeoutValue) # Consensus vote
```

When the leader receives the request for a vote, messages will be sent individually to each component in the group to request them
to vote. These components will receive the requests in the **handleVoteRequest** handler. The requests include an ID of the
component that requested the vote and the group instance name. Each member must immediately call **recv**/**recv_pyobj** on the
group instance to obtain the topic of the vote. The component will then determine their vote (true or false) and send the vote back
to the leader using **sendVote**, including the ID provided for the component who originally requested the vote.

Syntax:
```
def handleVoteRequest(group, rfvId):
    msg = group.recv_pyobj()
    vote = random.uniform(0,1) > 0.51        
    self.logger.info('handleVoteRequest[%s] = %s -->  %s' % (str(rfcId),str(msg), str(vote)))
    group.sendVote(rfcId,vote)
```

The leader will gather the votes and determine if the vote passes or fails. The results of the votes will then be sent back to the
group members. The result message will be received by the **handleVoteResult** handler within each group member component.

Syntax:
```
handleVoteResult(group, rfvId, vote)
  where vote = true or false
```

### Time-synchronized Coordinated Action

Group leaders can coordinate agreement amongst the group members regarding when a time-synchronized action should be performed in
the future. Most coordinated actions require the agreement from all members to make sure the impact of the action is understood and
expected by all group members. So, the default voting type is concensus. It is possible to setup an action vote that is only
needing a majority vote and can be setup by adding 'Poll.MAJORITY' in the request for action request.

This voting process starts with a group member requesting a vote on an action and the desired time in the future to execute the
action, using either a **requestActionVote** or **requestActionVote_pyobj** call. For time sensitive votes, a 'timeout' value can
be provided with this request. A generated id string identifier will be returned for successful requests.

Syntax:
```
rfcId = g.requestActionVote_pyobj("some action", when, timeout) # Consensus vote
```

When the leader receives the request for a vote, messages will be sent individually to each component in the group to request them
to vote. These components will receive the requests in the **handleActionVoteRequest** handler. The requests include an ID of the
component that requested the vote, the group instance name and when the action should be executed. Each member must immediately
call **recv**/**recv_pyobj** on the group instance to obtain the action that the members are voting on. The component will then
determine their vote (true or false) and send the vote back to the leader using **sendVote**, including the ID provided for the
component who originally requested the vote.

Syntax:
```
def handleActionVoteRequest(group, rfvId, when):
    msg = group.recv_pyobj()
    vote = random.uniform(0,1) > 0.51        
    self.logger.info('handleActionVoteRequest[%s] = %s @ %s -->  %s' % (str(rfcId),str(msg),str(when),str(vote)))
    group.sendVote(rfcId,vote)
```

The leader will gather the votes and determine if the vote passes or fails. The results of the votes will then be sent back to the
group members. The result message will be received by the **handleVoteResult** handler within each group member component.

Syntax:
```
handleVoteResult(group, rfvId, vote)
  where vote = true or false
```
