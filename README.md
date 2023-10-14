# Aquilo

A project for representing recipes as Directed Acyclic Graphs

# Introduction

A recipe describes how to start with a set of ingredients and apply a sequence of operations to arrive at a final product. A Directed Acyclic Graph (DAG) is a mathematical concept with nodes and directed edges that connect nodes. This project aims to create a specification of nodes and edges to enable concise representation of a recipe as a DAG.

For example, a recipe for whipped cream is represented by the DAG:

```mermaid
%%{
  init: {
    'theme': 'base',
    'themeVariables': {
      'fontFamily': 'monospace'
    }
  }
}%%
flowchart LR
  I["360 grams heavy cream
100 grams sugar
 10 grams vanilla extract
1/8 tsp kosher salt"]
  C{"Combine
in bowl"}
  W{"Whip
with whisk
until thick"}
  T[Whipped Cream]
  I --> C;
  C --> W;
  W --> T;
  style I text-align:left
```
  
# Ingredients and Recipe Steps as Nodes with Properties

A DAG is an abstract concept defined in terms of graph nodes and directed edges. Defining additional structure on a DAG brings the abstract definition into the domain of food recipes. Four node types, each having a list of required and optional properties, enable representing both recipes and hints for formatting the recipe for visual display:
- Ingredient Nodes
- Operation Nodes
- Final Product Nodes
- Grouping Nodes

The following subsections describe each node type and associated properties. A property is a key/value pair used to systematically encode important aspects of the node. For example, every node must have a **uuid** property key with a [RFC4122](https://datatracker.ietf.org/doc/html/rfc4122) Universally Unique IDentifier value and the node may have a **css** property key with a string value encoding a Cascading Style Sheet declarations for formatting the node. The uuid is only used to identify the node in a single recipe, so the same ingredient in two different recipes will have different uuid values.

A node also has **ports** to indicate what flows in and out. An ingredient node will typically have a single **output** port. Operation nodes will have one or more initial **input** ports and usually one **output** port. For operations that require inputs during the operations, such as slowly streaming the ingredient in over time, will have a **during** port. The port name **uno** is used when a port list has a single entry.

An edge is a link between an **output** port and an **input** or **during** port.  Ports in an edge specification are named by `uuid.name` for the uuid of the **node** and the port **name** forming one side of the edge.

In the following sections,
- a **bold** property key indicates it is required
- a normal text property key is optional
- a _italic_ property value indicates it is an example of the value and will likely be different in an actual recipe DAG
- a normal text property value is the specific value for the node type

To allow for specific applications to attach additional properties or metadata to a node or edge, the **yaml** key is optional with a valid YAML encoded data.

## Ingredients as Source Nodes

There are four ingredients in the whipped cream example in the introduction. The ingredients are presented as a single **group** node to improve readability. An ingredient node represents a single item and has the following properties
- **type**: source
- **uuid**: _4B049226-5437-4724-B5F1-2336938850AE_
- **name**: _heavy cream_
- **unit**: _gram_
- **quantity**: _360_
- **output**: [_uno_]
- css: _null_
- description: _cold_
- upc: _742365-216855_
- ean: _0-742365-216855_
- e-number: _E100_
- yaml: _{version: 1.3.4}_

## Operations as Internal Nodes

Combine and Whip are two operations in the whipped cream example. Combine simple means to group the ingredients together into a common vessel, but Whip requires a tool and termination condition. Terminate conditions describe when the operation is complete. Part of this specification is a registry of available operations, vessels, tools, and termination conditions. The registry will grow over time to accommodate recipes that cannot be accurately represented using the available operations.

An operation node has the following properties
- **type**: operation
- **uuid**: _32DB627B-F311-4FC7-998F-86F382E08F98_
- **name**: _whip_
- **input**: [_uno_]
- **during**: []
- **output**: [_uno_]
- **terminate**: [_until.thick_]
- vessel: _null_
- tool: _whisk_
- css: _null_
- description: _vigorously_
- yaml: _{version: 1.3.4}_

### Example Operation with Two Output Ports

Separating an egg yields the whites and yolk as outputs.  

```mermaid
%%{
  init: {
    'theme': 'base',
    'themeVariables': {
      'fontFamily': 'monospace'
    }
  }
}%%
flowchart LR
  I[1 large egg]
  Y[yolk]
  W[white]
  S{Separate}
  I --> S;
  S --> Y;
  S --> W;
```

- **type**: operation
- **uuid**: _82C766EB-43C4-4E6E-8C22-5676387E0E12_
- **name**: _separate_
- **input**: [_uno_]
- **during**: []
- **output**: [_yolk_, _white_]
- **terminate**: []

The ports `82C766EB-43C4-4E6E-8C22-5676387E0E12.yolk` and `82C766EB-43C4-4E6E-8C22-5676387E0E12.white` distinguish the two outputs from this example separation operation.

## Final Products as Sink Nodes

A node with no outgoing edges in a DAG is called a sink and in a recipe graph the final product is a sink. A sink node has the following properties
- **type**: sink
- **uuid**: _F85C5E8B-690F-4E77-96FA-E4DDF4994509_
- **name**: _Whipped Cream_
- **input**: [_32DB627B-F311-4FC7-998F-86F382E08F98.uno_]
- css: _null_
- description: _enjoy_
- yaml: _{version: 1.3.4}_

## Grouping Nodes for Streamlined Display

The Whipped Cream example grouped four ingredients into a single node. A group node consists of two or more source nodes, has a single output port, and has the properties
- **type**: group
- **uuid**: _4A7D3B86-D3D8-4EB0-8D7E-EDB7DA16DC33_
- **intput**: [_7B01EB97-A974-48E1-BA13-E482D81ED5ED.uno_, _C33FB5E1-6B19-49E6-81B7-74260E989E5E.uno_]
- **output**: [uno]
- css: _null_
- description: _null_
- yaml: _{version: 1.3.4}_

# Directed Edges

A directed edge is an arrow from an **output** port to an **input** or **during** port of a different node with properties
- **type**: edge
- **uuid**: _40A45365-CA92-4E0B-A865-35A89F0D9FB2_
- **source**: _F3274656-E756-46DC-B083-814225BBBE40.uno_
- **destination**: _2305DE0D-7255-4EAB-AD29-D5FC11F236C1.uno_
- css: _null_
- description: _null_
- yaml: _{version: 1.3.4}_

# Registry of Operation Nodes Properties

One aspect of the project **Aquilo** is to gather a concise list of operation nodes that enable the representation of a wide range of recipes. During the initial development of Aquilo, the registry of operations and properties will likely change often as more people and recipes attempt to use Aquilo.

# YAML as a Common Exchange format for Recipe DAGs

YAML is a convenient format to encode the collection of nodes and edges for a recipe with additional information, such as source, author, and date.

# A Complete Example

