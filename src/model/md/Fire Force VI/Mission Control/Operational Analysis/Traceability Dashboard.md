---
ontology: https://fireforce6.github.io/mission-control/bundle
---

# Traceability Dashboard

## Conformance: Verified Cases Cite Evidence

Method rule (from the Sierra verification template): a verification case marked Passed or Failed must have evidence.

```table
---
stylesheet:
  - selector: cell[col === "Result" && value]
    target: value
    style:
      padding: 4px 12px
      border-radius: 999px
      font-size: 12px
      font-weight: 600
      color: "#ffffff"
  - selector: cell[col === "Result" && value === "Conforms"]
    target: value
    style:
      background-color: "#10B981"
  - selector: cell[col === "Result" && value === "Violates"]
    target: value
    style:
      background-color: "#DC2626"
---
PREFIX verification: <https://www.modelware.io/sierra/verification#>

SELECT ?Case ?Status ?Evidence ?Result
WHERE {
  ?Case a verification:VerificationCase .
  OPTIONAL { ?Case verification:status ?Status }
  OPTIONAL { ?Case verification:hasEvidence ?Evidence }
  BIND(IF(?Status IN ("Passed", "Failed") && !BOUND(?Evidence), "Violates", "Conforms") AS ?Result)
}
ORDER BY ?Case
```

All cases conform. The query would flag a case reported as Passed or Failed with nothing behind it. VC2 has no evidence but is still Not Started, so it would become a violation if someone set it to Passed before attaching evidence.

## Near Miss: Capabilities One Link from Realised

A capability is realised when it is required by an Objective, assigned to an Entity, and described by a Process. Near misses meet 2 of the 3 criteria.

```r
include('src/method/r/utils.r')

result <- query("
  PREFIX mission: <https://www.modelware.io/sierra/mission#>
  PREFIX entity: <https://www.modelware.io/sierra/entity#>
  PREFIX process: <https://www.modelware.io/sierra/process#>
  SELECT ?capability ?objective ?entity ?process
  WHERE {
    ?capability a mission:Capability .
    BIND(IF(EXISTS { ?o mission:requires ?capability }, 1, 0) AS ?objective)
    BIND(IF(EXISTS { ?capability entity:isAssignedTo ?e }, 1, 0) AS ?entity)
    BIND(IF(EXISTS { ?p process:describes ?capability }, 1, 0) AS ?process)
  }
  ORDER BY ?capability
")

labels  <- display_label(NULL, result$capability)
obj     <- as.integer(result$objective)
ent     <- as.integer(result$entity)
proc    <- as.integer(result$process)
score   <- obj + ent + proc
missing <- ifelse(obj == 0, "Objective", ifelse(ent == 0, "Entity", "Process"))
near    <- score == 2

bar_chart(labels, score, "Capability Realisation (criteria met, of 3)")
display(paste0("<p><b>Near misses:</b> ", paste0(labels[near], " (needs ", missing[near], ")", collapse = ", "), "</p>"))
```

## Orphan: Unverified Requirements

**Question:** Which requirements have no evidence that they are met, and which of those matter most?

**Evidence:** Requirements with no verification case, sorted by priority.

```table
---
stylesheet:
  - selector: cell[col === "Priority" && value === "High"]
    target: value
    style:
      color: "#DC2626"
      font-weight: 600
---
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX base: <https://www.modelware.io/sierra/base#>
PREFIX stakeholder: <https://www.modelware.io/sierra/stakeholder#>
PREFIX verification: <https://www.modelware.io/sierra/verification#>

SELECT ?Requirement ?Description ?Priority ?StatedBy
WHERE {
  ?Requirement a stakeholder:Requirement .
  FILTER NOT EXISTS { ?vc verification:verifies ?Requirement }
  OPTIONAL { ?Requirement base:description ?Description }
  OPTIONAL { ?Requirement base:priority ?Priority }
  OPTIONAL { ?stakeholder stakeholder:states ?Requirement . ?stakeholder rdfs:label ?StatedBy }
}
ORDER BY (IF(?Priority = "High", 0, IF(?Priority = "Medium", 1, 2))) ?Requirement
```

**Interpretation:** 7 of 10 requirements (R1, R3, R4, R5, R6, R7, R10) have no verification case, so the model cannot show they will be met. Three of them are High priority: R1 Real-Time Map, R3 AI Warden Interface and R5 Historical Views. Every requirement the Operator stated is unverified, so the main user of the system has no verification coverage at all. Write verification cases for R1, R3 and R5 first, then the Medium-priority R4 and R7.

## Coverage: Stakeholders × Verification Cases

```compose
template: https://www.modelware.io/sierra/analysis/trace-coverage
title: "Stakeholders × Verification Cases"
rowType: https://www.modelware.io/sierra/stakeholder#Stakeholder
columnType: https://www.modelware.io/sierra/verification#VerificationCase
path: "stakeholder:states/^verification:verifies"
```

## View Graph: Verification Trace

Stakeholders (yellow) state requirements (cyan), which are verified by cases (blue) backed by evidence (grey). Requirements with no verification case are red and attached to the Unverified node.

```graph
---
layout:
  mode: force
  running: true
  fit: true
  padding: 24
  force:
    repulsion: 4200
    linkDistance: 110
    springStrength: 0.006
    gravity: 0.0012
    damping: 0.90
    maxSpeed: 4
stylesheet:
  - selector: node
    style:
      fill: lightgrey
      stroke: grey
      stroke-width: 1
  - selector: node [value.includes("/stakeholders#")]
    style:
      fill: yellow
  - selector: node [value.includes("/requirements#")]
    style:
      fill: cyan
  - selector: node [value.includes("/verifications#")]
    style:
      fill: "#4a90d9"
  - selector: node [value.includes("Unverified") || outgoing.some(e => e.target.value.includes("Unverified"))]
    style:
      fill: "#DC2626"
---
PREFIX stakeholder: <https://www.modelware.io/sierra/stakeholder#>
PREFIX verification: <https://www.modelware.io/sierra/verification#>

CONSTRUCT {
  ?stakeholder stakeholder:states ?req .
  ?vc verification:verifies ?req .
  ?vc verification:hasEvidence ?evidence .
  ?req <urn:sierra:gap> ?gap .
}
WHERE {
  { ?stakeholder stakeholder:states ?req }
  UNION
  {
    ?vc verification:verifies ?req .
    OPTIONAL { ?vc verification:hasEvidence ?evidence }
  }
  UNION
  {
    ?req a stakeholder:Requirement .
    FILTER NOT EXISTS { ?x verification:verifies ?req }
    BIND(<urn:sierra:Unverified> AS ?gap)
  }
}
```
