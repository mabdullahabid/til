# Rancher Namespace Project Binding

For preview deployments in Rancher, use the "move" action for namespace project assignment rather than direct annotation editing. The Rancher API has specific endpoints for this that handle project bindings correctly.

## Correct Approach

```python
# Use the Rancher API's move action
POST /v3/cluster/{clusterId}/namespaces/{namespaceId}?action=move
{
    "projectId": "c-xxxxx:p-yyyyy"
}
```

Directly editing namespace annotations can leave Rancher's internal state inconsistent.

## Role Binding Update

For Rancher v2.13.3+ with the v13.x provider, use `user_id` instead of `user_principal_id` for role bindings:

```hcl
# Rancher v2.13.3+
resource "rancher2_project_role_template_binding" "user_binding" {
  name           = "user-binding"
  project_id     = rancher2_project.preview.id
  role_template_id = "project-member"
  user_id        = data.rancher2_user.existing.id  # Not user_principal_id
}
```

## The Lesson

Rancher maintains its own state beyond Kubernetes resources. Always use the official API actions for project/namespace management rather than trying to manipulate resources directly.
