Backup and Migration
====================
http://${VEARCH_URL} represents the vearch service, DB_NAME is the name of the created database, and SPACE_NAME is the name of the created space.

Vearch can backup cluster table spaces to remote storage for migration or recovery. Before using the backup feature, you need to configure S3 storage.


Backup
----------------

::

  curl --location 'http://${VEARCH_URL}/backup/dbs/${DB_NAME}/spaces/${SPACE_NAME}' \
  --data '{
      "command": "create",
      "with_schema": true,
      "s3_param": {
          "access_key": "USER_ACCESS_KEY",
          "bucket_name": "USER_BUCKET_NAME",
          "endpoint": "S3_ENDPOINT",
          "secret_key": "USER_SECRET",
          "use_ssl": true
      }
  }'

with_schema: Set whether to backup table information, default is true. If set to true, backup table information; otherwise, only backup data.

access_key: Access key for S3 storage.

bucket_name: Bucket name for S3 storage.

endpoint: Endpoint for S3 storage.

secret_key: Secret key for S3 storage.

use_ssl: Whether to use SSL.

Restore
----------------

::

  curl --location 'http://${VEARCH_URL}/backup/dbs/${DB_NAME}/spaces/${SPACE_NAME}' \
  --data '{
      "command": "restore",
      "with_schema": true,
      "s3_param": {
          "access_key": "USER_ACCESS_KEY",
          "bucket_name": "USER_BUCKET_NAME",
          "endpoint": "S3_ENDPOINT",
          "secret_key": "USER_SECRET",
          "use_ssl": true
      }
  }'


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
