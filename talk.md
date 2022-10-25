---
theme: "black"
---

# CIngle Point of Failure

## Dark side of automation awesomeness

---

### ToC

---

# $ whoami

- Sample Pipelines
- SPoFs
- 

---

# <3 CI/CD/CD

Malgré toutes les erreurs d'implémentation présentées CI/CD/CD reste _best practice_

---

# Sample Pipelines: CI

```mermaid
flowchart TD

Commit --> Push

subgraph B[Base]
    A[start] --> CI[npm ci]
    A --> E[ESLint]
    A --> J[jsonnetfmt]
    A --> Madge
    A --> Prettier
    CI --> T[TSC]
    T --> M[Mocha/NYC]
    M --> S[Stryker]

    S --> em{Email?}
    E --> em
    J --> em
    Madge --> em
    Prettier --> em
end

subgraph C[Container]
    C1 --> launch[Launch Services ...]
    C1[start] --> C2[Docker buildx]
    C1 --> C1b[Wait for services...]
    C2 --> C2a["Launch Container"]
    C1b --> C2a
    C2 --> C3['Wait for container']
    C1b --> C3
    C3 --> C4[Tests]
    C4 --> C5{Email?}
end

Push --> B
Push --> C

```

---

# Sample Pipelines: C Delivery

```mermaid
flowchart LR

Commit --> Push
Push --> Base --> D
Push --> Container --> D


subgraph D[Branche Principale]
    S[Semantic-Release]
    S --> T[Git Tag]
    S --> P[Package]
end

P --> R[(Package<br/>Registry)]

```

---

# Sample Pipelines: C Deployment

```mermaid
flowchart LR

Tag --> D
R[(Container<br/>Registry)]

subgraph D[Deployment]
    C1 --> launch[Launch Services ...]
    C1[start] --> C2[Docker buildx]
    C2 -->|push|R
    C1 --> C1b[Wait for services...]
    R -->|pull| C2a
    C2 --> C2a["Launch Container"]
    C1b --> C2a
    C2 --> C3['Wait for container']
    C1b --> C3
    C3 --> C4[Tests]
    C4 --> C5[Prod]
    R -->|pull|C5
    C5 --> C5a[Post-Deploy Tests]
    C5a --> C6{Alert?}
end

```

---

# Single Point of Failures

- Drone (CI / CD / CD / Secrets)
- Artifacts Storage: Verdaccio/Artifactory
- Docker Registry

---

