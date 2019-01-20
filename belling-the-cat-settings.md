Settings
========
Linked Data
-----------
+ Virtuoso SPARQL endpoint: http://\[IP\]:\[PORT\]/sparql
+ Named RDF Graph: http://\[IP\]:\[PORT\]/claros
	+ as defined in the RDF load
+ Entity ID: http://\[IP\]:\[PORT\]/claros/1

DNS spoof
---------
+ edit /etc/hosts
	+ add \[RPI_IP\]    data.museum.org
+ set up Virtual server on VOS
	+ data.museum.org on port 80﻿

Management
----------
+ Delete Graph
	+ *Linked Data* -> *Graphs* -> Delete graph
