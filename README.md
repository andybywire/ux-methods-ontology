# UX Methods Ontology
This project is a sandbox/proof-of-concept for a "UX Methods" ontology, which is intended to provide context to a larger set of data about UX practices and online (web) resources. Similar concepts can be found at [Usability.gov](https://www.usability.gov/) and [18F Methods](https://methods.18f.gov/). This project seeks to improve on these resources by relating methods and concepts semantically, in accordance with how they are used in user-centered design workflows.  

The application this work is intended to support is currently envisioned as a web accessible UX Methods Knowledge Graph, ideally hosted as a static progressive web app. The SSG stack currently being explored for this is Jekyll, using the [Jekyll RDF](https://github.com/AKSW/jekyll-rdf) plugin from the Agile Knowledge Engineering and Semantic Web (AKSW) Research Group.

Currently, "methods" individuals are included in this single RDF file, though this is for testing and design only. Ultimately, individuals in the :Method and :WebResource classes will be drawn from a separate store. 

## Project Goals
- Create a useful, dynamic, and trustworthy resource for UX techniques, processes, and resources
- Bring together data from multiple sources and present it holistically through a single, unified interface layer
- Allow the class structure (and resulting top level categories) to evolve based on content, connections, and evolving knowledge and practice in the UX design profession
- Create a foundation upon which can be built affordances for soliciting and managing input from the wider design community

## Ontological Commitments
[Tom Gruber writes](http://www-ksl.stanford.edu/kst/what-is-an-ontology.html) that "pragmatically, a common ontology defines the vocabulary with which queries and assertions are exchanged among agents. Ontological commitments are agreements to use the shared vocabulary in a coherent and consistent manner." This ontology is based on a simple model in which the relationships between disciplines and methods are based primarily on outcomes. This allows for the autoclassificiation of methods polyhierarchically, and  roots both methods and disciplines in what they produce in user-centered design work (as opposed to politics, origins, or inertia).

![Graphical representation of UX Methods ontology structure](https://user-images.githubusercontent.com/3710835/99914748-582cd280-2cb4-11eb-8cf7-3d4709e98c0a.png)

This commitment is reflected in the ontology via two constructs:
- An individual-class mirror pattern used to align broad discipline descriptions and related methods
- A pseudo-hierarchy of method variants via transitive `:produces` property, which allows for "specializations" of methods, but without constraining those methods to a predetermined class hierarchy

## Competency questions & SPARQL test queries 
[Kendall and McGuinness](https://www.amazon.com/Ontology-Engineering-Elisa-F-Kendall-ebook/dp/B07T189GZZ) highlight the importance of "competency questions," which they refer to as "a set of questions and related answers that a knowledge base or application must be able to answer correctly" (38). Following [Allemang, Hendler, and Gandon's](https://www.amazon.com/Semantic-Web-Working-Ontologist-Effective/dp/1450376142) example of using SPARQL queries to demonstrate the semantics of a model (185), I've included several of the competency questions this knowledge graph seeks to answer, as well as the SPARQL queries that produce the intended results from the test set. 

Note: These queries were tested in the SPARQL Query and SnapSPARQL tools in Protégé. Due to the different ways they process SPARQL 1.1, queries are not processed identically across both tools. Where applicable, I have noted for which tool a query has been created. 

### What are the UX disciplines, how are they described, and what are the methods used with each of them?
```
PREFIX : <http://www.semanticweb.org/andyfitzgerald/uxmethods#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?disciplineLabel ?description (GROUP_CONCAT(DISTINCT ?methodLabel; SEPARATOR=", ") AS ?methods)

WHERE {
  ?discipline :focusesOn ?outcome ;
              dc:description ?description ;
              rdfs:label ?disciplineLabel .
  ?method :produces ?outcome ;
          rdfs:label ?methodLabel .  
}
GROUP BY ?disciplineLabel ?description
```
_Note: DISTINCT in GROUP_CONCAT does not work in SnapSPARQL_

### What methods contribute to/inform navigation design?
```
PREFIX : <http://www.semanticweb.org/andyfitzgerald/uxmethods#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?method ?description
WHERE {
  ?informingDiscipline :providesInputFor :NavigationDesign ;
                       rdfs:label ?method ;
                       dc:description ?description .
}
```
If we additionally want to know what methods navigation design contributes to:
```
PREFIX : <http://www.semanticweb.org/andyfitzgerald/uxmethods#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?method ?description?providesInputFor
WHERE {
  ?informingDiscipline :providesInputFor ?target ;
                       rdfs:label ?method ;
                       dc:description ?description .
  ?target rdfs:label ?providesInputFor .
  FILTER(?target = :NavigationDesign || ?informingDiscipline = :NavigationDesign)
}
```

### What would I use card sorting for? What insights does it produce? 
_Includes "specializations"_
```
PREFIX : <http://www.semanticweb.org/andyfitzgerald/uxmethods#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

# What would I use card sorting for? What insights does it produce and which design efforts do those inform?
# - card sorting | insights (produces — concat) | efforts it informs (providesInputFor — concat -- should be Category Design and Navigation Design)
# - bonus: include "specializations"

SELECT DISTINCT ?insightLabel ?insight_description
WHERE {
    {
      :CardSorting :produces ?insight . # Match insights that Card Sorting produces
    }
  UNION
    { 
      ?variant :specializes :CardSorting ; 
       :produces ?insight . #Match insights that specializations of Card Sorting produce
      FILTER (?insight != :CardSorting) # Filter out variant references to Card Sorting method
    } 
  ?insight dc:description ?insight_description .
  ?insight rdfs:label ?insightLabel .
    
}
```

### How do I perform a card sort and what results does it produce? 
```
PREFIX : <http://www.semanticweb.org/andyfitzgerald/uxmethods#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?method ?description ?steps (GROUP_CONCAT (?outcomes; SEPARATOR=" ,") AS ?produces)
WHERE {
  :CardSorting rdfs:label ?method ;
    dc:description ?description ;
    :steps ?steps;
    :produces ?outcome .
  ?outcome rdfs:label ?outcomes .
}
GROUP BY ?method ?description ?steps
```

### List all methods and related web resources by category
```
PREFIX : <http://www.semanticweb.org/andyfitzgerald/uxmethods#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX schema: <https://schema.org>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?discipline ?method ?WebResource

WHERE {
  ?discipline :focusesOn ?outcome .
  ?method :produces ?outcome .  
  OPTIONAL { ?method :describedBy ?WebResource }
}
```
_Note: SnapSPARQL is needed to return inferred `:describedBy` results._