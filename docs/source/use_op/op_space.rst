Space Operation
==================================

http://master_server is the master service, $db_name is the name of the created database, $space_name is the name of the created space.

Create Space
------------------------

::

    curl -XPOST -H "content-type: application/json" -d'
    {
        "name": "space1",
        "partition_num": 1,
        "replica_num": 3,
        "fields": [
            {
                "name": "field_string",
                "type": "string"
            },
            {
                "name": "field_int",
                "type": "integer"
            },
            {
                "name": "field_float",
                "type": "float",
                "index": {
                    "name": "field_float",
                    "type": "SCALAR",
                },
            },
            {
                "name": "field_string_array",
                "type": "stringArray",
                "index": {
                    "name": "field_string_array",
                    "type": "SCALAR",
                },
            },
            {
                "name": "field_int_index",
                "type": "integer",
                "index": {
                    "name": "field_int_index",
                    "type": "SCALAR",
                },
            },
            {
                "name": "field_vector",
                "type": "vector",
                "dimension": 128,
                "index": {
                    "name": "gamma",
                    "type": "IVFPQ",
                    "params": {
                        "metric_type": "InnerProduct",
                        "ncentroids": 2048,
                        "nlinks": 32,
                        "efConstruction": 40,
                    },
                },
            }
        ]
    }
    ' http://master_server/dbs/$db_name/spaces


Parameter description:

+---------------+-------------------+------------+------+--------------------+
|  field name   | field description | field type | must |      remarks       |
+===============+===================+============+======+====================+
| name          | space name        | string     | true |                    |
+---------------+-------------------+------------+------+--------------------+
| partition_num | partition number  | int        | true |                    |
+---------------+-------------------+------------+------+--------------------+
| replica_num   | replica number    | int        | true |                    |
+---------------+-------------------+------------+------+--------------------+
| fields        | schema config     | json       | true | define space field |
+---------------+-------------------+------------+------+--------------------+

1、Space name not be empty, do not start with numbers or underscores, try not to use special characters, etc.

2、partition_num: Specify the number of tablespace data fragments. Different fragments can be distributed on different machines to avoid the resource limitation of a single machine.

3、replica_num: The number of copies is recommended to be set to 3, which means that each piece of data has two backups to ensure high availability of data. 

index config:

+------------+-------------------+------------+------+---------+
| field name | field description | field type | must | remarks |
+============+===================+============+======+=========+
| name       | index name        | string     | true |         |
+------------+-------------------+------------+------+---------+
| type       | index type        | string     | true |         |
+------------+-------------------+------------+------+---------+
| params     | index parameters  | json       | true |         |
+------------+-------------------+------------+------+---------+

1. Index type
Index type currently supports seven types in two categories, scalar index: SCALAR; 
vector index: IVFPQ, HNSW, GPU, IVFFLAT, BINARYIVF, FLAT, please see the link for details
https://github.com/vearch/vearch/wiki/Vearch%E7%B4%A2%E5%BC%95%E4%BB%8B%E7%BB%8D%E5%92%8C%E5%8F% 82%E6%95%B0%E9%80%89%E6%8B%A9.

Scalar indexes only need to set name and type.

The parameter configurations and default values required for different vector index types are as follows:

IVFPQ:

+--------------------+------------------------------------------------------+------------+-------+----------------------------------------------------------------------------------------------------------------------------------------+
|     field name     |                  field description                   | field type | must  |                                                                remarks                                                                 |
+====================+======================================================+============+=======+========================================================================================================================================+
| metric_type        | computer type                                        | string     | true  | L2 orInnerProduct                                                                                                                      |
+--------------------+------------------------------------------------------+------------+-------+----------------------------------------------------------------------------------------------------------------------------------------+
| ncentroids         | number of buckets for indexing                       | int        | true  | default 2048                                                                                                                           |
+--------------------+------------------------------------------------------+------------+-------+----------------------------------------------------------------------------------------------------------------------------------------+
| nsubvector         | PQ disassembler vector size                          | int        | false | default 64                                                                                                                             |
+--------------------+------------------------------------------------------+------------+-------+----------------------------------------------------------------------------------------------------------------------------------------+
| bucket_init_size   | bucket init size                                     | int        | false | default 1000                                                                                                                           |
+--------------------+------------------------------------------------------+------------+-------+----------------------------------------------------------------------------------------------------------------------------------------+
| bucket_max_size    | max size for each bucket                             | int        | false | default 1280000                                                                                                                        |
+--------------------+------------------------------------------------------+------------+-------+----------------------------------------------------------------------------------------------------------------------------------------+
| training_threshold | training data size                                   | int        | false | The default ncentroids * 39 is the amount of data required for each shard training, not the amount of data in the space table. |
+--------------------+------------------------------------------------------+------------+-------+----------------------------------------------------------------------------------------------------------------------------------------+
| nprobe             | the number of cluster centers found during retrieval | int        | false | default 80                                                                                                                             |
+--------------------+------------------------------------------------------+------------+-------+----------------------------------------------------------------------------------------------------------------------------------------+

::

  "index_type": "IVFPQ",
  "index_params": {
      "metric_type": "InnerProduct",
      "ncentroids": 2048,
      "nsubvector": 64
  }

set ivfpq with hnsw：

::

  "index_size": 2600000,
  "id_type": "string",
  "index_type": "IVFPQ",
  "index_params": {
      "metric_type": "InnerProduct",
      "ncentroids": 65536,
      "nsubvector": 64,
      "hnsw" : {
          "nlinks": 32,
          "efConstruction": 200,
          "efSearch": 64
      }
  }

HNSW:

+----------------+-----------------------------+------------+-------+-------------------+
|   field name   |      field description      | field type | must  |      remarks      |
+================+=============================+============+=======+===================+
| metric_type    | computer type               | string     | true  | L2 orInnerProduct |
+----------------+-----------------------------+------------+-------+-------------------+
| nlinks         | Number of node neighbors    | int        | false | default 32        |
+----------------+-----------------------------+------------+-------+-------------------+
| efConstruction | Composition traversal depth | int        | false | default 40        |
+----------------+-----------------------------+------------+-------+-------------------+

::

  "index_type": "HNSW",
  "index_params": {
      "metric_type": "L2",
      "nlinks": 32,
      "efConstruction": 40
  }

  Note: 1. Vector storage only supports MemoryOnly

GPU (Compiled version for GPU):

+--------------------+------------------------------------------------------+------------+-------+----------------------------------------------------------------------------------------------------------------------------------------+
|     field name     |                  field description                   | field type | must  |                                                                remarks                                                                 |
+====================+======================================================+============+=======+========================================================================================================================================+
| metric_type        | computer type                                        | string     | true  | L2 orInnerProduct                                                                                                                      |
+--------------------+------------------------------------------------------+------------+-------+----------------------------------------------------------------------------------------------------------------------------------------+
| ncentroids         | number of buckets for indexing                       | int        | true  | default 2048                                                                                                                           |
+--------------------+------------------------------------------------------+------------+-------+----------------------------------------------------------------------------------------------------------------------------------------+
| nsubvector         | PQ disassembler vector size                          | int        | false | default 64, must be a multiple of 4                                                                                                    |
+--------------------+------------------------------------------------------+------------+-------+----------------------------------------------------------------------------------------------------------------------------------------+
| training_threshold | training data size                                   | int        | false | The default ncentroids * 39 is the amount of data required for each shard training, not the amount of data in the space table. |
+--------------------+------------------------------------------------------+------------+-------+----------------------------------------------------------------------------------------------------------------------------------------+
| nprobe             | the number of cluster centers found during retrieval | int        | false | default 80                                                                                                                             |
+--------------------+------------------------------------------------------+------------+-------+----------------------------------------------------------------------------------------------------------------------------------------+

::
 
  "index_type": "GPU",
  "index_params": {
      "metric_type": "InnerProduct",
      "ncentroids": 2048,
      "nsubvector": 64
  }

IVFFLAT:

+--------------------+------------------------------------------------------+------------+-------+----------------------------------------------------------------------------------------------------------------------------------------+
|     field name     |                  field description                   | field type | must  |                                                                remarks                                                                 |
+====================+======================================================+============+=======+========================================================================================================================================+
| metric_type        | computer type                                        | string     | true  | L2 orInnerProduct                                                                                                                      |
+--------------------+------------------------------------------------------+------------+-------+----------------------------------------------------------------------------------------------------------------------------------------+
| ncentroids         | number of buckets for indexing                       | int        | true  | default 256                                                                                                                            |
+--------------------+------------------------------------------------------+------------+-------+----------------------------------------------------------------------------------------------------------------------------------------+
| training_threshold | training data size                                   | int        | false | The default ncentroids * 39 is the amount of data required for each shard training, not the amount of data in the space table. |
+--------------------+------------------------------------------------------+------------+-------+----------------------------------------------------------------------------------------------------------------------------------------+
| nprobe             | the number of cluster centers found during retrieval | int        | false | default 80                                                                                                                             |
+--------------------+------------------------------------------------------+------------+-------+----------------------------------------------------------------------------------------------------------------------------------------+

::

  "index_type": "IVFFLAT",
  "index_params": {
      "metric_type": "InnerProduct", 
      "ncentroids": 256
  }

 Note: 1. The vector storage method only supports RocksDB  

BINARYIVF:

+--------------------+------------------------------------------------------+------------+-------+--------------------------------------------------------------------------------------------------------------------------------+
|     field name     |                  field description                   | field type | must  |                                                            remarks                                                             |
+====================+======================================================+============+=======+================================================================================================================================+
| ncentroids         | number of buckets for indexing                       | int        | true  | default 256                                                                                                                    |
+--------------------+------------------------------------------------------+------------+-------+--------------------------------------------------------------------------------------------------------------------------------+
| training_threshold | training data size                                   | int        | false | The default ncentroids * 39 is the amount of data required for each shard training, not the amount of data in the space table. |
+--------------------+------------------------------------------------------+------------+-------+--------------------------------------------------------------------------------------------------------------------------------+
| nprobe             | the number of cluster centers found during retrieval | int        | false | default 80                                                                                                                     |
+--------------------+------------------------------------------------------+------------+-------+--------------------------------------------------------------------------------------------------------------------------------+

::

  "index_type": "BINARYIVF",
  "index_params": {
      "ncentroids": 256
  }
  
  Note: 1. The vector length is a multiple of 8

FLAT:

+-------------+-------------------+------------+------+-------------------+
| field name  | field description | field type | must |      remarks      |
+=============+===================+============+======+===================+
| metric_type | computer type     | string     | true | L2 orInnerProduct |
+-------------+-------------------+------------+------+-------------------+

::

  "index_type": "FLAT",
  "index_params": {
      "metric_type": "InnerProduct"
  }
  
 Note: 1. The vector storage method only supports MemoryOnly

fields config:

1. There are seven types (that is, the value of type) supported by the field defined by the table space structure: string(keyword)，stringArray, integer， long， float，double， vector (keyword is equivalent to string).

2. The string type fields(include stringArray) support index. Index defines whether to create an index.

3. Integer, float, long, double type fields support the index attribute, and the fields with index set to true support the use of numeric range filtering queries.

4. Vector type fields are feature fields. Multiple feature fields are supported in a table space. The attributes supported by vector type fields are as follows:


+-------------+---------------------------+-----------+--------+------------------------------------------------------------+
|field name   |field description          |field type |must    |remarks                                                     | 
+=============+===========================+===========+========+============================================================+
|dimension    |feature dimension          |int        |true    |Value is an integral multiple of the above nsubvector value |
+-------------+---------------------------+-----------+--------+------------------------------------------------------------+
|store_type   |feature storage type       |string     |false   |support MemoryOnly and RocksDB                              |
+-------------+---------------------------+-----------+--------+------------------------------------------------------------+
|store_param  |storage parameter settings |json       |false   |set the memory size of data                                 |
+-------------+---------------------------+-----------+--------+------------------------------------------------------------+
|model_id     |feature plug-in model      |string     |false   |Specify when using the feature plug-in service              |
+-------------+---------------------------+-----------+--------+------------------------------------------------------------+


5. dimension: define that type is the field of vector, and specify the dimension size of the feature.

6. store_type: raw vector storage type, there are the following options

"MemoryOnly": Vectors are stored in the memory, and the amount of stored vectors is limited by the memory. It is suitable for scenarios where the amount of vectors on a single machine is not large (10 millions) and high performance requirements

"RocksDB": Vectors are stored in RockDB (disk), and the amount of stored vectors is limited by the size of the disk. It is suitable for scenarios where the amount of vectors on a single machine is huge (above 100 millions) and performance requirements are not high.


7. store_param: storage parameters of different store_type, it contains the following two sub-parameters

cache_size: interge type, the unit is M bytes, the default is 1024. When store_type="RocksDB", it indicates the read buffer size of RocksDB. The larger the value, the better the performance of reading vector. Generally set 1024, 2048, 4096 and 6144; store_type ="MemoryOnly", cache_size is not in effect.


Scalar Index
Gamma engine supports scalar index, provides the filtering function for scalar data, the opening method refers to the 2nd and 3rd in the "fields config", and the retrieval method refers to the "filter json structure elucidation" in the "Search"

View Space
--------------------
::

  curl -XGET http://master_server/dbs/$db_name/spaces/$space_name

返回数据详细格式：

+----------+----------------+--------+--------------+------+
| 字段标识 |    字段含义    |  类型  | 是否一定返回 | 备注 |
+==========+================+========+==============+======+
| code     | return code    | int    | 是           |      |
+----------+----------------+--------+--------------+------+
| msg      | return message | string | 否           |      |
+----------+----------------+--------+--------------+------+
| data     | return data    | json   | 否           |      |
+----------+----------------+--------+--------------+------+

return data:

+---------------+-------------------------------------+-------------+------+------------------------------------------+
|  field name   |          field description          | field type  | must |                 remarks                  |
+===============+=====================================+=============+======+==========================================+
| space_name    | space name                          | string      | yes  |                                          |
+---------------+-------------------------------------+-------------+------+------------------------------------------+
| db_name       | database name                       | string      | yes  |                                          |
+---------------+-------------------------------------+-------------+------+------------------------------------------+
| doc_num       | space document num                  | uint64      | yes  |                                          |
+---------------+-------------------------------------+-------------+------+------------------------------------------+
| partition_num | partition num                       | int         | yes  |                                          |
+---------------+-------------------------------------+-------------+------+------------------------------------------+
| replica_num   | replica num                         | int         | yes  |                                          |
+---------------+-------------------------------------+-------------+------+------------------------------------------+
| schema        | space struct schema                 | json        | yes  |                                          |
+---------------+-------------------------------------+-------------+------+------------------------------------------+
| status        | space status                        | string      | yes  | red means: There is a problem with space |
+---------------+-------------------------------------+-------------+------+------------------------------------------+
| partitions    | space partitions detail information | json        | yes  |                                          |
+---------------+-------------------------------------+-------------+------+------------------------------------------+
| errors        | space error information             | string list | no   |                                          |
+---------------+-------------------------------------+-------------+------+------------------------------------------+

return format:
::

    {
        "code": 0,
        "data": {
            "space_name": "ts_space",
            "db_name": "ts_db",
            "doc_num": 0,
            "partition_num": 1,
            "replica_num": 3,
            "schema": {
                "fields": [
                    {
                        "name": "field_string",
                        "type": "string"
                    },
                    {
                        "name": "field_int",
                        "type": "integer"
                    },
                    {
                        "name": "field_float",
                        "type": "float",
                        "index": {
                            "name": "field_float",
                            "type": "SCALAR"
                        }
                    },
                    {
                        "name": "field_string_array",
                        "type": "stringArray",
                        "index": {
                            "name": "field_string_array",
                            "type": "SCALAR"
                        }
                    },
                    {
                        "name": "field_int_index",
                        "type": "integer",
                        "index": {
                            "name": "field_int_index",
                            "type": "SCALAR"
                        }
                    },
                    {
                        "name": "field_vector",
                        "type": "vector",
                        "dimension": 128,
                        "index": {
                            "name": "gamma",
                            "type": "IVFPQ",
                            "params": {
                                "metric_type": "InnerProduct",
                                "ncentroids": 2048,
                                "nlinks": 32,
                                "efConstruction": 40
                            }
                        }
                    }
                ]
            },
            "status": "green",
            "partitions": [
                {
                    "pid": 1,
                    "replica_num": 1,
                    "status": 4,
                    "color": "green",
                    "ip": "x.x.x.x",
                    "node_id": 1,
                    "index_status": 0,
                    "index_num": 0,
                    "max_docid": -1
                },
                {
                    "pid": 2,
                    "replica_num": 1,
                    "status": 4,
                    "color": "green",
                    "ip": "x.x.x.x",
                    "node_id": 2,
                    "index_status": 0,
                    "index_num": 0,
                    "max_docid": -1
                },
                {
                    "pid": 3,
                    "replica_num": 1,
                    "status": 4,
                    "color": "green",
                    "ip": "x.x.x.x",
                    "node_id": 3,
                    "index_status": 0,
                    "index_num": 0,
                    "max_docid": -1
                }
            ],
        }
    }

more information
::

  curl -XGET http://master_server/dbs/$db_name/spaces/$space_name?detail=true

return format
::

    {
        "code": 0,
        "data": {
            "space_name": "ts_space",
            "db_name": "ts_db",
            "doc_num": 0,
            "partition_num": 1,
            "replica_num": 3,
            "schema": {
                "fields": [
                    {
                        "name": "field_string",
                        "type": "string"
                    },
                    {
                        "name": "field_int",
                        "type": "integer"
                    },
                    {
                        "name": "field_float",
                        "type": "float",
                        "index": {
                            "name": "field_float",
                            "type": "SCALAR"
                        }
                    },
                    {
                        "name": "field_string_array",
                        "type": "stringArray",
                        "index": {
                            "name": "field_string_array",
                            "type": "SCALAR"
                        }
                    },
                    {
                        "name": "field_int_index",
                        "type": "integer",
                        "index": {
                            "name": "field_int_index",
                            "type": "SCALAR"
                        }
                    },
                    {
                        "name": "field_vector",
                        "type": "vector",
                        "dimension": 128,
                        "index": {
                            "name": "gamma",
                            "type": "IVFPQ",
                            "params": {
                                "metric_type": "InnerProduct",
                                "ncentroids": 2048,
                                "nlinks": 32,
                                "efConstruction": 40
                            }
                        }
                    }
                ]
            },
            "status": "green",
            "partitions": [
                {
                    "pid": 1,
                    "replica_num": 1,
                    "path": "/export/Data/datas/",
                    "status": 4,
                    "color": "green",
                    "ip": "x.x.x.x",
                    "node_id": 1,
                    "raft_status": {
                        "ID": 1,
                        "NodeID": 1,
                        "Leader": 1,
                        "Term": 1,
                        "Index": 1,
                        "Commit": 1,
                        "Applied": 1,
                        "Vote": 1,
                        "PendQueue": 0,
                        "RecvQueue": 0,
                        "AppQueue": 0,
                        "Stopped": false,
                        "RestoringSnapshot": false,
                        "State": "StateLeader",
                        "Replicas": {
                            "1": {
                                "Match": 1,
                                "Commit": 1,
                                "Next": 2,
                                "State": "ReplicaStateProbe",
                                "Snapshoting": false,
                                "Paused": false,
                                "Active": true,
                                "LastActive": "2024-03-18T09: 59: 17.095112556+08: 00",
                                "Inflight": 0
                            }
                        }
                    },
                    "index_status": 0,
                    "index_num": 0,
                    "max_docid": -1
                },
                {
                    "pid": 2,
                    "replica_num": 1,
                    "path": "/export/Data/datas/",
                    "status": 4,
                    "color": "green",
                    "ip": "x.x.x.x",
                    "node_id": 2,
                    "raft_status": {
                        "ID": 2,
                        "NodeID": 1,
                        "Leader": 1,
                        "Term": 1,
                        "Index": 1,
                        "Commit": 1,
                        "Applied": 1,
                        "Vote": 1,
                        "PendQueue": 0,
                        "RecvQueue": 0,
                        "AppQueue": 0,
                        "Stopped": false,
                        "RestoringSnapshot": false,
                        "State": "StateLeader",
                        "Replicas": {
                            "1": {
                                "Match": 1,
                                "Commit": 1,
                                "Next": 2,
                                "State": "ReplicaStateProbe",
                                "Snapshoting": false,
                                "Paused": false,
                                "Active": true,
                                "LastActive": "2024-03-18T09: 59: 17.095112556+08: 00",
                                "Inflight": 0
                            }
                        }
                    },
                    "index_status": 0,
                    "index_num": 0,
                    "max_docid": -1
                },
                {
                    "pid": 3,
                    "replica_num": 1,
                    "path": "/export/Data/datas/",
                    "status": 4,
                    "color": "green",
                    "ip": "x.x.x.x",
                    "node_id": 3,
                    "raft_status": {
                        "ID": 3,
                        "NodeID": 1,
                        "Leader": 1,
                        "Term": 1,
                        "Index": 1,
                        "Commit": 1,
                        "Applied": 1,
                        "Vote": 1,
                        "PendQueue": 0,
                        "RecvQueue": 0,
                        "AppQueue": 0,
                        "Stopped": false,
                        "RestoringSnapshot": false,
                        "State": "StateLeader",
                        "Replicas": {
                            "1": {
                                "Match": 1,
                                "Commit": 1,
                                "Next": 2,
                                "State": "ReplicaStateProbe",
                                "Snapshoting": false,
                                "Paused": false,
                                "Active": true,
                                "LastActive": "2024-03-18T09: 59: 17.095112556+08: 00",
                                "Inflight": 0
                            }
                        }
                    },
                    "index_status": 0,
                    "index_num": 0,
                    "max_docid": -1
                }
            ]
        }
    }

Delete Space
------------------------
::
 
  curl -XDELETE http://master_server/dbs/$db_name/spaces/$space_name
