---
theme: "black"
---

# CIngle Points of Failure

## Dark side of automation awesomeness

---

# $ whoami

![duck tape](images/mythbusters.jpeg)

(c) Discovery Channel

--

# <3 CI/CD/CD

Malgré toutes les erreurs d'implémentation présentées CI/CD/CD reste _best practice_

--

### JWST

![JWST](images/jwst.png)

---

# Timeline

- 5 juillet: OpenSSL Annonce 5 CVEs sous embargo
- 6 juillet: CI fourni pas
- 7 juillet: Date prévue du release de Node.JS patché
- 8 juillet 16h: Release des containers Node.JS

---

# CI fournit pas

![drone](images/drone.png)

--

## Upscaler Workers

- Azure F16s_v4 -> F32s_v4
- Ubuntu 18.04 LTS -> 20.04 LTS

--

## Qu'est-ce qui pourrait mal aller?

- Repartir les Services
- Disk Backup
- Reconstituer les artéfacts à partir des tags

--

```
$ sudo docker ps -a
CONTAINER ID   IMAGE   COMMAND   CREATED   STATUS   PORTS   NAMES
```

--

# Perdu

- Docker Services

--

# Perdu

- ~~Docker Services~~
- Docker Registry

---

## Re-remplir le registre

```bash
sudo docker image ls \
    --format '{{.Repository}}:{{.Tag}}' | \
    grep docker.fgf.cloud | \
    xargs -n1 sudo docker push
```

---

## Pipelines

```mermaid
flowchart LR

T((Tag)) --> Assemble --> V[Validate] --> R[(Deliver<br/>Artifact)] --> D((Deploy))

V --> D
```

--

## Pipelines

```mermaid
flowchart LR

T((Tag)) --> Assemble --> Validate --> D[(Deliver<br/>Artifact)]

PR([Promote<br/>Tag]) --> R[(Pull<br/>Artifact)] --> D2((Deploy))
```

--

# Perdu

- ~~Docker Services~~
- ~~Docker Registry~~
- Build Database / History
- Secrets

--

### Build History

- Git contient encore les tags
- Drone crée un pipeline de tag à la **création** d'un tag

--

### Secrets

- AUTH0_CLIENT_ID
- AUTH0_MANAGEMENT_CLIENT_ID
- API_BACKEND_AUTH0_CLIENT_ID

--

### Secrets

- API_AUTH0_CLIENT_ID
- IDP_AUTH0_CLIENT_ID
- WEB_AUTH0_CLIENT_ID

---

# Perdu

- ~~Docker Services~~
- ~~Docker Registry~~
- ~~Build Database / History~~
- ~~Secrets~~
- Confiance des Devs
- Temps

---

# Azure Linux VM Disk Layout

- /: 30 Go SSD
- /mnt: 100+ Go SSD **éphémère**
- /mnt/\<path\>: On Demand SSD

---

# Questions?

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
    C2 --> C3['Wait for Container']
    C1b --> C3
    C3 --> C4[Tests]
    C4 --> C5{Email?}
end

Push --> B
Push --> C

B & C --> SR{Semantic-<br/>Release}

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

## Sample Pipelines: CDeployment

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
