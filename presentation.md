%title: Consensus Systems - What are they ?
%author: Markus Burger
%date: 2019-11-27

-> # What are we even talking about here ? <-

* We will cover a *lot* of theory
* We will try to map these concepts to systems that we actually use :D
* Hopefully we can establish a little more background knowlege of how things actually operate


Talking in terms of the infra we have here at jumio can anybody give me an example of what systems we might even be talking about ?

--------------------------------------------------
-> # Let's dive right in shall we <3 ? <-

< Consistency Models >
        \\   ^__^
         \\  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||


dammit magic cow it's not your time yet -.-
but okay :D since you bring up the topic does somebody here have a hunch what we might be talking about ?

-------------------------------------------------
-> # Consistency Models  <-


Note: I'm only gonna list models here that i think i can talk about without sounding like a complete dork :D

* Read Uncommited
* Read Commited
* Serializable
* Lineralizable
* Strict Serializable

But to say it with the voice of Billy Mays ^^

< But wait there's more >
        \\   ^__^
         \\  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

-------------------------------------------------

-> # Read Uncommited  <-

Ever sat down with a hot cup of coffee, your thoughts floating off to the edge of existance and you are thinking to yourself:

*"man i have a database and all is well but wouldnt it be awsome if i could just have no contrains at all? FREE THE B.. ÄHM DATA :D"*

then this is the right level for you :D all bets are off and i mean ALL of them
* no locking
* no real-time contrains
* no per-process ordering of transactions
* non-repeatable reads

the only thing it in practise is implemented is that dirty writes are still not possible, at least that ^^



-------------------------------------------------

-> # Read Commited  <-

It's like Read Uncommiteds big brother ^^ more mature and a little smarter about things, it's not much but at least something ^^

Read committed is a consistency model which strengthens read uncommitted committed by preventing dirty reads:
transactions are not allowed to observe writes from transactions which do not commit.

sooooo what does that mean ??

-------------------------------------------------

-> # Read Commited  <-

it means the following, you dont have dirty reads anymore :D yeah, so we can go home right ?

< But wait there's more >
        \\   ^__^
         \\  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

thank you magic cow -.- another day ruined
soooo what would you guys think is the difference between Read Uncommited and Commited ?

-------------------------------------------------

-> # Serializable  <-

Serializable sounds cool and fancy now this must be the nail in the coffin of all those pesky problems, no?
let's take a look at the definition ^^

Informally, serializability means that transactions appear to have occurred in some total order.

Serializability is a transactional model: operations (usually termed "transactions") can involve several primitive sub-operations performed in order.
Serializability guarantees that operations take place atomically:
a transaction’s sub-operations do not appear to interleave with sub-operations from other transactions.

It is also a multi-object property: operations can act on multiple objects in the system.
Indeed, serializability applies not only to the particular objects involved in a transaction,
but to the system as a whole—operations may act on predicates, like “the set of all cats”.


YEAH :D :D :D !!!!!
wait ...... i can already smell this f***ing magic cow around the corner again

-------------------------------------------------

-> # Serializable  <-

Serializability implies repeatable read, snapshot isolation, etc.
However, it does not impose any real-time, or even per-process constraints.
If process A completes write w, then process B begins a read r, r is not necessarily guaranteed to observe w.
For those kinds of real-time guarantees, see strict serializable.

Moreover, serializability does not require a per-process order between transactions.
A process can observe a write, then fail to observe that same write in a subsequent transaction.
In fact, a process can fail to observe its own prior writes, if those writes occurred in different transactions.

*sigh* nothing is perfect it seems

-------------------------------------------------

-> # Lineralizable  <-

Linearizability is one of the strongest single-object consistency models, and implies that every operation appears to take place atomically,
in some order, consistent with the real-time ordering of those operations:
e.g., if operation A completes before operation B begins, then B should logically take effect after A.

Linearizability is a single-object model, but the scope of “an object” varies.
Some systems provide linearizability on individual keys in a key-value store;
others might provide linearizable operations on multiple keys in a table,
or multiple tables in a database—but not between different tables or databases, respectively.

sooo what does this mean ?? i'm lost

-------------------------------------------------

-> # Strict Serializable  <-

Informally, strict serializability (a.k.a. PL-SS, Strict 1SR, Strong 1SR) means that operations appear to have occurred in some order,
consistent with the real-time ordering of those operations; e.g. if operation A completes before operation B begins, then A should appear to precede B in the serialization order.

Strict serializability implies serializability and linearizability.
You can think of strict serializability as serializability’s total order of transactional multi-object operations,
plus linearizability’s real-time constraints.
Alternatively, you can think of a strict serializable database as a linearizable object in which the object’s state is the entire database.

-------------------------------------------------

-> # WTF???  <-

and can anybody tell me here what of the above models apply to the following systems and why:
* Postgresql
* Zookeeper
* Consul

-------------------------------------------------

-> # Consensus Systems  <-

yeah we are finally there :D :D :D

fundamentally, the goal of consensus is not that of the negotiation of an optimal value of some kind,
but just the collective agreement on some value that was previously proposed by one of the participating servers in that round of the consensus algorithm.

-------------------------------------------------

-> # Raft  <-

< raft raft raft raft >
        \\   ^__^
         \\  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||


yes magic cow .... i know .... back to theory

-------------------------------------------------

-> # Raft  <-

hold on a little bit ... what even is consensus?

consensus is a fundamental problem in fault-tolerant distributed systems.
consensus involves multiple servers agreeing on values. once they reach a decision on a value, that decision is final.
typical consensus algorithms make progress when any majority of their servers is available;
for example, a cluster of 5 servers can continue to operate even if 2 servers fail. If more servers fail,
they stop making progress (but will never return an incorrect result).


consensus typically arises in the context of replicated state machines, a general approach to building fault-tolerant systems.
each server has a state machine and a log.
the state machine is the component that we want to make fault-tolerant, such as a hash table.
it will appear to clients that they are interacting with a single, reliable state machine, even if a minority of the servers in the cluster fail.
each state machine takes as input commands from its log.
in our hash table example, the log would include commands like set x to 3. A consensus algorithm is used to agree on the commands in the servers' logs.
the consensus algorithm must ensure that if any state machine applies set x to 3 as the nth command, no other state machine will ever apply a different nth command.
as a result, each state machine processes the same series of commands and thus produces the same series of results and arrives at the same series of states.

-------------------------------------------------

-> # Raft  <-

the raft protocol is based on the same set of assumptions as classic multi paxos:

* asynchronicity: no upper bound on message delays or computation times and no global clock synchronization.
* unreliable Links: possibility of indefinite networks delays, packet loss, partitions, reordering, or duplication of messages.
* fail crash failure model: processes may crash, may eventually recover, and in that case rejoin the protocol, byzantine failures cannot occur.

strong leadership extends classical leader-driven consensus by adding the following constraints:

* All message passing can only be initialized by the leader or a server attempting to become leader.
  This is enforced in the protocols specification through the modeling of all communications as RPCs, implicitly encoding a server’s role as either active or passive.
* The leader takes care of all communication with clients of the application and needs to be contacted directly by them.
* The system is only available when a leader has been elected and is alive. Otherwise, a new leader will be elected and the system will remain unavailable for the duration of the vote.

safety in the commitment step is guaranteed by only allowing commitment for log entries that are replicated on a majority of servers and that are from the current leader’s term or
the items preceding an AppendEntries RPC (may also be a heartbeat/ empty RPC) from the current term.
The moment a leader decides an entry is committed, it is from then on guaranteed that this entry will be contained in the logs of all future leaders.


< slow down man slow down >
        \\   ^__^
         \\  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

as you wish magic cow :)

-------------------------------------------------

-> # Consul  <-

-------------------------------------------------

-> # Zookeeper  <-

-------------------------------------------------

-> #   <-
