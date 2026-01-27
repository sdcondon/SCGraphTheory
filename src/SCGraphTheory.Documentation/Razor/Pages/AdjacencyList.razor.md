# SCGraphTheory.AdjacencyList

[![NuGet version (SCGraphTheory.AdjacencyList)](https://img.shields.io/nuget/v/SCGraphTheory.AdjacencyList.svg?style=flat-square)](https://www.nuget.org/packages/SCGraphTheory.AdjacencyList/) 

The SCGraphTheory.AdjacencyList NuGet package contains a mutable, in-memory adjacency list graph implementation that implements the interfaces defined in the [SCGraphTheory.Abstractions](abstractions) package, and can thus work with other packages that also use this abstraction - such as [SCGraphTheory.Search](search).

As I hope you'll agree from the examples below, it's very straightforward to use, and performs very well under search in the general case. 
All node/edge incidence determination is by direct reference - no dictionary look-ups etc.

## Usage Examples

**Directed graphs** are the simplest to use.
Create concrete node and edge classes derived from NodeBase&lt;TNode,TEdge&gt; and EdgeBase&lt;TNode,TEdge&gt; respectively, including whatever members you want.
Then instantiate a Graph&lt;TNode,TEdge&gt;, and `Add` node and edge instances to it.
Here's an example with a string-valued "Label" for each node and edge:

```csharp
using SCGraphTheory.AdjacencyList;

namespace MyDirectedGraph;

public class Node(string label) : NodeBase<Node, Edge>
{
    public string Label { get; } = label;
}

public class Edge(Node from, Node to, string label) : EdgeBase<Node, Edge>(from, to)
{
    public string Label { get; } = label;
}

public static class Program
{
    ...

    private static Graph<Node, Edge> MakeGraph()
    {
        Graph<Node, Edge> graph = new();

        Node node1 = new("node 1");
        graph.Add(node1);

        Node node2 = new("node 2");
        graph.Add(node2);

        graph.Add(new Edge(node1, node2, "edge 1-2"));
        ...
    }
}
```

**Undirected graphs** take a *little* more effort, though there's a handy `UndirectedEdgeBase` class to do most of the work for you.
Here's an example with a direction-ignorant settable edge data property:

```csharp
using SCGraphTheory.AdjacencyList;

namespace MyUndirectedGraph;

public class Node : NodeBase<Node, Edge>
{
    // No node properties for this one - which is fine.
    // We still need a concrete class derived from NodeBase, though.
}

public class Edge : UndirectedEdgeBase<Node, Edge>
{
    private string myEdgeProp;

    // Note the delegate passed to base ctor here - which is for constructing
    // the reverse edge of this new edge - and note that it calls the other
    // constructor of this class (see below).
    public Edge(Node from, Node to, string myEdgeProp)
        : base(from, to, r => new(r, myEdgeProp))
    {
        this.myEdgeProp = myEdgeProp;
    }

    // This ctor is to construct an edge whose reverse already exists - note that
    // it calls the other base class ctor to one called by the ctor above. Also note
    // that this is private - we only need to invoke it in the lambda above.
    private Edge(Edge reverse, string myEdgeProp)
        : base(reverse)
    {
        this.myEdgeProp = myEdgeProp;
    }

    public string MyEdgeProp
    {
        get => myEdgeProp;
        set
        {
            myEdgeProp = value;
            Reverse.myEdgeProp = value;
        }
    }
}

public static class Program
{
    ...

    public static Graph<Node, Edge> MakeGraph()
    {
        Graph<Node, Edge> graph = new();

        Node node1 = new();
        graph.Add(node1);

        Node node2 = new();
        graph.Add(node2);

        graph.Add(new Edge(node1, node2, "A"));
        ...
    }
}
```

*Note that `UndirectedEdgeBase` still conforms to the `IEdge<TNode, TEdge>` interface, so each undirected edge actually consists of a pair of edge objects.
If we really wanted a single object on the heap for an undirected edge, we *could* make the IEdges value types that refer to the single ref type "edge".
The extra complexity and resulting caveats (edge structs as IEdge will be boxed, need to be careful with mutability, etc) mean that it's not something I've bothered exploring thus far.
See the [Abstractions](abstractions) package docs for more on directedness in SCGraphTheory*

Finally, here's an example with "undirected" edges with a **direction-specific edge data property** (reverse edge negates the value of the property).
Obviously its significant that `int` is a value type - solution would be a little more complex with a mutable reference type.

```csharp
using SCGraphTheory.AdjacencyList;

namespace MyUndirectedGraph2;

public class Node : NodeBase<Node, Edge>
{
}

public class Edge : UndirectedEdgeBase<Node, Edge>
{
    private int myEdgeProp;

    public Edge(Node from, Node to, int myEdgeProp)
        : base(from, to, r => new Edge(r, -myEdgeProp))
    {
        this.myEdgeProp = myEdgeProp;
    }

    private Edge(Edge reverse, int myEdgeProp)
        : base(reverse)
    {
        this.myEdgeProp = myEdgeProp;
    }

    public int MyEdgeProp
    {
        get => myEdgeProp;
        set
        {
            myEdgeProp = value;
            Reverse.myEdgeProp = -value;
        }
    }
}

public static class Program
{
    ...

    public static Graph<Node, Edge> MakeGraph()
    {
        Graph<Node, Edge> graph = new Graph<Node, Edge>();

        Node node1 = new();
        graph.Add(node1);

        Node node2 = new();
        graph.Add(node2);

        graph.Add(new Edge(node1, node2, 1));
        ...
    }
}
```

## Notes

* **"But this isn't an adjacency list representation..":** Yes, it is. Your algorithm textbook won't lumber itself with the conventions of object orientation, but .NET gives us them, and we should use the tools that we are given. If it helps, think of the node objects as the keys of a direct address table that points at each list.