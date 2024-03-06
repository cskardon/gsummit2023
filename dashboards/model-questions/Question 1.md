# Question 1: What is the route from Operational Point X to Operational Point Y?

> What’s the quickest way to get a repair crew from Technical Services to a given Switch?

## Introduction

We've already seen an example between two stations - Malmö and London Paddington.

```cypher
MATCH 
    (:OperationalPointName {name:'Malmö central'})<-[:NAMED]-(malmo:OperationalPoint),
    (:OperationalPointName {name:'PAD London Paddington'})<-[:NAMED]-(paddington:OperationalPoint)
WITH 
    malmo, paddington
MATCH shortest = shortestPath( (malmo)-[:SECTION*]-(paddington) )
RETURN shortest
```

And using APOC to get the route based on the journey length:

```cypher
MATCH 
    (:OperationalPointName {name:'Malmö central'})<-[:NAMED]-(malmo:OperationalPoint),
    (:OperationalPointName {name:'PAD London Paddington'})<-[:NAMED]-(paddington:OperationalPoint)
WITH 
    malmo, paddington
CALL apoc.algo.dijkstra(malmo, paddington, 'SECTION', 'sectionlength') YIELD path, weight
RETURN path, weight;
```

So we know _how_ to query the database, but how do we represent this in NeoDash? 

## Plugins

First we need to have a new dashboard, and turn on 'Advanced Visualisations' and 'Report Actions'.

To do that, we need to open the Plugins dialog, by pressing the puzzle 🧩 icon in the top right of the dashboard:

<img alt="Picture showing the Plugin dialog icon, with a red arrow pointing at it" src="https://raw.githubusercontent.com/cskardon/gsummit2023/main/images/dashboard/question1/Question1_PluginButton.png">

On that dialog, we need to select the Advanced Visualisations and Report Actions plugins (as indicated below).

<img alt="Picture showing the Plugin dialog, with a red arrow pointing at the Advanced Visualisations select check box, and another red arrow pointing at the Report Actions select check box." src="https://raw.githubusercontent.com/cskardon/gsummit2023/main/images/dashboard/question1/Question1_PluginSelect.png">

## Parameter Entry

First we need parameters, we don't want to hardcode the locations in, as our dashboard would have to be re-written for every possible combination. So, we'll first add a **Parameter Select** report. In it we're simply going to set a parameter (`startlocation`) to the value of a free text entry.

<img alt="Picture showing the Parameter select report in NeoDash with Free Text selected" src="https://raw.githubusercontent.com/cskardon/gsummit2023/main/images/dashboard/question1/Question1_ParameterSelect.png">

We will want to repeat this for the `endlocation` as well. This allows our users to just type in any station name.

> NB. Please note the case of the actual parameter name, that **NeoDash** shows, not what you _type_.

## Using the parameters to get the `OperationalPoint`s

Now we have a way for our business users to search for a particular location, we need to use that to find the actual `OperationalPoint` instances to find our route. Cypher wise that's pretty simple:

```cypher
MATCH (opn:OperationalPointName)<-[:NAMED]-(op:OperationalPoint)
WHERE opn.name CONTAINS $neodash_startlocation
RETURN opn.name AS name, op.id AS __ID
```

First, let's add that to a 'Table' report connected to our Neo4j database, then we'll go into what we're doing.

We've used a `WHERE` clause here to allow us to use `CONTAINS` - which means we don't have to spell out the entirety of a location to find it. We're `RETURN`ing the Name and the ID of the `OperationalPoint`.

But what's this convention of using `__` before the `ID`? That means NeoDash won't render it to the screen. You can see that now, if you save the query and type in 'Berlin' into our Parameter field, we get the names of the locations with 'Berlin' in them. 

We can't select which of those we want though - and that's something we would like our business users to do - so... we now 'edit' our report and on the bottom right, you'll see a 'sparkles' icon ✨

<img alt="Picture showing the Sparkles icon on the edit report page" src="https://raw.githubusercontent.com/cskardon/gsummit2023/main/images/dashboard/question1/Question1_ReportSelect.png">

Once opened, we want to choose 'Cell select' and type in `name` (or whatever you called your column).

Then we're going to set _another_ parameter - in this case we've called it `startId` - though - you can choose whatever you want that makes sense to you. 

Finally we choose what we want to set that parameter to, and in this case it's `__ID`.

<img alt="Picture showing the add actions page" src="https://raw.githubusercontent.com/cskardon/gsummit2023/main/images/dashboard/question1/Question1_ReportActions.png">

If we save and go back to the report, we can see the report now shows the names of the `OperationalPoint`s as buttons that are selectable.

We need to repeat this for the End ID.

**STRETCH GOAL** This is case sensitive - can you think how to make it case insensitive? So you can just search for 'malmö' for example.

## Render the route as a Graph

We've done all the setup work now, and we can render the output as a Graph!

Add a new Report, and this time, select 'Graph' as the type, keeping the database as Neo4j.

Cypher wise - we're going to render the following:

```cypher
MATCH 
    (start:OperationalPoint {id:$neodash_startId}),(end:OperationalPoint {id:$neodash_endId})
CALL apoc.algo.dijkstra(start, end, 'SECTION', 'sectionlength') YIELD path, weight
RETURN path, weight;
```

We're using the parameters we've selected from the options and rendering them to the screen.

<img alt="Picture showing the Graph view" src="https://raw.githubusercontent.com/cskardon/gsummit2023/main/images/dashboard/question1/Question1_GraphView.png">

## Render the route as a Map

The Graph output is OK, but as we're actually talking about points on a map, can we render the results on a map?

Yes! We turned on the 'Advanced Visualisations' earlier, and that gets us access to the Map view, so let's add another report, and this time select 'Map' and render exactly the same Cypher as before.

Now when this is saved you should see a map output of the results.

<img alt="Picture showing the Map view" src="https://raw.githubusercontent.com/cskardon/gsummit2023/main/images/dashboard/question1/Question1_MapView.png">