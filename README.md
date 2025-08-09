# Port IO Assessment
---
## Exercise #1
### 1. JQ patterns that will extract the following information:

#### a. The current replica count:
> Pattern:
>```bash
>.status.replicas
>```

While we could also get the same information from the `.spec.replicas` field, the `.status.replicas` field represents the actual number of replicas that currently exist in the cluster, regardless of their readiness state. This is the most accurate representation of the "current replica count" as requested in your question.

#### b. The deployment strategy:
>Pattern:
>```bash
>.spec.strategy.type
>```

The command above returns the deployment strategy type, which is `RollingUpdate` for the input JSON object.
To get the full objecy of the deployment, we would use the pattern: 
> `.spec.strategy`

#### c. The “service” and “environment” label of the deployment concatenated with a hyphen (-) in the middle:
>Pattern:
>```bash
>.metadata.labels.service + "-" + .metadata.labels.environment
>```

### 2. Extract all issue IDs for all subtasks, into an array:
>Pattern:
>```bash
>[.fields.subtasks[].key]
>```
This pattern navigates to the `subtasks` array within `fields`, uses `[]` to iterate through each subtask object, and extracts the `key` field containing the issue ID. Adding the `[...]` brackets collects all results into a single array.

---

## Exercise #2
