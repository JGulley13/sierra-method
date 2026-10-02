---
template:
  id: https://www.modelware.io/sierra/analysis/trace-coverage
  name: "Trace Coverage"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
    - id: title
      type: string
      defaultValue: "Trace Coverage"
    - id: rowType
      type: iri
      required: true
    - id: columnType
      type: iri
      required: true
    - id: path
      type: string
      required: true
---
## ${title}

How many times each row element reaches each column element along `${path}`. A 0 is a trace that does not exist.

```matrix
---
rowColumnLabel: Row / Column
stylesheet:
  - selector: cell [Number(value) === 0]
    style:
      background-color: "#fde2e2"
  - selector: cell [Number(value) > 0]
    style:
      background-color: lightgreen
---
PREFIX base: <https://www.modelware.io/sierra/base#>
PREFIX mission: <https://www.modelware.io/sierra/mission#>
PREFIX stakeholder: <https://www.modelware.io/sierra/stakeholder#>
PREFIX entity: <https://www.modelware.io/sierra/entity#>
PREFIX process: <https://www.modelware.io/sierra/process#>
PREFIX verification: <https://www.modelware.io/sierra/verification#>

SELECT ?row ?column (COALESCE(?n, 0) AS ?value)
WHERE {
  ?row a <${rowType}> .
  ?column a <${columnType}> .

  OPTIONAL {
    SELECT ?row ?column (COUNT(*) AS ?n)
    WHERE {
      ?row ${path} ?column .
    }
    GROUP BY ?row ?column
  }
}
ORDER BY ?row ?column
```

### Untraced rows

Rows with no trace to any column element. These are the all-zero rows of the matrix above.

```list
---
stylesheet:
  - selector: item
    target: marker
    style:
      color: "#b00020"
---
PREFIX base: <https://www.modelware.io/sierra/base#>
PREFIX mission: <https://www.modelware.io/sierra/mission#>
PREFIX stakeholder: <https://www.modelware.io/sierra/stakeholder#>
PREFIX entity: <https://www.modelware.io/sierra/entity#>
PREFIX process: <https://www.modelware.io/sierra/process#>
PREFIX verification: <https://www.modelware.io/sierra/verification#>

SELECT ?row
WHERE {
  ?row a <${rowType}> .
  FILTER NOT EXISTS {
    ?row ${path} ?column .
    ?column a <${columnType}> .
  }
}
ORDER BY ?row
```
