## Complex modelling

Below some examples of how to model complex concepts in RDF

#### 1. Standard reification

```
:man :hasSpouse :woman .
:id1 rdf:type rdf:Statement ;
    rdf:subject :man ;
    rdf:predicate :hasSpouse ;
    rdf:object :woman ;
    :startDate "2020-02-11"^^xsd:date .
```

#### 2. N-ary relations

```
:Marriage1 rdf:type :Marriage ;
    :partner1 :man ;
    :partner2 :woman ;
    :startDate "2020-02-11"^^xsd:date .
```

#### 3. Named Graphs

```
:man :hasSpouse :woman :statementId#1 .
:statementId#1 :startDate "2020-02-11"^^xsd:date :metadata .
```

#### 4. RDF* 

```
:man :hasSpouse :woman .
<<:man :hasSpouse :woman>> :startDate "2020-02-11"^^xsd:date .
```

# Creating RDF* triples

Taking the RDF* example above

```
:man :hasSpouse :woman .
<<:man :hasSpouse :woman>> :startDate "2020-02-11"^^xsd:date .
```

Note that :hasSpouse is a symmetric relation so that it can be inferred in the opposite direction. However, the metadata in the opposite direction is not asserted automatically, so it needs to be added. The file will look like:

```
:man :hasSpouse :woman .
<<:man :hasSpouse :woman>> :startDate "2020-02-11"^^xsd:date .
<<:woman :hasSpouse :man>> :startDate "2020-02-11"^^xsd:date .
```

#### Expressing data certainty

```
:man :hasSpouse :woman .
<<:man :hasSpouse :woman>> :startDate "2020-02-11"^^xsd:date .
<<:man :hasSpouse :woman>> :certainty 0.9 .
```


#### Expressing data value qualifiers

```
<<:painting :height 32.1>>
  :unit :cm;
  :measurementTechnique :laserScanning;
  :measuredOn "2020-02-11"^^xsd:date.
```

#### Expressing data provenance

```
<<:man :hasSpouse :woman>>
  :source :TheNationalEnquirer;
  :webpage <http://nationalenquirer.com/news/2020-02-12>;
  :retrieved "2020-02-13"^^xsd:dateTime.
 ```

#### A nesting example

```
<< <<:man :hasSpouse :woman>> :startDate "2020-02-11"^^xsd:date >>
    :webpage <http://nationalenquirer.com/news/2020-02-12> .
```

# Basic SPARQL* queries

List all metadata for the given reference to a statement

```
PREFIX : <http://example.org/>

SELECT *
WHERE {
    <<?man :hasSpouse :woman>> ?p ?o
    FILTER (?man = :man)
}
```

Binding nested triples. Note that SPARQL* modifies the BIND clauses to select a group of embedded triples by using free variables

```
PREFIX : <http://example.org/>

SELECT *
WHERE {
    BIND(<<:man :hasSpouse :woman>> as ?t)
    ?t ?p ?o
    
}
```

# Coverting RDF to RDF*

Conversion can be done in the following ways.

#### 1. GraphDB Construct

```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
CONSTRUCT {
    <<?subject ?predicate ?object>> ?p ?o .
} WHERE {
    ?reification a rdf:Statement .
    ?reification rdf:subject ?subject .
    ?reification rdf:predicate ?predicate .
    ?reification rdf:object ?object .
    ?reification ?p ?o .
    FILTER (?p NOT IN (rdf:subject, rdf:predicate, rdf:object) &&
    (?p != rdf:type && ?object != rdf:Statement))
}
```

#### 2. GraphDB command line

Using the command line, run `reification-convert <inputfile>` 

#### 3. Use an existing conversion to tool

eg : (Java) https://github.com/RDFstar/RDFstarTools
