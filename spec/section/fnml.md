## RML-FNML Vocabulary Definition {#fnml}

This section defines the classes, properties, and constraints of RML-FNML.

<figure id="figure-diagram">
  <img src="resources/diagram.png" alt="FNML diagram" />
  <figcaption>Visual overview of how FNML introduced terms align with RML-Core and FnO</figcaption>
</figure>

### Function-valued Expression Map {#function-valued-expression-map}

A <dfn>Function-valued Expression Map</dfn> is an `rml:ExpressionMap` that generates values by executing an [[FnO]] function and capturing one of its declared return values.

* A [=Function-valued Expression Map=] **MUST** have exactly one `rml:functionExecution` property.
* A [=Function-valued Expression Map=] **MAY** have zero or one `rml:returnMap` property (or `rml:return` shortcut property).

| Property | Domain | Range | Shortcut |
| :--- | :--- | :--- | :--- |
| `rml:functionExecution` | `rml:ExpressionMap` | `rml:FunctionExecution` | — |
| `rml:returnMap` | `rml:ExpressionMap` | `rml:ReturnMap` | `rml:return` |

#### Return Map (`rml:ReturnMap`)

A <dfn>Return Map</dfn> (`rml:ReturnMap`) is a [=Term Map=] that **MUST** generate an IRI specifying the [=Return=] of the referenced [=Function=].
The property `rml:return` is the [=constant expression shortcut property=] of `rml:returnMap`.

<p class="note" title="Return Map Reference Target">
A Return Map (or its <code>rml:return</code> shortcut) <strong>MUST</strong> generate the IRI of the <code>fno:Output</code> instance (such as <code>grel:stringOut</code>), and <strong>MUST NOT</strong> refer to the <code>fno:predicate</code> property IRI (such as <code>grel:stringOutput</code>).
</p>

If `rml:returnMap` and `rml:return` are omitted, the first return value declared in the `fno:returns` list of the referenced [=Function=] is used by default.

### Conditional Expressions (`rml:condition`) {#conditions}

An `rml:ExpressionMap` MAY specify an <dfn>rml:condition</dfn> property to conditionally govern whether RDF terms are generated during mapping evaluation.

| Property | Domain | Range | Shortcut |
| :--- | :--- | :--- | :--- |
| `rml:condition` | `rml:ExpressionMap` | `rml:FunctionExecution` | — |

* The value of `rml:condition` **MUST** be an `rml:FunctionExecution`.
* The referenced function execution **MUST** return an `xsd:boolean` value.

#### Evaluation Semantics

During the evaluation of an expression map for a logical iteration:
1. The condition function execution is evaluated first.
2. If the condition evaluates to `true` (`"true"^^xsd:boolean`), the expression map proceeds with its normal evaluation and term generation.
3. If the condition evaluates to `false` (`"false"^^xsd:boolean`), evaluates to `null`, or produces an error, the expression map generates no value for that iteration, and term generation is omitted.

The `rml:condition` property is functionally equivalent to wrapping the expression map in a ternary conditional function execution (`rml-fn:IF`).

An expression map specifying `rml:condition`:
```turtle
[
    rml:condition <#ConditionExecution> ;
    rml:constant "[Default Title]"
]
```

is equivalent to:
```turtle
[
    rml:functionExecution [
        rml:function rml-fn:IF ;
        rml:input [
            rml:parameter rml-fn:boolParam ;
            rml:inputValueMap [ rml:functionExecution <#ConditionExecution> ]
        ] , [
            rml:parameter rml-fn:expressionParam ;
            rml:inputValue "[Default Title]"
        ]
    ]
]
```

### Function Execution (`rml:FunctionExecution`) {#function-execution}

A <dfn>Function Execution</dfn> (`rml:FunctionExecution`) is a construct that represents an invocation of a [[FnO]] [=Function=].
It binds concrete values to the [=Function=]'s [=Parameter=]s via [=Input=]s, and specifies the target function to execute via an [=Function Map=].

* An `rml:FunctionExecution` **MUST** have exactly one [=Function Map=] specified via the `rml:functionMap` property, or via the `rml:function` shortcut property.
* An `rml:FunctionExecution` **MAY** have zero or more [=Input=] properties specified via the `rml:input` property.

| Property | Domain | Range | Shortcut |
| :--- | :--- | :--- | :--- |
| `rml:functionMap` | `rml:FunctionExecution` | `rml:FunctionMap` | `rml:function` |
| `rml:input` | `rml:FunctionExecution` | `rml:Input` | — |

An <dfn>Function Map</dfn> (`rml:FunctionMap`) is a [=Term Map=] that **MUST** generate an IRI. That IRI specifies the referenced [[FnO]] [=Function=].
The property `rml:function` is the [=constant expression shortcut property=] of `rml:functionMap`.

#### Execution Reuse

An `rml:FunctionExecution` can be identified by an IRI or a blank node:

* **Named Execution (IRI)**: When an `rml:FunctionExecution` is identified by an IRI, conforming RML processors **MUST** treat this as a shared, reusable function execution. If multiple expression maps (or multiple triples maps) reference the same execution IRI for a given logical iteration, the processor **MUST** evaluate the execution once and reuse the results across all referencing locations, rather than executing the function multiple times.
* **Anonymous Execution (Blank Node)**: When identified by a blank node, the execution is scoped locally to its parent expression map.

#### Dynamic Function Selection (`rml:functionMap`)

In the vast majority of mappings, the function to execute is known statically at mapping design time, in which case the constant shortcut `rml:function` is used.

However, because `rml:functionMap` is a full [=Term Map=], it also supports dynamic function selection. When specified using an `rml:reference` or `rml:template`, the target function IRI can be determined dynamically per iteration from data values in the logical source (e.g., dynamically choosing between different hashing, translation, or parsing functions specified in a source column).

### Inputs and Parameter Mapping (`rml:Input`) {#inputs}

An <dfn>Input</dfn> (`rml:Input`) is a construct that provides a pairwise connection between a [[FnO]] [=Parameter=] and an input value generated from the logical iteration.

* An `rml:Input` **MUST** have exactly one [=Parameter Map=] specified via the `rml:parameterMap` property, or via the `rml:parameter` shortcut property.
* An `rml:Input` **MUST** have exactly one input value map specified via the `rml:inputValueMap` property, or via the `rml:inputValue` shortcut property.

| Property | Domain | Range | Shortcut |
| :--- | :--- | :--- | :--- |
| `rml:parameterMap` | `rml:Input` | `rml:ParameterMap` | `rml:parameter` |
| `rml:inputValueMap` | `rml:Input` | `rml:TermMap` | `rml:inputValue` |

A <dfn>Parameter Map</dfn> (`rml:ParameterMap`) is a [=Term Map=] that **MUST** generate an IRI. That IRI specifies the referenced [[FnO]] [=Parameter=].
The property `rml:parameter` is the [=constant expression shortcut property=] of `rml:parameterMap`.

<p class="note" title="Parameter Map Reference Target">
A Parameter Map (or its <code>rml:parameter</code> shortcut) <strong>MUST</strong> generate the IRI of the <code>fno:Parameter</code> instance (such as <code>grel:valueParam</code>), and <strong>MUST NOT</strong> refer to the <code>fno:predicate</code> property IRI (such as <code>grel:valueParameter</code>).
In FnO, the <code>fno:Parameter</code> IRI denotes the parameter entity itself within the function declaration, whereas <code>fno:predicate</code> is used only within generic RDF execution descriptions. RML-FNML relies on the parameter entity IRI to remain independent of RDF serialization formats.
</p>

#### Dynamic Parameter Mapping (`rml:parameterMap`)

In standard mappings, the parameter to which an input value is bound is fixed and declared using the `rml:parameter` constant shortcut.

Using an explicit, dynamic `rml:ParameterMap` allows mappings to bind parameters dynamically when working with semi-structured data sources containing key-value pairs or dynamic attributes, where the target parameter IRI is determined at runtime from the data record itself.

#### Input Value Mapping (`rml:inputValueMap`)

The property `rml:inputValueMap` links an `rml:Input` to a [=Term Map=] that generates the value to be bound to the referenced [=Parameter=].
This [=Term Map=] refers to values from the iteration of the enclosing triples map's logical source, handled just like any regular [=Term Map=] (e.g. using `rml:reference`, `rml:template`, or nested `rml:functionExecution`).
The property `rml:inputValue` is the [=constant expression shortcut property=] of `rml:inputValueMap`.

### Complete Mapping Example

The following example shows an object map that executes `grel:toUppercase` on the `name` reference, capturing the `grel:stringOut` return, and using `rml:condition` to only generate terms when `name` is not null:

<aside class="example" id="example-complete-mapping" title="Function-valued Expression Map with Condition">
<aside class="ex-mapping">

```turtle
<#Person_Mapping>
    rml:logicalSource <#LogicalSource> ;
    rml:subjectMap [
        rml:template "http://example.com/person/{id}"
    ] ;
    rml:predicateObjectMap [
        rml:predicate ex:title ;
        rml:objectMap [
            # Function-valued Expression Map
            rml:functionExecution <#UppercaseExecution> ;
            rml:return grel:stringOut ;
            # Condition shortcut: only generate when name is not null
            rml:condition [
                rml:function rml-fn:isNotNull ;
                rml:input [
                    rml:parameter rml-fn:param ;
                    rml:inputValueMap [ rml:reference "name" ]
                ]
            ]
        ]
    ] .

<#UppercaseExecution> a rml:FunctionExecution ;
    rml:function grel:toUppercase ;
    rml:input [
        a rml:Input ;
        rml:parameter grel:valueParam ;
        rml:inputValueMap [ rml:reference "name" ]
    ] .
```

</aside>
</aside>
