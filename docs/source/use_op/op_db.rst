Database Operation
==================================

http://${VEARCH_URL} is the vearch service, $db_name is the name of the created database.

List Database
------------------------

::

   curl -XGET http://${VEARCH_URL}/dbs


Create Database
------------------------

::

   curl -XPOST http://${VEARCH_URL}/dbs/$db_name


View Database
------------------------

::

   curl -XGET http://${VEARCH_URL}/dbs/$db_name


Delete Database
------------------------

::

   curl -XDELETE http://${VEARCH_URL}/dbs/$db_name

Cannot delete if there is a table space under the datebase.

View Database Spaces
------------------------

::

   curl -XGET http://${VEARCH_URL}/dbs/$db_name/spaces
