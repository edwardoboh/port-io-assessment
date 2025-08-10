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
The updated Jira Integration Mapping which used the Component(s) name to map Jira Issue to GitHub Repository is shown below:
```yaml
resources:
#   - kind: user ...
#   - kind: project ...
  - kind: issue
    selector:
      query: 'true'
      jql: (statusCategory != Done) OR (created >= -1w) OR (updated >= -1w)
    port:
      entity:
        mappings:
          identifier: .key
          title: .fields.summary
          blueprint: '"jiraIssue"'
          properties:
            url: (.self | split("/") | .[:3] | join("/")) + "/browse/" + .key
            status: .fields.status.name
            issueType: .fields.issuetype.name
            components: .fields.components
            creator: .fields.creator.emailAddress
            priority: .fields.priority.name
            labels: .fields.labels
            created: .fields.created
            updated: .fields.updated
            resolutionDate: .fields.resolutiondate
          relations:
            project: .fields.project.key
            parentIssue: .fields.parent.key
            subtasks: .fields.subtasks | map(.key)
            jira_user_assignee: .fields.assignee.accountId
            jira_user_reporter: .fields.reporter.accountId
            assignee:
              combinator: '"or"'
              rules:
                - property: '"jira_user_id"'
                  operator: '"="'
                  value: .fields.assignee.accountId // ""
                - property: '"$identifier"'
                  operator: '"="'
                  value: .fields.assignee.email // ""
            reporter:
              combinator: '"or"'
              rules:
                - property: '"jira_user_id"'
                  operator: '"="'
                  value: .fields.reporter.accountId // ""
                - property: '"$identifier"'
                  operator: '"="'
                  value: .fields.reporter.email // ""
            repository: .fields.components | map(.name)
```

In the `issue` resource kind above, i added a `repository` identifier under the `relations` field (i.e `.port.entity.mappings.relations`) and equated it to `.fields.components | map(.name)`.
What this does is to extract the component names from the Jira issue component(s) array and returns them as a simple list. We could have also made use of the patter `.fields.components[].name` as they prodoce the same result.