﻿# SCGraphTheory.Search

[![NuGet version (SCGraphTheory.Search)](https://img.shields.io/nuget/v/SCGraphTheory.Search.svg?style=flat-square)](https://www.nuget.org/packages/SCGraphTheory.Search/) 

The SCGraphTheory.Search NuGet package contains graph search algorithms that work against any graph type implementing the interfaces defined in the [SCGraphTheory.Abstractions](abstractions) package.

## Classic search algorithms

The `Classic` namespace contains (single-source) implementations of the breadth-first, depth-first (including limited and iterative deepening variants), Dijkstra, and A-star search algorithms, all conforming to a common interface - [ISearch<TNode,TEdge>](https://github.com/sdcondon/SCGraphTheory.Search/blob/main/src/Search/Classic/ISearch{TNode,TEdge}.cs).
They should be fairly intuitive to use.
Here are some example instantiations:

```csharp
var breadthFirst = new BreadthFirstSearch<MyNodeType, MyEdgeType>(
    source: mySpecificSourceNode,
    isTarget: n => n == mySpecificTargetNode);

var depthFirst = new DepthFirstSearch<MyNodeType, MyEdgeType>(
    source: mySpecificSourceNode,
    isTarget: n => n == mySpecificTargetNode);

var dijkstra = new DijkstraSearch<MyNodeType, MyEdgeType>(
    source: mySpecificSourceNode,
    isTarget: n => n.MyProperty == myDesiredValue,
    getEdgeCost: e => e.MyEdgeCost);

var aStar = new AStarSearch<MyNodeType, MyEdgeType>(
    source: myGraph.MyNodeIndex[0, 0],
    isTarget: n => n.Coords == targetCoords,
    getEdgeCost: e => e.MyEdgeCost,
    getEstimatedCostToTarget: n => EuclideanDistance(n.Coords, targetCoords));
```

Searches are executed step-by-step via the `NextStep()` method of the [ISearch<TNode,TEdge>](https://github.com/sdcondon/SCGraphTheory.Search/blob/main/src/Search/Classic/ISearch{TNode,TEdge}.cs) interface. This (as opposed to having to execute a search all the way to completion) is to maximise the flexibility with which potentially expensive searches can be executed. A `Complete()` extension method is defined though; which continuously calls `NextStep()` until the search completes.

Notes:
- All search algorithms expose details of visited edges via the `Visited` property.
This does add a little to the memory footprint that is overhead if you don't need this information.
The extra is relatively small though, since all of the algorithms require a quick way to determine if a node has already been visited anyway.
Using a Dictionary (as opposed to a HashSet) for this is a relatively minor addition.
- In the A* and Dijkstra implementations above, costs are float-valued. As of version 3.0.0, when targetting .NET 7 or greater, there are
alternatives for each that allow a user-specified (possibly non-numeric) cost type to be used.
- This namespace also contains `RecursiveDFS<TNode, TEdge>` - for consumers that don't need or want step-by-step execution.

## Local search algorithms

The `Local` namespace contains implementations of the (steepest-ascent) hill climb and simulated annealing search algorithms. They should also be fairly intuitive to use. Here are some example instantiations:

```csharp
var hillClimb = new HillClimb<MyNodeType, MyEdgeType, MyUtilityType>(
    source: mySpecificSourceNode,
    getUtility: n => n.MyUtilityProp);

var simulatedAnnealing = new SimulatedAnnealing<MyNodeType, MyEdgeType>(
    source: mySpecificSourceNode,
    getUtility: n => n.MyUtilityProp,
    annealingSchedule: t => Math.Max(1 - (.01f * t), 0));
```

Like the `Classic` searches, the local searches are executed step-by-step via a `NextStep()` method.
This (as opposed to having to execute a search all the way to completion) is to maximise the flexibility with which potentially expensive searches can be executed.

## And-or search algorithms

The `AndOr` namespace contains implementations (well, just a DFS for now) of search algorithms for "and-or" graphs.
The overall approach taken here is that a delegate is used to identify edges that actually represent a set of conjoined "and" edges (_all_ of which must ultimately lead to a target node in a search solution).
The actual edges are represented by the outbound edges of the node that the collection edge connects to.
Another way of looking at this is that we divide our graph into "or" nodes and "and" nodes.
See the [Specialized.AndOr namespace](https://github.com/sdcondon/SCGraphTheory.Search/blob/main/src/Search.TestGraphs/Specialized/AndOr) in the test graphs project for a couple of and-or graph examples.

```csharp
var andOrDFS = new AndOrDFS<MyBaseNodeType, MyBaseEdgeType>(
    source: mySourceNode,
    isTarget: IsTargetNode,
    isAndEdgeCollection: e => e is MyConjoinedEdgeCollectionType);
```