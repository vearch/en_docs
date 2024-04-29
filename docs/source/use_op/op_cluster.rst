Cluster Monitoring
=================

http://master_server is the master service.

Cluster Status
--------

::

   curl -XGET http://master_server/cluster/stats


Health Status
--------

::

   curl -XGET http://master_server/cluster/health


Server Status
--------

::

   curl -XGET http://master_server/servers

Partition Status
--------

::

   curl -XGET http://master_server/partitions

Clean lock
--------

::

  curl -XGET http://master_server/clean_lock

The cluster will be locked when creating a table. If the service is abnormal during this process, the lock will not be released and will need to be manually cleared before a new table can be created.

Replica expansion and contraction
--------

::

  curl -XPOST -H "content-type: application/json"  -d'
  {
      "partition_ids":[1],
      "node_id": 1,
      "method": 0
  }
  ' http://master_server/partition/change_member

method=0: Add a copy of shard id 1 on node id 1; method=1: Delete the copy of shard id 1 on node id 1.

Cluster data migration
--------
You can copy the data of a certain cluster to a new cluster through the following methods to realize cluster data migration.

1. Create a new target cluster

The number of nodes in the new cluster should be consistent with that of the cluster to be migrated, a complete Vearch system should be deployed, and the processes of all ps nodes in the new cluster should be killed.

2. Metadata synchronization

Use the mirror function of etcd to copy the metadata of the cluster to be migrated to the target cluster. etcdctl is the client tool for etcd.

The operation command is as follows:
::

  export ETCDCTL_API=3
  # The principle of etcd mirroring is to read the key row by row and write it to another cluster. Among them: sourceMasterIP is a node of the original cluster master, and targetMasterIP is a node of the target cluster master.
  # ETCDCTL_API=3 ./etcdctl make-mirror target --endpoints=source
  ETCDCTL_API=3 ./etcdctl make-mirror ${targetMasterIP}:2370 --endpoints=${sourceMasterIP}:2370


3. Delete the /$cluster_name/server meta-information of the target cluster
cluster_name can be found in the configuration file config.toml
::

  export ETCDCTL_API=3
  ./etcdctl --endpoints=http://${targetMasterIP}:2370 del /$cluster_name/server --prefix


4. Copy vector data
::

  scp -r root@sourcePsIP:/export/vdb/baud root@targetPsIP:/export/vdb
  ... 

sourcePsIP is the IP of the PS node of the cluster to be migrated, and targetPsIP is the IP of the PS node of the target cluster. Here you only need to ensure that the ps node IPs of the cluster to be migrated and the target cluster are migrated one-to-one, and no special order is required.