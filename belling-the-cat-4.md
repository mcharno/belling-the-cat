Setup VOS for Linked Open Data
==============================
The following instructions have been largely lifted from the very detailed OpenLink Software Guide on [Deploying Linked Data](http://virtuoso.openlinksw.com/dataspace/doc/dav/wiki/Main/VirtDeployingLinkedDataGuide)

Upload RDF to VOS
-----------------
+ Start up the LOD Server if its not already running
+ Copy over the RDF file to your LOD Server

```
pi@ubuntu $ scp 5san30o3.rdf pi@192.168.56.101:~/
pi@192.168.56.101's password: 
5san30o3.rdf                                          100%  184KB 183.9KB/s   00:00
```

+ From here you can either work directly on the LOD Server or just ssh into it
+ Log into the SQL terminal with the following command

```
pi@raspberrypi ~ $ isql-vt -U dba
Connected to OpenLink Virtuoso
Driver: 06.01.3127 OpenLink Virtuoso ODBC Driver
OpenLink Interactive SQL (Virtuoso), version 0.9849b.
Type HELP; for help and EXIT; to exit.
SQL> 
```

+ Load the RDF via the SQL terminal
	+ Replace the parameters below with the path to the RDF and the Graph IRI the data will appear in (ie. http://www.museum.org/data) \[*[source](http://docs.openlinksw.com/virtuoso/fn_rdf_load_rdfxml_mt.html)*\]

```
SQL> DB.DBA.RDF_LOAD_RDFXML_MT (file_to_string_output ('/home/pi/5san30o3.rdf'), '', 'http://www.museum.org/data');

Done. -- 95 msec.
SQL>
```

+ Check for the data in VOS Conductor

![A list of the Graphs in VOS](resources/images/vos-graphs.png "A list of the Graphs in VOS")

+ And you can run a SPARQL query to ensure that data actually exists in the Graph

```
SELECT * 
WHERE {
    GRAPH <http://www.museum.org/data> {
        ?s ?p ?o
    }
}
LIMIT 100
```

![Selecting 100 enteries of the RDF data we just loaded](resources/images/vos-sparql.png "Selecting 100 enteries of the RDF data we just loaded")

+ You can also explicitly define the Graph IRI in the field above the SPARQL text area and simplify the SPARQL query

URL Rewriting
-------------
+ The following URL will render an HTML page:

```
http://192.168.56.101:8890/sparql?query=construct%20%7B%3Chttp%3A%2F%2Fwww.museum.org%2Fdata%2FE22_BCB287%3E%20%3Fp%20%3Fo%7D%20where%20%7B%3Chttp%3A%2F%2Fwww.museum.org%2Fdata%2FE22_BCB287%3E%20%3Fp%20%3Fo%7D&format=text%2Fhtml	
```

+ Which is effectively following the SPARQL query:

```
CONSTRUCT {<http://www.museum.org/data/E22_BCB287> ?p ?o} 
WHERE {<http://www.museum.org/data/E22_BCB287> ?p ?o}
```

That's all fine and dandy as a proof of concept, but real Linked Open Data resolves to a proper URI in the domain we declared in our RDF. Accomplishing that will be dependent on your own network settings, however, we can set up your local machine to pretend...

+ Update the {{/etc/hosts}} file on your workstation to forward all network traffic to our imaginary domain to the LOD Server

```
pi@ubuntu$ sudo nano /etc/hosts
```

+ Add an entry for our `www.museum.org` domain to the file

```
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
192.168.56.101  www.museum.org
127.0.0.1       localhost
255.255.255.255 broadcasthost
::1             localhost
fe80::1%lo0     localhost

                                 [ Read 13 lines ]
^G Get Help   ^O WriteOut   ^R Read File   ^Y Prev Page   ^K Cut Text     ^C Cur Pos
^X Exit       ^J Justify    ^W Where Is    ^V Next Page   ^U UnCut Text   ^T To Spell
```

+ Now we need to configure VOS to convert a URI to that SPARQL query using URL rewriting
+ Go to *Web Application Server* -> *Virtual Domains & Directories*
	+ Enter __0.0.0.0__ in *Interface*, __80__ in *Port* and __www.museum.org__ in *HTTP Host* and click *Add*
+ Now we need to create a SPARQL directory, so click on *New Directory*

![](resources/images/vos-newdir.png)

+ Set the new directory *Type* to SPARQL access point

![](resources/images/vos-dir.png)

+ The settings for the directory will be automatically set except for the *Path*, which we need to set to __/sparql__

![](resources/images/vos-dirsettings.png)

From here the instructions are designed to set up a similar system to the ADS AllegroGraph/Pubby offering. VOS is extermely configurable, so the following settings can be customised according to your preferences or your networks limitations.

+ Now we need to set up the HTML representation of our Linked Open Data, so start by creating another new virtual directory
+ This time create a __File system__ *Type* virtual directory

![](resources/images/vos-dirdata.png)

+ Set the *Path* to __/data__ and save your changes

Now we need to set up URL redirect rules to make VOS render the correct content.

+ Click on the *URL-rewrite* link for the data virtual directory

![](resources/images/vos-rewrite.png)

+ Change the settings as follows
	+ *Request Path pattern* - __(/[^#]*)__
	+ *Rule matching* - __Normal__
	+ *SPARQL Query* - __CONSTRUCT {<http://www.museum.org$s1> ?p ?o} WHERE {<http://www.museum.org$s1> ?p ?o}__
	+ *Destination Path format* - 

```
/sparql?query=CONSTRUCT%20%7B%3Chttp%3A%2F%2Fwww.museum.org$s1%3E%20%3Fp%20%3Fo%7D%20%0D%0AWHERE%20%7B%3Chttp%3A%2F%2Fwww.museum.org$s1%3E%20%3Fp%20%3Fo%7D&format=html
```

+ open a web browser and go to a node and check
+ can also use the following command in the terminal with different options for Accept header

```
$ curl -H "Accept:text/html" http://www.museum.org/data/E22_BCB287
```

or

```
$ curl -I -H "Accept:text/html" http://www.museum.org/data/E22_BCB287
```

From here it'd be nice to have content negotiation to provide different serialisations of the data depending on the user-agent or the request parameters. VOS supports this, but there was not enough time to investigate this for the workshop. OpenLink documentation can be found [here](http://www.openlink.com).


[[previous|belling-the-cat-3]] | [[next|belling-the-cat-5]]