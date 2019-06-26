# MCM Image Policy



```
apiVersion: securityenforcement.admission.cloud.ibm.com/v1beta1
kind: ClusterImagePolicy
metadata:
  name: mcm-image-policy
  spec:
    repositories:
      name: gcr.io/kubernetes-e2e-test-images/*
      policy:
        va:
          enabled: false
events:
```
```yaml
apiVersion: compliance.mcm.ibm.com/v1alpha1
kind: Compliance
metadata:
  name: image-compliance-rules
  namespace: kube-system
  description: "apply image compliance"
spec:
  runtime-rules:
  - apiVersion: policy.mcm.ibm.com/v1alpha1
    kind: Policy
    metadata:
      name: policy-image
    spec:
      remediationAction: inform # inform or enforce
      complianceType: musthave
    object-templates:
    - complianceType: musthave
      objectDefinition:
        apiVersion: securityenforcement.admission.cloud.ibm.com/v1beta1
        kind: ClusterImagePolicy
        namespace: {}
        metadata:
          name: mcm-image-policy
          spec:
          repositories:
            name: gcr.io/kubernetes-e2e-test-images/*
            policy:
              va:
                enabled: false


```





```
cat <<EOF | kubectl apply -f -
apiVersion: mcm.ibm.com/v1alpha1
kind: PlacementPolicy
metadata:
  name: image-placement-policy # name of placement policy
spec:
  clusterLabels:
    matchLabels:
      environment: Prod # condition for my placements
---
apiVersion: mcm.ibm.com/v1alpha1
kind: PlacementBinding
metadata:
  name: image-binding
placementRef:
  name: image-placement-policy # refer to the placement policy name
  apiGroup: mcm.ibm.com
  kind: PlacementPolicy
subjects:
- name: image-compliance-rules # refer to the compliance rules
  apiGroup: mcm.ibm.com
  kind: Compliance
EOF
```



