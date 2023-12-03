# New syntax `Trailing Block` for constructing tree-structure objects

Dear friends: in this topic I am proposing a new syntax named `Trailing Block` or `Trailing Compound Statement`, for the purpose of constructing or instantiating complex objects with tree structure. My vison for this proposal is that with this syntax sugar, Python can natively speek complex data structure as conveniently as XML or other declarative DSL does.

The new `Trailing Block` syntax consists of three parts:
+ new special type `Trailer`,
+ new dunder method `__trailing__`, named `Trailing Method`,
+ new block syntax & variable scope when constructing objects.

Let me take an example to explain my idea. With present python syntax, to construct a tree, usually you have to the following codes:

```
class Node:
    def __init__(self, *args, **kwargs):
        self.child_nodes = []
        ...

    def add_child(self, node):
        ...
    
root_node = Node(...)
child_node = Node(...)
grandchild_node0 = Node(...)
grandchild_node1 = Node(...)

child_node.add_child(grandchild_node0)
child_node.add_child(grandchild_node1)
root_node.add(child_node)
```

The `add_child` method is used repeatly for every level of the tree. With the proposed syntax, such codes are placed in `__trailing__` method, as


```
class Node:
    def __init__(self, *args, **kwargs):
        self.child_nodes = []
        ...

    def add_child(self, node):
        ...

    def __trailing__(self, *args, **kwargs):
        """ trailing method """
        # codes that called whenever a node is constructed
        ...
```

`Trailer` refers to the type of object that with a `__trailing__` dunder method. A `Trailer` object, right after constructed, can be followed by an indent code block, named `trailing block`, such as

```
child_node = Node(...):
    # trailing block
    Node(...)  # grand node 0
    Node(...)  # grand node 1
```

What connects `__trailing__` method and `Trailing Block` is that local variables of the trailing block are passed to the arguments of `__trailing__` method:

+ new `Trailer` instances, whether assigned to varibles or not, are passed as the positional arguments of  `__trailing__` method;
+ any assigned local variables are passed to the keyword arguments of  `__trailing__` method, just like passing the result of builtin method `locals`.


In the example above, the 2 newly constructed `Node` instances are passed to the `__trailing__` method of `Node` class. And we complete the class definition as

```
class Node:
    def __init__(self, *args, **kwargs):
        self.child_nodes = []
        ...

    def add_child(self, node):
        ...

    def __trailing__(self, *args: Trailer, **kwargs):
        """ trailing method """
        for node in args:
            if isinstance(node, Node):  # it is OK to use the class itself here
                self.add_child(node)
```

And thus the tree constructing can be greatly simplified:
```
root = Node(...):
    Node(...):  # child node
        Node(...)  # grand node 0
        Node(...)  # grand node 1
``` 

Further more, you can set attributes for each node with clear syntax

```
root = Node(...):
    # attributes
    red_or_black = True

    Node(...):  # child node
        # function attributes
        def on_walk(...):
            ...

        Node(...)  # grand node 0
        Node(...)  # grand node 1
``` 

and define attribute-setting codes in `__trailing__` method:

```
class NodeTrailingType(TypedDict):
    """ pep-0692: use `TypeDict` for kwargs typing """
    red_or_black: Bool
    on_walk: Callable


class Node:
    def __init__(self, *args, **kwargs):
        self.child_nodes = []
        self.red_or_black: Bool = True
        self.on_walk: Callable = None
        ...

    def add_child(self, node):
        ...

    def __trailing__(self, *args: Trailer, **kwargs: NodeTrailingType):
        """ trailing method """
        for node in args:
            if isinstance(node, Node):  # it is OK to use the class itself here
                self.add_child(node)

        # set attributes
        self.red_or_black = kwargs['red_or_black']
        self.on_walk = kwargs['on_walk']
```

Some clarification:
+ Maybe `Trailing block` syntax seems to the `with` statement without `with` keyword itself. 
However, the purpose and what focus on are significantly different. `Trailing block` has nothing to 
do with ***defer*** usage. 

+ It does not conflict the Zen of Python: ***flat is better than nested***. The new syntax is designed specially to handle nested object.


I am eager to hear your comments about this idea :-)