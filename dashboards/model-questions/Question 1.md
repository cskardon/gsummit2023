# What is the route from Operational Point X to Operational Point Y?


What’s the quickest way to get a repair crew from Technical Services to a given Switch?


Sweden: SEM "Malmö central"
UK: UKN2431 (Paddington) "PAD London Paddington"

```
MATCH 
    (:OperationalPointName {name:'Malmö central'})<-[:NAMED]-(malmo:OperationalPoint),
    (:OperationalPointName {name:'PAD London Paddington'})<-[:NAMED]-(paddington:OperationalPoint)
WITH 
    malmo, paddington
MATCH shortest = shortestPath( (malmo)-[:SECTION*]-(paddington) )
RETURN shortest
```

```
MATCH 
    (:OperationalPointName {name:'Malmö central'})<-[:NAMED]-(malmo:OperationalPoint),
    (:OperationalPointName {name:'PAD London Paddington'})<-[:NAMED]-(paddington:OperationalPoint)
WITH 
    malmo, paddington
MATCH shortest = shortestPath( (malmo)-[:SECTION*]-(paddington) )
RETURN shortest
CALL apoc.algo.dijkstra(brussels, berlin, 'SECTION', 'sectionlength') YIELD path, weight
RETURN path, weight;