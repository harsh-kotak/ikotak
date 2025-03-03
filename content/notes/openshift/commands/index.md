---
title: OpenShift Reference
weight: 40
---

<!-- Variable -->
{{< note title="Operator Install Plan" >}}

```bash
##find which one need approval
oc get ip -n my-project -o=jsonpath='{.items[?(@.spec.approved==false)].metadata.name}'

#here how to approve all (if batch approval is desired
oc patch installplan $(oc get ip -n my-project-o=jsonpath='{.items[?(@.spec.approved==false)].metadata.name}') -n my-project --type merge --patch '{"spec":{"approved":true}}'  
```

{{< /note >}}
