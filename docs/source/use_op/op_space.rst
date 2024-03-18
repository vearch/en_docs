Space Operation
=================

http://master_server is the master service, $db_name is the name of the created database, $space_name is the name of the created tablespace.

Create Space
------------

::
   
  curl -XPUT -H "content-type: application/json" -d'
  {
      "name": "space1",
      "partition_num": 1,
      "replica_num": 1,
      "index": {
          "index_name": "gamma",
          "index_type": "HNSW"
      },
      "properties": {
          "field1": {
              "type": "keyword"
          },
          "field2": {
              "type": "integer"
          },
          "field3": {
              "type": "float",
              "index": true
          },
          "field4": {
              "type": "string",
              "array": true,
              "index": true
          },
          "field5": {
              "type": "integer",
              "index": true
          },
          "field6": {
              "type": "vector",
              "dimension": 128
          },
          "field7": {
              "type": "vector",
              "dimension": 256,
              "format": "normalization"
          }
      }
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
| index         | index config      | json       | true |                    |
+---------------+-------------------+------------+------+--------------------+
| fields        | schema config     | json       | true | define space field |
+---------------+-------------------+------------+------+--------------------+

1、Space name not be empty, do not start with numbers or underscores, try not to use special characters, etc.

2、partition_num: Specify the number of tablespace data fragments. Different fragments can be distributed on different machines to avoid the resource limitation of a single machine.

3、replica_num: The number of copies is recommended to be set to 3, which means that each piece of data has two backups to ensure high availability of data. 

index config:

+--------------+-----------------------+------------+-------+---------+
|  field name  |   field description   | field type | must  | remarks |
+==============+=======================+============+=======+=========+
| index_name   | slice index threshold | int        | false |         |
+--------------+-----------------------+------------+-------+---------+
| index_type   | index type            | string     | true  |         |
+--------------+-----------------------+------------+-------+---------+
| index_params | index parameters      | json       | false |         |
+--------------+-----------------------+------------+-------+---------+

1. index_type search model, now support IVFPQ，HNSW，GPU，IVFFLAT，BINARYIVF，FLAT.

IVFPQ:

+--------------------+--------------------------------+------------+-------+-------------------------+
|     field name     |       field description        | field type | must  |         remarks         |
+====================+================================+============+=======+=========================+
| metric_type        | computer type                  | string     | false | L2 orInnerProduct       |
+--------------------+--------------------------------+------------+-------+-------------------------+
| ncentroids         | number of buckets for indexing | int        | false | default 2048            |
+--------------------+--------------------------------+------------+-------+-------------------------+
| nsubvector         | PQ disassembler vector size    | int        | false | default 64              |
+--------------------+--------------------------------+------------+-------+-------------------------+
| bucket_init_size   | bucket init size               | int        | false | default 1000            |
+--------------------+--------------------------------+------------+-------+-------------------------+
| bucket_max_size    | max size for each bucket       | int        | false | default 1280000         |
+--------------------+--------------------------------+------------+-------+-------------------------+
| training_threshold | training data size             | int        | false | default ncentroids * 39 |
+--------------------+--------------------------------+------------+-------+-------------------------+

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
        2. No training is required to create an index, and the index_size value can be greater than 0


GPU (Compiled version for GPU):

+--------------------+--------------------------------+------------+-------+-------------------------------------+
|     field name     |       field description        | field type | must  |               remarks               |
+====================+================================+============+=======+=====================================+
| metric_type        | computer type                  | string     | true  | L2 orInnerProduct                   |
+--------------------+--------------------------------+------------+-------+-------------------------------------+
| ncentroids         | number of buckets for indexing | int        | false | default 2048                        |
+--------------------+--------------------------------+------------+-------+-------------------------------------+
| nsubvector         | PQ disassembler vector size    | int        | false | default 64, must be a multiple of 4 |
+--------------------+--------------------------------+------------+-------+-------------------------------------+
| training_threshold | training data size             | int        | false | default ncentroids * 39             |
+--------------------+--------------------------------+------------+-------+-------------------------------------+
::
 
  "index_type": "GPU",
  "index_params": {
      "metric_type": "InnerProduct",
      "ncentroids": 2048,
      "nsubvector": 64
  }

IVFFLAT:

+--------------------+--------------------------------+------------+---------+-------------------------+
|     field name     |       field description        | field type |  must   |         remarks         |
+====================+================================+============+=========+=========================+
| metric_type        | computer type                  | string     | true    | L2 orInnerProduct       |
+--------------------+--------------------------------+------------+---------+-------------------------+
| ncentroids         | number of buckets for indexing | int        | default | default 256             |
+--------------------+--------------------------------+------------+---------+-------------------------+
| training_threshold | training data size             | int        | false   | default ncentroids * 39 |
+--------------------+--------------------------------+------------+---------+-------------------------+
::
 
  "index_type": "IVFFLAT",
  "index_params": {
      "metric_type": "InnerProduct", 
      "ncentroids": 256
  }

 Note: 1. The vector storage method only supports RocksDB  

BINARYIVF:

+--------------------+--------------------------------+------------+---------+-------------------------+
|     field name     |       field description        | field type |  must   |         remarks         |
+====================+================================+============+=========+=========================+
| ncentroids         | number of buckets for indexing | int        | default | default 256             |
+--------------------+--------------------------------+------------+---------+-------------------------+
| training_threshold | training data size             | int        | false   | default ncentroids * 39 |
+--------------------+--------------------------------+------------+---------+-------------------------+
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

properties config:

1. There are four types (that is, the value of type) supported by the field defined by the table space structure: keyword, integer, float, vector (keyword is equivalent to string).

2. The keyword type fields support index and array attributes. Index defines whether to create an index, and array specifies whether to allow multiple values.

3. Integer, float type fields support the index attribute, and the fields with index set to true support the use of numeric range filtering queries.

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
----------
::
  
  curl -XGET http://master_server/dbs/$db_name/spaces/$space_name

return data
+---------------+-------------------------------------+------------+------+------------------------------------------+
|  field name   |          field description          | field type | must |                 remarks                  |
+===============+=====================================+============+======+==========================================+
| space_name    | space name                          | string     | yes  |                                          |
+---------------+-------------------------------------+------------+------+------------------------------------------+
| db_name       | database name                       | string     | yes  |                                          |
+---------------+-------------------------------------+------------+------+------------------------------------------+
| doc_num       | space document num                  | uint64     | yes  |                                          |
+---------------+-------------------------------------+------------+------+------------------------------------------+
| partition_num | partition num                       | int        | yes  |                                          |
+---------------+-------------------------------------+------------+------+------------------------------------------+
| replica_num   | replica num                         | int        | yes  |                                          |
+---------------+-------------------------------------+------------+------+------------------------------------------+
| schema        | space struct schema                 | json       | yes  |                                          |
+---------------+-------------------------------------+------------+------+------------------------------------------+
| status        | space status                        | string     | yes  | red means: There is a problem with space |
+---------------+-------------------------------------+------------+------+------------------------------------------+
| partitions    | space partitions detail information | string     | yes  |                                          |
+---------------+-------------------------------------+------------+------+------------------------------------------+
| errors        | space error information             | string     | no   |                                          |
+---------------+-------------------------------------+------------+------+------------------------------------------+

return format:
::
  {
      "code": 200,
      "msg": "success",
      "data": {
          "space_name": "ts_space",
          "db_name": "ts_db",
          "doc_num": 0,
          "partition_num": 1,
          "replica_num": 1,
          "schema": {
              "fields": {
                  "field_string": {
                      "type": "keyword"
                  },
                  "field_int": {
                      "type": "integer"
                  },
                  "field_float": {
                      "type": "float",
                      "index": true
                  },
                  "field_string_array": {
                      "type": "string",
                      "array": true,
                      "index": true
                  },
                  "field_int_index": {
                      "type": "integer",
                      "index": true
                  },
                  "field_vector": {
                      "type": "vector",
                      "dimension": 128
                  },
                  "field_vector_normal": {
                      "type": "vector",
                      "dimension": 256,
                      "format": "normalization"
                  }
              },
              "index": {
                  "index_name": "gamma",
                  "index_type": "HNSW",
                  "index_params": {
                      "metric_type": "InnerProduct",
                      "ncentroids": 2048,
                      "nsubvector": 32,
                      "nlinks": 32,
                      "efConstruction": 40,
                      "nprobe": 80,
                      "efSearch": 64,
                      "training_threshold": 70000
                  }
              }
          },
          "status": "green",
          "partitions": [
              {
                  "pid": 4,
                  "replica_num": 1,
                  "status": 4,
                  "color": "green",
                  "ip": "11.3.240.73",
                  "node_id": 1,
                  "index_status": 0,
                  "index_num": 0,
                  "max_docid": -1
              }
          ],
      }
  }

more information:
::
  
  curl -XGET http://master_server/dbs/$db_name/spaces/$space_name?detail=true

return format:
::

  {
      "code": 200,
      "msg": "success",
      "data": {
          "space_name": "ts_space",
          "db_name": "ts_db",
          "doc_num": 0,
          "partition_num": 1,
          "replica_num": 1,
          "schema": {
              "fields": {
                  "field_string": {
                      "type": "keyword"
                  },
                  "field_int": {
                      "type": "integer"
                  },
                  "field_float": {
                      "type": "float",
                      "index": true
                  },
                  "field_string_array": {
                      "type": "string",
                      "array": true,
                      "index": true
                  },
                  "field_int_index": {
                      "type": "integer",
                      "index": true
                  },
                  "field_vector": {
                      "type": "vector",
                      "dimension": 128
                  },
                  "field_vector_normal": {
                      "type": "vector",
                      "dimension": 256,
                      "format": "normalization"
                  }
              },
              "index": {
                  "index_name": "gamma",
                  "index_type": "HNSW",
                  "index_params": {
                      "metric_type": "InnerProduct",
                      "ncentroids": 2048,
                      "nsubvector": 32,
                      "nlinks": 32,
                      "efConstruction": 40,
                      "nprobe": 80,
                      "efSearch": 64,
                      "training_threshold": 70000
                  }
              }
          },
          "status": "green",
          "partitions": [
              {
                  "pid": 137,
                  "replica_num": 1,
                  "path": "/home/zc/program/vearch/deploy/export/Data/datas/",
                  "status": 4,
                  "color": "green",
                  "ip": "11.3.240.73",
                  "node_id": 1,
                  "raft_status": {
                      "ID": 137,
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
------------
::
 
  curl -XDELETE http://master_server/dbs/$db_name/spaces/$space_name
