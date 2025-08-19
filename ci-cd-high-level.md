# CI/CD to Kubernetes: The Big Picture

```mermaid
flowchart TD
  A["Developer commits code (GitHub/GitLab)"]

  subgraph CI["Continuous Integration"]
    direction LR

    subgraph JENK["Jenkins"]
      J1["Build & test in pod"]
      J2["Package Docker image"]
    end

    subgraph GHA_GL["GitHub Actions/GitLab CICD"]
      G1["Build & test in runner"]
      G2["Package Docker image"]
    end
  end

  R["Push image to container registry"]

  subgraph KDEP["Deploy to Kubernetes"]
    direction TB
    K7["Cluster pulls image from registry"]
    K8["Deployment creates/updates Pods"]
    K9["ConfigMaps & Secrets inject config"]
    K10["Readiness/Liveness probes enforce health"]
    K11["Jobs/CronJobs run batch & scheduled tasks"]
  end

  %% Flow choices
  A --> B{Select CI engine}
  B -->|Jenkins| J1 -->|fail| A
  J1 -->|pass| J2 --> R
  B -->|Actions/GitLab| G1 -->|fail| A
  G1 -->|pass| G2 --> R

  %% Deploy path
  R --> K7 --> K8 --> K9 --> K10
  K8 --> K11
```

**Q: Are Jenkins and GitHub Actions/GitLab CI/CD mutually exclusive?**

**A:** Not at all! Teams often use them together. GitHub/GitLab CI is great for repo-native builds and quick checks, while Jenkins excels at complex deployments, integrations, and on-prem environments. Many orgs run both in parallel during transitions or for different use cases.
