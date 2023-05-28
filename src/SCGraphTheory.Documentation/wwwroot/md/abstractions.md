# SCGraphTheory.Abstractions

[![NuGet version (SCGraphTheory.Abstractions)](https://img.shields.io/nuget/v/SCGraphTheory.Abstractions.svg?style=flat-square)](https://www.nuget.org/packages/SCGraphTheory.Abstractions/) 

The SCGraphTheory.Abstractions NuGet package contains interfaces for the three fundamental elements of graph theory - [IGraph<TNode,TEdge>](https://github.com/sdcondon/SCGraphTheory.Abstractions/blob/main/src/Abstractions/IGraph{TNode,TEdge}.cs), [INode<TNode,TEdge>](https://github.com/sdcondon/SCGraphTheory.Abstractions/blob/main/src/Abstractions/INode{TNode,TEdge}.cs) and [IEdge<TNode,TEdge>](https://github.com/sdcondon/SCGraphTheory.Abstractions/blob/main/src/Abstractions/IEdge{TNode,TEdge}.cs) - to allow for graph algorithms that do not depend on a particular graph representation.

Example implementation and usage can be found in the separate [SCGraphTheory.AdjacencyList](https://github.com/sdcondon/SCGraphTheory.AdjacencyList) and [SCGraphTheory.Search](https://github.com/sdcondon/SCGraphTheory.Search) repositories, respectively.
Additional (test-focused) implementation examples can be found in the [TestGraphs library](https://github.com/sdcondon/SCGraphTheory.Search/tree/main/src/Search.TestGraphs) in SCGraphTheory.Search.
Notably:
- A very simple (though rather inefficient) [LINQ-powered immutable implementation](https://github.com/sdcondon/SCGraphTheory.Search/blob/main/src/Search.TestGraphs/LinqGraph.cs). Used for tests in the search algorithm package.
- A [square grid implementation using structs](https://github.com/sdcondon/SCGraphTheory.Search/blob/main/src/Search.TestGraphs/ValGridGraph%7BT%7D.cs). Involves no up-front heap allocations other than a 2D array of node values, but performs a little worse under search because lots of data gets moved around compared to a class-based implementation (including a little boxing - see notes, below). Included in search benchmarks project because I was interested in the performance impact.
- A bare-bones [adjacency matrix implementation](https://github.com/sdcondon/SCGraphTheory.Search/blob/main/src/Search.TestGraphs/AdjacencyMatrixGraph.cs). Doesn't actually feature in any tests - I was just curious to know what an adjacency matrix implementation of these interfaces could look like.
- Some [and-or graph implementations](https://github.com/sdcondon/SCGraphTheory.Search/tree/main/src/Search.TestGraphs/Specialized/AndOr) that also serve as general examples of graphs with more than one edge and node type, and of graphs with reference type nodes/edges that are lazily initialised as the graph is explored.

Notes:
* The fact that the IEdge abstraction has a "From" and a "To" doesn't make this abstraction unsuitable for undirected graphs.
Graph algorithms will generally traverse edges in a particular direction, making this a useful interface,
and while the AdjacencyList implementation doesn't do this (thus favouring low latency over low memory usage),
there's nothing stopping an implementation (with class-valued edges) from making the IEdge implementation a struct created from the "actual" edge,
depending on the current node - thus avoiding "duplicated" undirected edges on the heap.  
  
  Of course, the Edges property of IGraph returns IEdges, so necessarily should include both directions of an undirected edge - which could cause confusion.
However, it should be noted that this is also justified by what it facilitates for algorithms using the abstraction.
Consider Bellman-Ford, for example - which (iterates graph edges and) operates specifically on directed graphs.
By including both directions of an undirected edge, we allow algorithms such as these to be used against all graphs that implement this abstraction correctly.
In this way, treating the two directions of undirected edges separately is a form of normalisation.  
  
  (There's a justification based on Liskov here, too - which I'll get around to articulating at some point).  
  
  ..I should however probably expand a little on the "confusion" mentioned above, and in general the downside of this simplicity. Consider the act of building a
spanning tree for an undirected graph. Which direction of an edge is included would depend on where you start, which might not be ideal. There are of course ways
to deal with this. The addition of an edge-valued `Reverse` property, the addition of a property to indicate the "actual" underlying `Undirected` edge (which is
identical for reverse edges), and so on. See SCGraphTheory.AdjacencyList for an example of an undirected edge implementation with a Reverse property. The only cases
where this is really going to cause a "problem" is for algorithms that enumerate the edges of a graph directly (i.e. not starting from a particular node or nodes),
and operate *specifically* on undirected graphs. In such cases, you are likely to need to expand on the abstraction (again, probably using a `Reverse` and/or 
`Undirected` prop) in order to write the algorithm in the first place.  
  
  A final thought on this: one way of thinking about this abstraction is that it deals with edge traversals (which are inherently directed) rather than edges - and
it is down to the consumer to decide how the two concepts are related in their usage. This of course could result in a little extra work in certain cases, but (in
the author's humble opinion) in the vast majority of situations will not, and thus the resulting simplicity is worthwhile.
* The declaration of the edges collection of each node as an `IReadOnlyCollection<TEdge>` necessitates boxing by consumers of these interfaces when this collection is a value type. See [an alternative formulation in the benchmarks project of the SCGraphTheory.Search](https://github.com/sdcondon/SCGraphTheory.Search/tree/main/src/Search.Benchmarks/AlternativeAbstractions/TEdges) for more on this.
* Naming is hard:
  * Why `INode` and not `IVertex`? Simply because its shorter. I must confess that I am slightly regretting this one though..
  * Why call it "SCGraphTheory"? SC are my initials. I do worry that using my initials comes off as a little arrogant - but I'd argue that just calling it "GraphTheory.." would be more arrogant. And inventing a brand name would be tedious - and no doubt longer than SC.. 
