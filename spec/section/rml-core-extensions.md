## Extension to Expression Maps {#rml-core-extensions}

RML-FNML extends the [[[RML-Core]]] <a data-cite="RML-Core#dfn-expression-map">Expression Map</a> definition by adding new properties.

An <a data-cite="RML-Core#dfn-expression-map">expression map</a> MUST have one of the following properties:
* 0 or 1 `rml:constant`, or
* 0 or 1 `rml:reference`, or
* 0 or 1 `rml:template`, or
* **0 or 1 `rml:functionExecution`**, or
* another property defined by a subclass of `rml:ExpressionMap`.

An expression map that specifies `rml:functionExecution` is termed a [=Function-valued Expression Map=]. A Function-valued Expression Map MUST have zero or one `rml:returnMap` property (or `rml:return` shortcut property).

Additionally, an expression map MAY specify 0 or 1 `rml:condition` property.

An <a data-cite="RML-Core#dfn-rml-processor">RML processor</a> conforming to RML-FNML MUST support the `rml:functionExecution` and `rml:condition` properties on an <a data-cite="RML-Core#dfn-expression-map">expression map</a>.
