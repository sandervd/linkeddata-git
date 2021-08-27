# LOD delta transfer over GIT 
A conceptual overview of how the GIT protocol could be used to transfer incremental linked data.

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
In order to represent graphs in GIT, the following steps need to occure:
- Convert the graph to subject frames
- Convert the subject frame bodies into blob objects. The blob objects are referencable through their blob object id (hash).
- URL encode the subject frame ids
- Create a tree object, consisting of the pairs of URL encoded subject frame id - blob object id
- In case quads are stored instead of triples, an additional tree object is needed to represent each graph.
- A commit object is created to represent the state of the graph.

### From graph to set of subject frames
- Spit the graph into subgraphs by non-anonymous subject. All triples with a blank node as subject are added to the subgraph that contains the triple that defining said blank node.
- Anonymise the subgraph, by replacing the non-anymous subjects with a blank node.
- Serialize the subgraph as tutrle
The frame id is now set to the subject, the frame body is set to the anonymised serialized graph.

### Creating a GIT Blob object from a Subject Frame
The frame body can now be converted into a blob object, as per https://git-scm.com/book/en/v2/Git-Internals-Git-Objects#_object_storage

The blob object is stored as a file. The identifier of the blob is a hash of the stored object.
As a result we now have a frame id - blob id (hash of the blob object) pair.

Given the previous example, this results in the followin two objects 
id: 3d22ba434bb5bf80cd8e8caaea9967038b18f898
contents (uncompressed):
```
@prefix elem: <http://example.org/elements> .
[ ]
    elem:atomicNumber 2 ;
    elem:atomicMass 4.002602 ;
    elem:specificGravity 1.663E-4 .
```
id: 6f74f03ef616f976381c89c7751e0031cce2df9
contents (uncompressed):
```
@prefix dc: <http://purl.org/dc/elements/1.1/> .
@prefix ex: <http://example.org/stuff/1.0/> .
[ ]
  dc:title "RDF/XML Syntax Specification (Revised)" ;
  ex:editor [
    ex:fullname "Dave Beckett";
    ex:homePage <http://purl.org/net/dajobe/>
  ] .
```

### Creating a Tree object from a set of Subject Frame blobs
Before constructing a Tree object from the set of frame id, blob id pairs, the frame ids are url encoded. Typically the identifiers in git objects represent filenames, so the identifier is not allowed to contain characters that are dissalowed in filesnames, such a '/'.

The tree object is then created from the frame id and the blob id.
id: c0fb89b04be43be2d2292138a3abb5734fb4e412
contents:

In our example, this results in the following tree object (as displayed by git cat-file):
100644 blob 3d22ba434bb5bf80cd8e8caaea9967038b18f898    http%3A%2F%2Fexample.org%2Felements
100644 blob 75ee98c7043d32df52e216c9c1aeff1322bc4f33    http%3A%2F%2Fwww.w3.org%2FTR%2Frdf-syntax-grammar

### Creating a Commit object to represent a point in time
A commit is then simply an object pointing to a tree object, with the needed metadata.
Although commits typically reference a parent commit, this might nod be ideal in scenario where deltas are synced over on a regular update basis.


## Potential usage scenarios
This model could provide efficient in case of:
- CRUD heavy datasets
- Append heavy datasets (sensor data)
This model would be less ideal when new properties are added to existing objects, as then all those objects would need to be retransferred.

As both push and pull scenarios are supported by the git protocol, different data update scenarios can be envisaged as well.
In case the data needs to be materialized in a triplestore, git hooks might serve as an elegant integration point. As both the previous state previous 'checked out commit', and the next one are known, data can be reloaded in a very efficient way.
