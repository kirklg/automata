# automata
Configuration Management System designed in the context of how the word "automata" is defined.

# Design Goals for me
* Linux only (but it _should_ work on BSD since this is haskell after all)
* Keep it simple stupid where possible, but burn this rule when I feel like we need it
* Respect functional programming in its purity
* Simplicity in design, usage, and implementation
* All effing changes (state transitions) are to be immutable.
* ^ NEVER allows fancy things like in-place file modifications (use a `mv`, right?).
* Abstract all the things (e.g disk/network IO, consensus, replication, state machines)
* Test things damnit!

# Design Goals for you
* Linux only
* Should be silly stupid to use and setup
* You should never use automata to provision a system, use something else (maybe PXE Boot with a single golden image?)
* All effing changes (state transitions) are to be immutable.
* ^ NEVER allows fancy things like in-place file modifications (use a `mv`, right?).
* Rollback transactions
* Custom templating for static file configs
* Tagging and node groups
* Does not and will NEVER support outputs from custom shell scripting and run-time introspection, 1) It's not safe and 2) If your doing this, why not just configure everything in bash?
* Scales well? (not web-scale though, I don't really know what this means)

# Implementation Details
Server Architecture: Meant to always be running on a master node (or N-node cluster)
Client-only Architecture: Can run without a master node in client mode, but if this is the case, an operator must run the states manually (like Ansible)


Server Architecture layout:
On boot up, automata will fork itself into 3 (1 is optional) units of work (processes, fu#! threads (sharing memory - wat?!)
that are responsible for the following:

1. Consensus node of state changes and writes to the configured state machine (replication if specified)
Consensus is only used when RSM is specified as the final state

2. State scheduler: the things that do the actual configuring of system state on a host. All state changes will be rolled-back if a failure occurs, or if specified by the user.

3. If running in a N-node cluster, will spawn a RSM writer to fufill a state change that has consensus to the configured state machine.
State machines can be any of the 3 disk / memory / replicated state machine state machines

**NOTE**: If memory state machine is used, I hope its obvious that this is volatile memory and all transactions WILL NOT SURVIVE A REBOOT, and if running in a N-node cluster,
forces state to be reset on all state machines to ensure correctness. This is also because I'm too lazy to figure out a way to fix this, and plus I've told you here so you have been
WARNED, AND it is quite idiotic to even try imo.

RSM: which is just a fancy way of saying Log structured merge tree logs on N nodes using the configured node ring data structure, shit that was way too fancy too ;)
NodeRing data structure can be 1 of 2 things: circular ring buffers (bip-buffer I'm looking at you!) or a token ring (via consistent hashing, like how Cassandra does it


Client-only Architecture layout:
TODO
