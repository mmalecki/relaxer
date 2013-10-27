# relaxer

CouchDB needs a relaxer.
I love Redis sentinel. It's easy to configure and it works.

## Architecture
We'll steal lots of Redis sentinel's architecure. 

### Masters
One relaxer can watch one master (unlike Redis sentinel). That might change in the future but we want to make it easier to start with.

### Relaxer autodiscovery
Relaxer creates `relaxer` database (or as configured) database in the CouchDB. All other connected relaxers check the DB periodically to find out other relaxers.

### Leader election
The first connected relaxer will become the leader (as marked in the `relaxer` database).
If leader fails, newest relaxer becomes the new leader.

### Failover process
Let's assume a cluster with servers `A`, `B` and `C`. `A` was configured as master. `B` and `C` are replicating from `A`. In the same cluster there are sentinels `X`, `Y`, `Z`. They are configured to quorum 2 (that means that we need 2 Sentinels to confirm that master is down and start the failover.

1. `A` fails.
2. All relaxers which noticed fail notify the current leader.
3. As soon as leader receives enough confirmations to reach the quorum, it performs the failover (announces `ODOWN`).
4. Random slave (which is not down) is elected as a new master. Change is announced.
5. All other slaves are directed to replicate from new master.
