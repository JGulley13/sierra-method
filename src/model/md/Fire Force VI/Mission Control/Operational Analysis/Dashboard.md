---
ontology: https://fireforce6.github.io/mission-control/bundle
---

```compose
template: https://www.modelware.io/sierra/operational-analysis/dashboard
```

```compose
template: https://www.modelware.io/sierra/analysis/trace-coverage
title: "Entities × Activities"
rowType: https://www.modelware.io/sierra/entity#Entity
columnType: https://www.modelware.io/sierra/process#Activity
path: "^process:isAllocatedTo"
```