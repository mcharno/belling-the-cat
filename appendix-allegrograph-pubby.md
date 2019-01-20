AllegroGraph and Pubby Configuration
====================================

AllegroGraph is a graph database which holds RDF triples. Pubby is a Java web application that publishes content from a triple store as Linked Open Data and does a few other things like content negotiation. The following is a basic overview of how to configure AllegroGraph and Pubby to work together to publish Linked Open Data.

AllegroGraph Installation
-------------------------

Download and follow the installation instructions on the [AllegroGraph website](http://www.franz.com/agraph/downloads/).

### Settings ###

|           |
|-----------|-------------
|Config File|`[AG_HOME]/ag44/lib/agraph.cfg`
|Data Directory|`[AG_HOME]/ag44/data`
|Log Files|`[AG_HOME]/ag44/log`

### Starting ###

```
[[AG_HOME]/ag44/bin/agraph-control --config /home/adssys/ag44/lib/agraph.cfg start
```

### Stopping ###

```
[[AG_HOME]/ag44/bin/agraph-control --config /home/adssys/ag44/lib/agraph.cfg stop
```

### AG Web View ###

This is the web interface for managing the triple store and repositories. All management commands of the triple store, including creating and deleting repositories, adding and removing triples, as well as user management, etc can be done via this interface.

+ __Location__: `http://[[SERVER]:10035/`

### SPARQL endpoint ###

This is located at root of AG Web View appended with the path to the repository.  For an imaginary `ADSTest` repository, the SPARQL endpoint would be `http://[[SERVER]:10035/repositories/ADSTest`. This is not available outside of your network unless you open the AllegroGraph port through your firewall.

### `data` Sub Domain ###

Pubby at the ADS has been configured to live on the root of the `data.archaeologydataservice.ac.uk` sub domain.  This domain points at a separate server, where Apache then forwards requests to Tomcat on the same machine.  Additionally, a reverse proxy is set up to forward all requests for AllegroGraph from the outside to go through the `sparql/` context.  This is configured in `[APACHE_HOME]/sites-available/data.archaeologydataservice.ac.uk`:

```
<Proxy *>
    Order deny, allow
    Allow from all
</Proxy>
ProxyRequests Off
ProxyPass /sparql/ http://localhost:10035/
ProxyPassReverse /sparql/ http://localhost:10035/
```

### Pubby ###

Pubby is the interface between a request and the triple store. When a web browser requests an entity, Pubby will return an HTML representation of the data. When a machine (ie. SPARQL client) makes a request, Pubby will return an XML representation of the data. This is known as content negotiation.

### Connecting Pubby to AllegroGraph ###

The following are Pubby settings for connecting to AllegroGraph. This content is found in the `config.ttl` file within the WAR package:

```
conf:projectName "Archaeology Data Service";
conf:projectHomepage <http://archaeologydataservice.ac.uk>;
conf:webBase <http://data.archaeologydataservice.ac.uk/>;
conf:indexResource <http://data.archaeologydataservice.ac.uk>;

#Dataset 1
conf:sparqlEndpoint <http://[[SERVER]/repositories/ADS>;
conf:datasetBase <http://data.archaeologydataservice.ac.uk/>;
conf:webResourcePrefix "";

#Dataset 2
conf:sparqlEndpoint <http://[[SERVER]/repositories/ADS>;
conf:datasetBase <http://data.archaeologydataservice.ac.uk/>;
conf:datasetURIPattern "(class|property)/.*";
```

Any changes to Pubby (`config.ttl`) require Tomcat to be restarted to pick up those changes.
