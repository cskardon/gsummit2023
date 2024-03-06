# What is an alternative route if an Operational Point on a route is closed?

> A Switch is broken and we need to reroute Trains

## Introduction

The [previous question](Question%201.md) we worked out how to look at paths in NeoDash, and we're not going to be going too different here.

In essence we want the same page as before, but this time, we want to add one extra parameter - and that's the 'location to avoid' parameter.

## Examples

For this question, a good set of example stations to try are from 'Paddington' to 'Harlesden' avoiding 'Willesden Junction LL'.

## Parameter Entry

As with the previous question, we're going to add a free-text entry to find our location to avoid, we know how to do that already, so we can just repeat what we did before, but with a unique `avoidlocation` parameter.

## Graph and Map views to avoid

We can now repeat our map from our previous dashboard, but we're going to change the query so we can use [Quantified Path Patterns](https://neo4j.com/docs/cypher-manual/current/patterns/concepts/#quantified-path-patterns):

```cypher
MATCH 
    (start:OperationalPoint {id:$neodash_q2StartId}),
    (end:OperationalPoint {id:$neodash_q2EndId})
WITH start, end 
MATCH p =
    (start)
    ( (:OperationalPoint)-[:SECTION]-(op:OperationalPoint WHERE NOT(op.id = $neodash_q2AvoidId) ) )+
    (end)
RETURN p LIMIT 1
```

The start of the query is the same as before, we're `MATCH`ing the `start`/`end` as before. 

We then `MATCH` to find a path between those two nodes with the provided 'pattern':

```cypher
( (:OperationalPoint)-[:SECTION]-(op:OperationalPoint WHERE NOT(op.id = $neodash_q2AvoidId) ) )+
```

In the pattern, there are two key bits, first, we're excluding any pattern that includes an `OperationalPoint` with the `q2AvoidId` parameter, and secondly we're doing an `+` expansion. This is the equivalent of `1..*` expansion, i.e. this pattern has to repeat _at least once_ but there is no upper bound.

> This **could** be problematic as we could find a large amount of paths, in particular for two stations a long distance apart - and our sandbox environments don't have the memory capacity to run with them.

## Graph and Map views without avoiding

Visually, it's nice to be able to see what the route looked like originally, so you can compare the differences, to do so you can add another report with the following Cypher.

```cypher
MATCH 
    (start:OperationalPoint {id:$neodash_q2StartId}),(end:OperationalPoint {id:$neodash_q2EndId})
MATCH p =
    (start)
    ( (:OperationalPoint)-[:SECTION]-(op:OperationalPoint) )+
    (end)
RETURN p LIMIT 1
```

This is the same Cypher as the previous step, only it doesn't include the `WHERE NOT` section.

## Improvements to the experience

As it stands, we have a functional dashboard, but it does heavily rely on knowing the names of the junctions, and doesn't give us the opportunity to 'play' around, and just try different `OperationalPoint`s on the route.

So, we'll add another 'table' report, but this one will contain all the `OperationalPoint`s on the route. The majority of the query stays the same, as we want the same `path`, but this time, after getting our `path` we [`UNWIND`](https://neo4j.com/docs/cypher-manual/current/clauses/unwind/) the nodes in the path using the [`nodes()`](https://neo4j.com/docs/cypher-manual/current/functions/list/#functions-nodes) function.

We do some filtering (`WHERE p <> start AND p <> end`) to ensure we don't return our start/end points and then return the names of the `OperationalPoint`s on the way.

```cypher
MATCH 
    (start:OperationalPoint {id:$neodash_q2StartId}),(end:OperationalPoint {id:$neodash_q2EndId})
MATCH path =
    (start)
    ( (:OperationalPoint)-[:SECTION]-(op:OperationalPoint) )+
    (end)
WITH path LIMIT 1
UNWIND nodes(path) AS p
WITH p WHERE p <> start AND p <> end
MATCH (p)-[:NAMED]->(pName:OperationalPointName)
RETURN DISTINCT pName.name AS name, p.id AS __ID
```