Installation and Use
==================


Compile
--------

Environmental dependence

1. CentOS, Ubuntu and Mac OS are all OK (recommend CentOS >= 7.2).
2. go >= 1.19 required.
3. gcc >= 5 required.(recommend gcc >= 9 if want to use scann)
4. cmake >= 3.17 required.
5. OpenBLAS.
6. tbb，In CentOS it can be installed by yum. Such as: yum install tbb-devel.x86_64.
7. RocksDB == 6.2.2 (optional). You don't need to install it manually, the script installs it automatically. But you need to manually install the dependencies of rocksdb. Please refer to the installation method: https://github.com/facebook/rocksdb/blob/master/INSTALL.md
8. CUDA >= 9.0, if you want GPU support.

Compile

-  Enter the GOPATH directory, cd $GOPATH/src mkdir -p github.com/vearch cd github.com/vearch

-  Download the source code: git clone https://github.com/vearch/vearch.git ($vearch denotes the absolute path of vearch code)

-  To add GPU Index support: change BUILD_WITH_GPU from "off" to "on" in $vearch/engine/CMakeLists.txt

-  To add Scann Index support: change BUILD_WITH_SCANN from "off" to "on" in $vearch/engine/CMakeLists.txt

-  Compile vearch and gamma

   1. ``cd build``
   2. ``sh build.sh``
   
   generate \ ``vearch``\ file compile success

Deploy
--------

Before run vearch, you shuld set ``LD_LIBRARY_PATH``, Ensure that system can find gamma dynamic libraries. The gamma dynamic library that has been compiled is in the $vearch/build/gamma_build folder.

Local Model:

-  generate configuration file conf.toml
::

   [global]
       # the name will validate join cluster by same name
       name = "vearch"
       # you data save to disk path ,If you are in a production environment, You'd better set absolute paths
       data = ["datas/"]
       # log path , If you are in a production environment, You'd better set absolute paths
       log = "logs/"
       # default log type for any model
       level = "debug"
       # master <-> ps <-> router will use this key to send or receive data
       signkey = "vearch"
       skip_auth = true

   # if you are master you'd better set all config for router and ps and router and ps use default config it so cool
   [[masters]]
       # name machine name for cluster
       name = "m1"
       # ip or domain
       address = "127.0.0.1"
       # api port for http server
       api_port = 8817
       # port for etcd server
       etcd_port = 2378
       # listen_peer_urls List of comma separated URLs to listen on for peer traffic.
       # advertise_peer_urls List of this member's peer URLs to advertise to the rest of the cluster. The URLs needed to be a comma-separated list.
       etcd_peer_port = 2390
       # List of this member's client URLs to advertise to the public.
       # The URLs needed to be a comma-separated list.
       # advertise_client_urls AND listen_client_urls
       etcd_client_port = 2370
       
   [router]
       # port for server
       port = 9001
   
   [ps]
       # port for server
       rpc_port = 8081
       # raft config begin
       raft_heartbeat_port = 8898
       raft_replicate_port = 8899
       heartbeat-interval = 200 #ms
       raft_retain_logs = 10000
       raft_replica_concurrency = 1
       raft_snap_concurrency = 1 

-  start

::

   ./vearch -conf conf.toml all



Cluster Model:  

- vearch has three module: ps(PartitionServer) , master, router, run ./vearch -f conf.toml ps/router/master start ps/router/master module

Now we have five machine, two master, two ps and one router

-  master

   -  192.168.1.1
   -  192.168.1.2

-  ps

   -  192.168.1.3
   -  192.168.1.4

-  router

   -  192.168.1.5



::

    [global]
        name = "vearch"
        data = ["datas/"]
        log = "logs/"
        level = "info"
        signkey = "vearch"
        skip_auth = true

    # if you are master, you'd better set all config for router、ps and router, ps use default config it so cool
    [[masters]]
        name = "m1"
        address = "192.168.1.1"
        api_port = 8817
        etcd_port = 2378
        etcd_peer_port = 2390
        etcd_client_port = 2370
    [[masters]]
        name = "m2"
        address = "192.168.1.2"
        api_port = 8817
        etcd_port = 2378
        etcd_peer_port = 2390
        etcd_client_port = 2370
    [router]
        port = 9001
        skip_auth = true
    [ps]
        rpc_port = 8081
        raft_heartbeat_port = 8898
        raft_replicate_port = 8899
        heartbeat-interval = 200 #ms
        raft_retain_logs = 10000
        raft_replica_concurrency = 1
        raft_snap_concurrency = 1
        

-  on 192.168.1.1 , 192.168.1.2 run master

::

    ./vearch -conf config.toml master

-  on 192.168.1.3 , 192.168.1.4 run ps

::

    ./vearch -conf config.toml ps

-  on 192.168.1.5 run router

::

    ./vearch -conf config.toml router

