# go-like-generics-proposal
An alternative Go-like proposal for Go2 generics.
Based on the ideas of Burak Serdar here: https://groups.google.com/forum/m/#!topic/golang-nuts/eqaDEb9xtGM

# Generic type parameter specification.

Generic types can be specified with a like clause . 
A like clause mentions one or more concrete types 
that the parameterized type must match. 
The concrete type in like clause can be a primitive type, an interface or a struct type.

# Proposal (WIP)

A type template is declared like this:

```
type T like (X,Y,...)
```

This means that any one of X, Y, ... can be substituted in place of
T. In effects, it is the intersection of the operations defined on X,
Y, ...

  * If X is a primitive type such as int, any type derived from int
    can be substituted.
  * If X is an interface type, any type implementing X can be substituted
  * If X is a struct type, any type implementing all the methods of X
    can be substituted
  * If X is an array type, map type, or channel type, then any
    array/map/channel type derived from X can be substituted.
    That is:
```
  type ArrType []int
```

    can be substituted for

```
  type T like []int
```
  but not for:

```
  type T like []int64
```

Based on the above, the following should also be meaningful:

   * map<string,T>: this would be a map template whose values are
   described by T.
   * []like int : An array of int-like values
   

Function template is declared like this:
```
  func F(a, b type T like (X, Y)) T
```
or if T is already defined as a type template:
```
  func F(a,b type T) T
```

## Struct and interface templates

The following syntax defines struct and interface templates:

```
type TemplateStruct like struct {
 ConcreteField int
 Field1 type T like(X,Y,...)
 Field2 type U
}
```
or
```
type TemplateInterface like interface {
  Func(int)
  FuncTemplate1(type T like(X,Y...)
  FuncTemplate2(type T)
 ...
}
```

To instantiate a template struct:

```
type MyStruct TemplateStruct {
  ConcreteField int // Template fields must be redeclared with concrete types
  Field1 X 
  Field2 UDerivative
   ...
}
```
Above syntax declares MyStruct type as an instance of the TemplateStruct.
That means you have to redefine all the fields defined in the TemplateStruct for MyStruct,
and you can add any additional fields.

There is no "instantiating a template interface". The compiler should
be able to deduce the types of methods of a template interface when a
concrete type is substituted in place of that template interface (is this
really true?)

With this, a linked list implementation may look like the following:

The template:
```
package linkedlist

type LinkedList like struct {
  head *like Node
}

type Node like struct {
  next *like Node
}

func (l *LinkedList) Add(in *Node) {
  ...
}
```
Usage:
```
type TheLinkedList linkedlist.LinkedList {
  head *TheNode
}

type TheNode linkedlist.Node {
  next *TheNode
  more stuff
}

var myLinkedList TheLinkedList
```

Reworking the graph/node/edge case from the official proposal:

Template:
```
type Node like interface {
   Edges() []Edge
}

type Edge like interface {
   Nodes() (Node,Node)
}

type Graph like struct {
   Nodes []*Node
}

func New(nodes []*Node) *Graph {
  return &Graph{Nodes:nodes}
}
```


The instantiation:
```
type MyNode struct {...}

func (m MyNode) Edges() []MyEdge {...}

type MyEdge  struct { ...}

func (e MyEdge) Nodes() (*MyNode,*MyNode) {...}

type MyGraph graph.Graph {
  Nodes []*MyNodes
}

x:=graph.New(MyGraph)(myNodes)
```

If a function signature contains a type template, the function itself is a template:

```
type T like(X,Y)

func TemplateFunc(input T) {...{
```

