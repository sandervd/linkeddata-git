# LOD delta transfer over GIT 
A conceptual overview of how the GIT protocol could be used to transfer incremental data

## Concepts
### Subject frames
Consider a graph G, which contains statements (triples). A subject frame S1 is a subset of G, containing all statements with subject s1, as well as the anonymous statement (blank nodes) associated to s1.
e.g. the following graph G can be split into to subgraphs:

G = {(s1, p1-1, o1-1), (s1, p1-2, ((p1-2-1, o1-2-1), (p1-2-2, o1-2-2)), (s2, p2-1, o2-1)}

G<sub>S1</sub> = {(p1-1, o1-1), (p1-2, ((p1-2-1, o1-2-1), (p1-2-2, o1-2-2))}

G<sub>S2</sub> = {(p2-1, o2-1)}

A subject frame consists of a subject frame id and a subject frame body. Any given graph can as such be represented as a set of key-value pairs (subject frame id - subject frame body pairs).
#### Example:
Considering the following graph:
```Turtle
@prefix dc: <http://purl.org/dc/elements/1.1/> .
@prefix elem: <http://example.org/elements> .
@prefix ex: <http://example.org/stuff/1.0/> .
<http://en.wikipedia.org/wiki/Helium>                                                                                  
    elem:atomicNumber 2 ;
    elem:atomicMass 4.002602 ;
    elem:specificGravity 1.663E-4 .

<http://www.w3.org/TR/rdf-syntax-grammar>
  dc:title "RDF/XML Syntax Specification (Revised)" ;
  ex:editor [
    ex:fullname "Dave Beckett";
    ex:homePage <http://purl.org/net/dajobe/>
  ] .
```  
This graph can be split in to following two subject frames  
**Subject frame id:** http://en.wikipedia.org/wiki/Helium

**Subject frame body:**
```Turtle
@prefix elem: <http://example.org/elements> .
[ ]
    elem:atomicNumber 2 ;
    elem:atomicMass 4.002602 ;
    elem:specificGravity 1.663E-4 .                                                                      
```
**Subject frame id:** http://www.w3.org/TR/rdf-syntax-grammar

**Subject frame body:**
```Turtle
@prefix dc: <http://purl.org/dc/elements/1.1/> .
@prefix ex: <http://example.org/stuff/1.0/> .
[ ]
  dc:title "RDF/XML Syntax Specification (Revised)" ;
  ex:editor [
    ex:fullname "Dave Beckett";
    ex:homePage <http://purl.org/net/dajobe/>
  ] .
```

### GIT Concepts
#### Blob object
See https://git-scm.com/book/en/v2/Git-Internals-Git-Objects
#### Tree objects
See https://git-scm.com/book/en/v2/Git-Internals-Git-Objects
#### Commit objects
See https://git-scm.com/book/en/v2/Git-Internals-Git-Objects

## Storing Linked Data in GIT
In order to represent a graph in GIT, the following steps need to occure:
- Convert the graph to subject frames
- Convert the subject frame bodies into blob objects. The blob objects are referencable through their blob object id (hash).
- URL encode the subject frame ids
- Create a tree object, consisting of the pairs of URL encoded subject frame id - blob object id
- In case quads are stored instead of triples, an additional tree object is needed to represent each graph.
- A commit object is created to represent the state of the graph.

### Creating a GIT Blob object from a Subject Frame

### Creating a Tree object from a set of Subject Frame blobs

### Creating a Commit object to represent a point in time
