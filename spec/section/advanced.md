## Advanced usage

### Multivalue processing

Aligned with the other RML specifications,
multivalue expression evaluation results are processed in sequence.
So, if a multivalue expression evaluation contains the multivalue `"a"`, `"b"`, and `"c"`,
the function is applied to each individual value in that order.

> **FIXME**
> Do we really want to handle multiple values that way?
> Because then we have the issue that some functions would need to have all values. Doing it with LV or CC to split collect is possible but having a function like `array:map` could do the multi-valued processing (or `array:cartesianProduct`) while keeping it nice for all teh others. However I understand it makes sense to follow the same logic as template.
>
> By keeping it that way we must have functionThatNeedsAllValues(gather("$.values[*]") with append strategy)
>


> FIXME
> How functions are executed also with the other comments such as the reuse of function, errors handling, functions parameters, type conversion and rdf natural datatype mapping.

> **FIXME**
> About type conversion (or not):
> What happens when we do a `divide(trim(reference "price"), 100)`, price gives a " 2200 " that encodes 22€
> should the function crash, do RML-FNML define that trim that return a xsd:string and divide that expects a xsd:number specify the processor should do a implicite conversion or should the user introduce an explicit conversion: `divide(parseNumber(trim(reference "price")), 100)`
> What happens with those types is unspecified to my knowledge?
>
> Previously with fno:Execution there was that **rdf intermediary** all the time but if that is removed then we can have native values and should at least specify something or we could also leave typing out and remove the requested fno:type ? 
>
> Are the executions models of FnO and RML-FNML compatible?

### Nested functions

As the values of a function are represented using [=Expression Map=]s,
it is possible to nest functions: you generate a term in a first function, and that term is used as an parameter value in a second function.

<p class="issue" data-format="markdown">
For an old example, see [RMLFNOTC0018](https://github.com/RMLio/rml-fno-test-cases/tree/master/RMLFNOTC0018-CSV).
</p>

> FIXME To be removed.
> Ahh but i understand now what meant that it was generating the fno:Execution.

<p class="issue" data-number="3" data-format="markdown">
For now, it is unclear how to handle a nested function where that nested Triples Map contains a join condition.
</p>

> FIXME Probably remove as well.

<aside class="example" id="example-nested-function" title="usage of nested function">
<aside class="ex-mapping">

```turtle
@prefix grel: <http://users.ugent.be/~bjdmeest/function/grel.ttl#> .
@prefix rml: <http://w3id.org/rml/> .
@prefix ex: <http://example.com/> .

<#Person_Mapping>
    rml:logicalSource <#LogicalSource> ;
    rml:subjectMap <#SubjectMap> ;
    rml:predicateObjectMap <#NameMapping> .

<#NameMapping>
    rml:predicate ex:title ;
    rml:objectMap [
        rml:functionExecution <#Execution> ;
        rml:return grel:stringOut
    ] ;
    .

<#Execution> a rml:FunctionExecution ;
    rml:function grel:toUppercase ;
    rml:input
        [
            a rml:Input ;
            rml:parameter grel:valueParam ;
            rml:inputValueMap [
                rml:functionExecution <#Execution2> ; # Link to another function-valued expression map to nest functions
                rml:return grel:stringOut
            ]
        ] .

<#Execution2> a rml:FunctionExecution ;               # First, replace spaces with dashes from the `name` reference
    rml:function grel:string_replace ;
    rml:input
        [
            a rml:Input ;
            rml:parameter grel:valueParam ;
            rml:inputValueMap [
                rml:reference "name"
            ]
        ] ,
        [
            a rml:Input ;
            rml:parameter grel:param_find ;
            rml:inputValue " "
        ] ,
        [
            a rml:Input ;
            rml:parameter grel:param_replace  ;
            rml:inputValue "-"
        ] .
```

</aside>
</aside>

### Conditions

Conditions are a shortcut to make RML mappings more intuitive, but rely on existing FNML functionality.
It is a shortcut that is applied using the `rml:condition`: an additional ExpressionMap predicate.
To be able to use this shortcut, conforming mapping engines MUST support following functions:

> FIXME: as an additional ExpressionMap predicate it must be done by a sub-class so need to define one also or change the formulation in rml-core? 
> Would probably make sense to also mention it near where the extensions to ExpressionMap are defined.

- isNull
- isNotNull
- equals: see the [SPARQL specification for definition](https://www.w3.org/TR/sparql11-query/#OperatorMapping)
- notEquals: see the [SPARQL specification for definition](https://www.w3.org/TR/sparql11-query/#OperatorMapping)
- IF

<p class="note" title="Condition function definitions">
  isNotNull and IF are defined below, rest is an excercise for the reader.
  The actual FnO definitions are TODO.
</p>

> FIXME
> Definition of NULL not provided!
> With or without the nulls from the rml logical source?
>
> IF should be https://w3id.org/imec/idlab/function#IF or in the new list of MUST be supported functions?

<aside class="example" id="example-condition" title="usage of condition">
<aside class="ex-mapping">

```turtle
@prefix fns: <http://example.com/fns#> .
@prefix rml: <http://w3id.org/rml/> .
@prefix ex: <http://example.com/> .

<#Person_Mapping>
    rml:logicalSource <#LogicalSource> ;
    rml:subjectMap <#SubjectMap> ;
    rml:predicateObjectMap <#NameMapping> .

<#NameMapping>
    rml:predicate ex:title ;
    # A condition can be defined in any expression map
    rml:objectMap [
        # new predicate that links to a function-valued expression map,
        # that function MUST return a boolean
        rml:condition [
            rml:functionExecution [
                # isNotNull(parameter: X) / definition: X != NULL ? TRUE : FALSE ;
                rml:function fns:isNotNull ;
                rml:input [
                    # The parameter that is checked for NULL
                    rml:parameter fns:parameter ;
                    rml:inputValueMap [
                        rml:reference "name"
                    ]
                ]
            ] ;
            rml:return fns:boolOut # if fno:boolOut is the first specified return, this triple can be ommitted.
        ] ;
        # The actual expression used if the condition returns TRUE
        rml:constant "[a filled in title]"
    ] .
```
</aside>

This is actually a shortcut to the following

<aside class="ex-mapping">

```turtle
@prefix fns: <http://example.com/fns#> .
@prefix rml: <http://w3id.org/rml/> .
@prefix ex: <http://example.com/> .

<#Person_Mapping>
    rml:logicalSource <#LogicalSource> ;
    rml:subjectMap <#SubjectMap> ;
    rml:predicateObjectMap <#NameMappingExtended> .

<#NameMappingExtended>
    rml:predicate ex:title ;
    rml:objectMap [
        rml:functionExecution [
            # IF(bool: X, expression: Y)
            # Function definition: X === TRUE ? Y : NULL
            rml:function fns:IF ;
            rml:input [
                # = original condition function
                rml:parameter fns:boolParameter ;
                rml:inputValueMap [
                    rml:functionExecution [
                        rml:function fns:isNotNull ;
                        rml:input [
                            rml:parameter fns:parameter ;
                            rml:inputValueMap [
                                rml:reference "name"
                            ]
                        ]
                    ]
                ]
            ] , [
                # = original expression
                rml:parameter fns:expressionParameter ;
                rml:inputValueMap [
                    rml:constant "[a filled in title]"
                ]
            ]
        ] ;
    ] .
# Any custom function can be used,
# or nested functions (eg AND/OR),
# depending on what the engines support
```

</aside>
</aside>

#### Multivalue Conditions - example

Let's take following example data

```json
[
    {
        "conditionValue": [1, 0, 5],
        "values": ["a", "b", "c"]
    }
]
```

If we execute following RML mapping

<aside class="example" id="example-condition" title="usage of conditions in multivalues">
<aside class="ex-mapping">

```turtle
@prefix fns: <http://example.com/fns#> .
@prefix rml: <http://w3id.org/rml/> .
@prefix ex: <http://example.com/> .

<#Person_Mapping>
    rml:logicalSource <#LogicalSource> ;
    rml:subjectMap <#SubjectMap> ;
    rml:predicateObjectMap <#NameMapping> .

# Suggestion: add rml:condition predicate to expression map,
# and conforming mapping engines MUST support following functions:
# - isNull, isNotNull, equals, noEquals, IF
# (isNotNull and IF are defined below, rest is an excercise for the reader)
<#NameMapping>
    rml:predicate ex:id ;
    # A condition can be defined in any expression map
    rml:objectMap [
        # new predicate that links to a function-valued expression map,
        # that function MUST return a boolean
        rml:condition [
            rml:functionExecution [
                # notEquals(parameter: X, compared: Y) / definition: X != Y ? TRUE : FALSE ;
                rml:function fns:notEquals ;
                rml:input [
                    # The parameter that is checked
                    rml:parameter fns:parameter ;
                    rml:inputValueMap [
                        rml:reference "conditionValue"
                    ]
                ] , [
                    # The parameter that is compared to
                    rml:parameter fns:compared ;
                    rml:input "0"
                ]
            ] ;
            rml:return fns:boolOut # if fno:boolOut is the first specified return, this triple can be ommitted.
        ] ;
        # The actual expression used if the condition returns TRUE,
        # In this case a UUID generator function is used.
        rml:objectMap <#ANestedFunctionThatReturnsAUUIDv4> ;
    ] .
```

</aside>
</aside>

> FIXME
> A small note about return.
> Is there an order in the returns in the definition of the function yes but not in the returns of the function if i understands right. So returns still keep their oredr implicitly? Could it be extended for return asking for an index from teh list or return is really just about the return iri.
>

This will result in two triples, because in `conditionValue` you have two valid condition executions

```turtle
<subject> ex:id "df2c61cc-6fad-435d-aa95-10761840478b" .
<subject> ex:id "44d01ee9-448f-4804-acc8-3272939494b0" .
```
