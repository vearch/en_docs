Cluster Monitoring
==================================

http://${VEARCH_URL} is the vearch service.

Cluster Status
------------------------

::

   curl -XGET http://${VEARCH_URL}/cluster/stats


Health Status
------------------------

::

   curl -XGET http://${VEARCH_URL}/cluster/health


Server Status
------------------------

::

   curl -XGET http://${VEARCH_URL}/servers

Partition Status
------------------------

::

   curl -XGET http://${VEARCH_URL}/partitions

Clean lock
------------------------

::

  curl -XGET http://${VEARCH_URL}/clean_lock

The cluster will be locked when creating a table. If the service is abnormal during this process, the lock will not be released and will need to be manually cleared before a new table can be created.

Replica expansion and contraction
------------------------------------------------

::

  curl -XPOST -H "content-type: application/json"  -d'
  {
      "partition_ids":[1],
      "node_id": 1,
      "method": 0
  }
  ' http://${VEARCH_URL}/partition/change_member

method=0: Add a copy of shard id 1 on node id 1; method=1: Delete the copy of shard id 1 on node id 1.
