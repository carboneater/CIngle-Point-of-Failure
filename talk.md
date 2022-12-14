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

GC((Commit)) --> Push

subgraph B[Base]
    A((start)) --> CI[npm ci]
    CI --> E[ESLint]
    A --> J[jsonnetfmt]
    A --> Madge
    CI --> Prettier
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
    C1((start)) --> C2[Docker buildx]
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

# Sample Pipelines: CDelivery

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

# Sample Pipelines: CDeployment

```mermaid
flowchart LR

T((Tag)) --> CD
R[(Container<br/>Registry)]

subgraph CD[Deployment]
    C1 --> launch[Launch Services ...]
    C1 --> C1b[Wait for services...]
    C1((start)) --> C2[Docker buildx]
    C2 -->|push|R
    R -->|pull| C2a
    C2 --> C2a["Launch Container"]
    C1b --> C2a
    C2 --> C3['Wait for container']
    C1b --> C3
    C3 --> C4[Tests]
    C4 --> C5[Post-Deploy Tests]

    subgraph P[Prod]
        PD[Deploy on Servers]
    end

    C4 -->|ssh| P
    R -->|pull|P
    P -.-> C5
    C5 --> C6{Alert?}
end

```

---

# Single Point of Failures

- Drone (CI / CD / CD / Secrets)
- Artifacts Storage:
    - Verdaccio
    - Artifactory
    - Docker Registry

--

## SPoFs: Artifact Storage

- Restart Services
- Disk Backup
- Rebuild Package Index from Tags

--

## SPoFs: Drone (CI / CDel / CDep / Secrets)

- CI: Pipelines as Code
- Secrets: Secrets Manager
- CD:
    - Artifact Storages
    - Missing Secrets
    - Firewalls
- Backups
- Tâches reproductibles

---

# Et si on doit faire un rollback?

---



---

# Morales

- fstab clair
- Secrets Clairs
- Stop the world upgrades?
