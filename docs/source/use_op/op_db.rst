Database Operation
=================

http://master_server is the master service, $db_name is the name of the created database.

List Database
--------

::

   curl -XGET http://master_server/dbs


Create Database
--------

::

   curl -XPOST http://master_server/dbs/$db_name


View Database
--------

::

   curl -XGET http://master_server/dbs/$db_name


Delete Database
--------

::

   curl -XDELETE http://master_server/dbs/$db_name

Cannot delete if there is a table space under the datebase.

View Database Space
--------

::

   curl -XGET http://master_server/dbs/$db_name/spaces



