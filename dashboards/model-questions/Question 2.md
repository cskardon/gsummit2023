# What is an alternative route if an Operational Point on a route is closed?

> A Switch is broken and we need to reroute Trains



//Path avoiding somewhere
MATCH 
    (start:OperationalPoint {id:$neodash_q2StartId}),(end:OperationalPoint {id:$neodash_q2EndId})
MATCH p =
    (start)
    ( (:OperationalPoint)-[:SECTION]-(op:OperationalPoint WHERE NOT(op.id = $neodash_q2AvoidId) ) ){1,10}
    (end)
RETURN p LIMIT 1


// Path without avoiding anywhere
MATCH 
    (start:OperationalPoint {id:$neodash_q2StartId}),(end:OperationalPoint {id:$neodash_q2EndId})
MATCH p =
    (start)
    ( (:OperationalPoint)-[:SECTION]-(op:OperationalPoint) ){1,10}
    (end)
RETURN p LIMIT 1





//Get all the Operational points en route.

MATCH 
    (start:OperationalPoint {id:$neodash_q2StartId}),(end:OperationalPoint {id:$neodash_q2EndId})
MATCH path =
    (start)
    ( (:OperationalPoint)-[:SECTION]-(op:OperationalPoint) ){1,10}
    (end)
UNWIND nodes(path) AS p
WITH p WHERE p <> start AND p <> end
MATCH (p)-[:NAMED]->(pName:OperationalPointName)
RETURN DISTINCT pName.name AS name, p.id AS __ID




