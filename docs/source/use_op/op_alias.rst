Alias operation
==================================

The alias can be a short string to facilitate identification and access of the corresponding space. 

Usually the alias will replace the name of the Space to facilitate business switching and other scenarios.

http://master_server represents the master service, $db_name is the name of the created database, $space_name is the name of the created space

Create space alias
--------------------------------
::
 
   curl -XPOST http://master_server/alias/$alias_name/dbs/$db_name/spaces/$space_name


Update space alias
--------------------------------
::
 
   curl -XPUT http://master_server/alias/$alias_name/dbs/$db_name/spaces/$space_name


Get alias details
--------------------------------
::
 
   curl -XGET http://master_server/alias/$alias_name

Delete space alias
--------------------------------
::
 
   curl -XDELETE http://master_server/alias/$alias_name