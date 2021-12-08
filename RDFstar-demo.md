## Complex modelling

Below some examples of how to model complex relationships in RDF

#### 1. Standard reification

```
:bob :hasSpouse :alice .
:id1 rdf:type rdf:Statement ;
    rdf:subject :bob ;
    rdf:predicate :hasSpouse ;
    rdf:object :alice ;
    :startDate "2020-02-11"^^xsd:date .
```

#### 2. N-ary relations

```
:marriage1 rdf:type :marriage ;
    :partner1 :bob ;
    :partner2 :alice ;
    :startDate "2020-02-11"^^xsd:date .
```

#### 3. Named Graphs (TriG notation)

```
@prefix : <http://www.example.org/> .
:statementId#1 { :bob :hasSpouse :alice }
{ :statementId#1 :startDate "2020-02-11"^^xsd:date :metadata . }
```

#### 4. RDF-star

```
:bob :hasSpouse :alice .
<<:bob :hasSpouse :alice>> :startDate "2020-02-11"^^xsd:date .
```

# Creating RDF-star triples

Taking the RDF-star example above

```
:bob :hasSpouse :alice .
<<:bob :hasSpouse :alice>> :startDate "2020-02-11"^^xsd:date .
```

Note that :hasSpouse is a symmetric relation so that it can be inferred in the opposite direction. However, the metadata in the opposite direction is not asserted automatically, so it needs to be added. The file will look like:

```
:bob :hasSpouse :alice .
<<:bob :hasSpouse :alice>> :startDate "2020-02-11"^^xsd:date .
<<:alice :hasSpouse :bob>> :startDate "2020-02-11"^^xsd:date .
```

#### Expressing data certainty

```
<<:bob :hasSpouse :alice>> :startDate "2020-02-11"^^xsd:date .
<<:bob :hasSpouse :alice>> :certainty 0.4 .
```

#### A nesting example

```
<< <<:bob :hasSpouse :alice>> :startDate "2020-02-11"^^xsd:date >>
    :webpage <http://nationalenquirer.com/news/2020-02-12> .
```

### Disputable Cases 
These two examples are typical cases were some intermediary nodes would be a better modelling. Without the intermediary nodes, the information about the different measurements, and the different references, respectively, would be mixed up. See the dedicated section in the RDF-star specification at [https://w3c.github.io/rdf-star/cg-spec/2021-07-01.html#the-seminal-example](https://w3c.github.io/rdf-star/cg-spec/2021-07-01.html#the-seminal-example)


#### Expressing data value qualifiers

```
<<:painting :height 32.1>>
  :unit :cm;
  :measurementTechnique :laserScanning;
  :measuredOn "2020-02-11"^^xsd:date.
```

But consider another modelling with intermediary nodes.

```
<< :painting1 :heightInCm 32.1 >>
  :measured [
    :measurementTechnique :laserScanning;
    :measuredOn "2020-02-11"^^xsd:date;
  ],[
    :measurementTechnique :measuringTape;
    :measuredOn "2020-02-12"^^xsd:date;
  ].
```

#### Expressing data provenance

```
<<:bob :hasSpouse :alice>>
  :source :TheNationalEnquirer;
  :webpage <http://nationalenquirer.com/news/2020-02-12>;
  :retrieved "2020-02-13"^^xsd:dateTime.
```

But consider another modelling with intermediary nodes.


```
<<:man :hasSpouse :woman>>
  :reference [
    :source :theNationalEnquirer;
    :webpage <http://nationalenquirer.com/news/2020-02-12>;
    :retrieved "2020-02-13"^^xsd:dateTime
  ],[
    :source :theNewYotkTime;
    :webpage <http://nytimes.com/news/2020-02-13>;
    :retrieved "2020-02-13"^^xsd:dateTime
  ].
  
```


# Basic SPARQL-star queries

List all metadata for the given reference to a statement

```
PREFIX : <http://example.org/>

SELECT *
WHERE {
    <<?man :hasSpouse :alice>> ?p ?o
    FILTER (?man = :bob)
}
```

Binding nested triples. Note that SPARQL-star modifies the BIND clauses to select a group of embedded triples by using free variables. In the following example, the generated triple is is not known in advance, and it could not have been produced directly.

```
PREFIX : <http://example.org/>

SELECT ?t 
WHERE {
    <<?man :hasSpouse :woman >> ?p ?o
    BIND(<<:woman :hasSpouse ?man>> as ?t)   
}

```

# Converting RDF to RDF-star

Conversion can be done in the following ways:

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

Using the command line, run `reification-convert <inputfile>` from GraphDB

#### 3. Use an existing conversion to tool

For example, you could use a Java-based https://github.com/RDFstar/RDFstarTools library.
