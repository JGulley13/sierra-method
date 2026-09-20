---
template:
  id: https://www.modelware.io/sierra/operational-analysis/verification
  name: "Verification"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Verification

Define verification cases that trace each requirement to the evidence used to confirm it.

```table-editor
---
columns: { this: { label: "Verification Case" } }
stylesheet:
  - selector: cell[col === "Status" && value]
    target: value
    style:
      padding: 4px 12px
      border-radius: 999px
      font-size: 12px
      font-weight: 600
      color: "#ffffff"
  - selector: cell[value === "Passed"]
    target: value
    style:
      background-color: "#10B981"
  - selector: cell[value === "Failed"]
    target: value
    style:
      background-color: "#DC2626"
  - selector: cell[value === "In Progress"]
    target: value
    style:
      background-color: "#F59E0B"
  - selector: cell[value === "Not Started"]
    target: value
    style:
      background-color: "#6B7280"
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix base: <https://www.modelware.io/sierra/base#> .
@prefix stakeholder: <https://www.modelware.io/sierra/stakeholder#> .
@prefix verification: <https://www.modelware.io/sierra/verification#> .

verification:VerificationCaseShape
    a sh:NodeShape ;
    sh:targetClass verification:VerificationCase ;
    sh:sparql [
      sh:message "A verification case marked Passed or Failed must have evidence populated." ;
      sh:select """
          PREFIX verification: <https://www.modelware.io/sierra/verification#>
          SELECT $this
          WHERE {
              $this verification:status ?s .
              FILTER(?s IN ("Passed", "Failed"))
              FILTER NOT EXISTS { $this verification:hasEvidence ?e }
            }
        """ ;
    ] ;

    sh:property [
        sh:path verification:verifies ;
        sh:name "Requirement" ;
        sh:class stakeholder:Requirement ;
        sh:minCount 1 ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path verification:method ;
        sh:name "Method" ;
        sh:in ( "Inspection" "Analysis" "Demonstration" "Test") ;
        sh:message "Method must be one of: Inspection, Analysis, Demonstration, Test" ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    sh:property [
        sh:path verification:status ;
        sh:name "Status" ;
        sh:in ( "Not Started" "In Progress" "Passed" "Failed") ;
        sh:message "Status must be one of: Not Started, In Progress, Passed, Failed" ;
        sh:maxCount 1 ;
        sh:order 3 ;
    ] ;
    sh:property [
        sh:path verification:hasEvidence ;
        sh:name "Evidence" ;
        sh:order 4 ;
    ] ;
    sh:property [
        sh:path base:expression ;
        sh:name "Pass/Fail Criteria" ;
        dash:editor dash:TextAreaEditor ;
        sh:maxCount 1 ;
        sh:order 5 ;
    ] ;
    sh:property [
        sh:path base:description ;
        sh:name "Description" ;
        dash:editor dash:TextAreaEditor ;
        sh:maxCount 1 ;
        sh:order 6 ;
    ] ;
    sh:property [
        sh:path base:category ;
        sh:name "Categories" ;
        sh:order 7 ;
    ] ;
    .
```
