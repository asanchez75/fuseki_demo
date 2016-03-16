Notes on playing with Apache Jena Fuseki


## Apache Jena or Apache Jena Fuseki
*Apache Jena* is a Java API for semantic web applications. It is a set packages that you can reference from within Java applications. https://jena.apache.org/tutorials/rdf_api.html#ch-Jena%20RDF%20Packages

*Apache Jena Fuseki* on the other hand is an SPARQL Server that runs on top of Jena and makes Jena functionality available to applications via HTTP.


## Download
Download Apache Jena Fuseki from https://jena.apache.org/download/index.cgi upzip it and run the server

    cd Downloads/apache-jena-fuseki-2.3.1
    ./fuseki-server

Browse to `http://localhost:3030` and you can play with the user interface.


## A first example
With the Fuseki Server running, add a few triples to it

    curl -X POST -d @the_raven.n3 localhost:3030/datasetone/update

...and then query them

    curl -X POST -d "query=select ?s where { ?s ?p ?o . }" localhost:3030/datasetone/query

... you can also query the triples from the web page at http://localhost:3030/dataset.html?tab=query&ds=/datasetone

    select ?s ?p ?o
    where { ?s ?p ?o . }


## Adding more data

Add the data in the `data.n3` file

    curl -X POST -d @data.n3 localhost:3030/datasetone/update

we will use this data in the rest of the examples.


## Querying with SPARQL
Browse to http://localhost:3030/dataset.html?tab=query&ds=/datasetone and enter the following query:

    select ?s ?p ?o
    where { ?s ?p ?o }

This will return all the predicates and their values for all the subjects.

To fetch only the data for a single subject:

    select ?s ?p ?o
    where { <http://demo/zoia> ?p ?o }

To all the predicates for the the subjects:

    select ?p
    where { ?s ?p ?o }

Use the distinct clause if you want unique values

    select distinct ?p
    where { ?s ?p ?o }


## Let's all go to the movies
The movie data as well as the queries used in these examples were based on Chapter 4 of the Semantic Web for the Working Ontologist.

Find all movies where James Dean played a role.

    select ?movie
    where { <http://demo/jamesDean> <http://test/predicates/playedIn> ?movie }


Find all James Dean's movies and their directors (notice that there is no comma in between variables in the select!)

    select ?movie ?director
    where {
      <http://demo/jamesDean> <http://test/predicates/playedIn> ?movie .
      ?movie <http://test/predicates/directedBy> ?director .
    }

Find all actresses that played with James Dean

    select ?actress ?movie
    where {
      <http://demo/jamesDean> <http://test/predicates/playedIn> ?movie .
      ?actress <http://test/predicates/playedIn> ?movie .
      ?actress <http://test/predicates/type> <http://demo/woman> .
    }


## Inferencing
Find all animals, the ones marked directly as animals and the ones that marked as something that is a subclass of an animal.

    prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>

    select ?something
    where {?class rdfs:subClassOf* <http://demo/animal> .
      ?something a ?class .
    }


## Gotchas
If you add a value with a type, for example if the value of "Zoia Horn" was entered as

    "Zoia Horn"^^<http://www.w3.org/2001/XMLSchema#string>

you must use the value with the type when you search for it.

    select ?s
    where { ?s ?p "Zoia Horn"^^<http://www.w3.org/2001/XMLSchema#string> }



## More SPARQL queries
Query with OPTIONAL

    # all triples of type person and their age
    # only returns people with an age
    select ?s ?age
    where {
      ?s a <http://demo/person> .
      ?s <http://test/predicates/age> ?age .
    }


    # all triples of type person and their age
    # returns all people (and the age for those that have one)
    # similar to an OUTER JOIN in SQL
    select ?s ?age
    where {
      ?s a <http://demo/person> .
      OPTIONAL { ?s <http://test/predicates/age> ?age . }
    }


Query with GROUP, COUNT, and HAVING

    select ?blogger
    where {
      ?blogger a <http://demo/blogger> .
      ?blog <http://test/predicates/author> ?blogger .
      ?blog <http://test/predicates/title> ?title .
    }
    group by ?blogger
    having (count(?blog) > 2)


Note: UNSAID is not supported.


Using CONSTRUCT to generate graphs (instead of lists)

    construct {
      ?blogger <http://demo/published> ?title .
    }
    where {
      ?blogger a <http://demo/blogger> .
      ?blog <http://test/predicates/author> ?blogger .
      ?blog <http://test/predicates/title> ?title .
    }


