Want counterexamples to demonstrate the role of each pre-condition for determining a config is "safe". When a config is "safe" this means a node is allowed to install a new config.

Config Safety Checks:
- Term Quorum Check
- Config Quorum Check
- Op Committed in Config

### Correctness Properties:

#### NeverRollbackCommitted (Don't roll back committed entries.)

(a) You can never leave a config Ci and go to Cj if there is a committed entry E that is not present on a quorum of nodes in Ci. Why? Well, we are sure that any quorum of Cj will overlap with any quorum of Ci, but if a committed entry is not present on a quorum of Ci, then a leader could get elected in Cj without the committed entry.

(b) You can never leave a config Ci and go to Cj until you are sure that no entries will be committed in previous configs in the future. Why? If you guarantee that all entries committed in configs <= Ci are on a quorum of Ci, then you guarantee leaders in Cj will have them. But that doesn't solve the problem of new entries potentially getting committed in the future in earlier configs. If that happened, then a leader in Cj could get elected with some entries, but then after that a new entry could be committed in an earlier config, leading to rollback of committed writes. How to prevent any earlier configs from committing new writes? 

#### ElectionSafety (Don't elect two leaders in same term.)

(a) You can never leave a config Ci and go to Cj if there is a leader that has been elected in term T but a quorum of nodes do not yet know about term T. Why? Well, we are sure that any quorum of Cj will overlap with any quorum of Ci, but if less than a quorum of nodes know about a given term, then it is possible for a candidate in Cj to garner votes from a majority in term T, which we don't want, if some other node has already been elected in term T.

(b) You can never leave a config Ci and go to Cj until you are sure that no new leaders will be elected in config <= Ci. Why? If you guarantee that the terms of any previously elected leaders have propagated to a quorum of the current config, then you guarantee that in Cj no-one can get elected in the same term as an old leader. This doesn't, however, solve the issue of new leaders potentially getting elected in older configs after you move to Cj. So, you must prevent any earlier configs from electing new leaders in the future. If we are currently moving from Ci to Cj, we want to ensure that no new leaders will ever be elected in Ci or or any earlier config after moving to Cj. If we make sure that a quorum of nodes in Ci have moved to the new config, then...  



Summary: 

1. new configs must have all committed entries on a quorum of the new config.
2. new configs must prevent any entries from being committed in older configs.
3. new configs must make sure that any terms previously propagated to a quorum are propagated to a quorum in new config.
4. new configs must make sure that no leaders can get elected in old configs in future.
