# MCM Image Policy

cloudctl catalog delete-chart --name gbapp -r local-charts -v 0.1.0


kubectl get clusterimagepolicy
kubectl describe clusterimagepolicy guestbook
kubectl delete clusterimagepolicy mcm-image-policy
kubectl delete clusterimagepolicy guestbook

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: securityenforcement.admission.cloud.ibm.com/v1beta1
kind: ClusterImagePolicy
metadata:
  name: mcm-image-policy
spec:
  repositories:
  - name: 'gcr.io/google-samples/*'
    policy:
      va:
        enabled: false
EOF
```
```
apiVersion: securityenforcement.admission.cloud.ibm.com/v1beta1
kind: ClusterImagePolicy
metadata:
  name: guestbook
spec:
  repositories:
  - name: 'gcr.io/google-samples/*'
    policy:
      va:
        enabled: false

```


```yaml
cat <<EOF | kubectl apply -f -
apiVersion: mcm.ibm.com/v1alpha1
kind: PlacementPolicy
metadata:
  name: mcm-image-placement-policy # name of placement policy
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
  name: mcm-image-placement-policy # refer to the placement policy name
  apiGroup: mcm.ibm.com
  kind: PlacementPolicy
subjects:
- name: mcm-compliance-image-policy # refer to the compliance rules
  apiGroup: mcm.ibm.com
  kind: Compliance
EOF
```

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: compliance.mcm.ibm.com/v1alpha1
kind: Compliance
metadata:
  name: mcm-compliance-image-policy
  namespace: kube-system
  description: "apply image compliance"
spec:
  runtime-rules:
  - apiVersion: policy.mcm.ibm.com/v1alpha1
    kind: Policy
    metadata:
      name: policy-image
    spec:
      remediationAction: enforce # inform or enforce
      namespaces:
        exclude:
          - kube*
        include:
          - '*'
      complianceType: musthave
      object-templates:
        - complianceType: musthave
          objectDefinition:
            apiVersion: securityenforcement.admission.cloud.ibm.com/v1beta1
            kind: ClusterImagePolicy
            metadata:
              name: mcm-image-policy
            spec:
              repositories:
               - name: [gcr.io/kubernetes-e2e-test2-images/*, gcr.io/google-samples/*]
EOF               
```


### Guestbook Complance image policy - Dont work correctly

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: securityenforcement.admission.cloud.ibm.com/v1beta1
kind: ClusterImagePolicy
metadata:
  name: guestbook
spec:
  repositories:
  - name: 'gcr.io/google-samples/gb-redisslave'
    policy:
      va:
        enabled: false
  - name: 'gcr.io/kubernetes-e2e-test-images/redis'
    policy:
      va:
        enabled: false
  - name: 'gcr.io/google-samples/gb-frontend'
    policy:
      va:
        enabled: false
EOF
```

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: mcm.ibm.com/v1alpha1
kind: PlacementPolicy
metadata:
  name: guestbook-image-placement-policy # name of placement policy
spec:
  clusterLabels:
    matchLabels:
      environment: Prod # condition for my placements
---
apiVersion: mcm.ibm.com/v1alpha1
kind: PlacementBinding
metadata:
  name: guestbook-image-binding
placementRef:
  name: guestbook-image-placement-policy # refer to the placement policy name
  apiGroup: mcm.ibm.com
  kind: PlacementPolicy
subjects:
- name: guestbook-compliance-image-policy # refer to the compliance rules
  apiGroup: mcm.ibm.com
  kind: Compliance
EOF

```

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: compliance.mcm.ibm.com/v1alpha1
kind: Compliance
metadata:
  name: guestbook-compliance-image-policy
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
      namespaces:
        exclude:
          - kube*
        include:
          - '*'
      complianceType: musthave
      object-templates:
        - complianceType: musthave
          objectDefinition:
            apiVersion: securityenforcement.admission.cloud.ibm.com/v1beta1
            kind: ClusterImagePolicy
            metadata:
              name: guestbook
            spec:
              repositories:
                [name: gcr.io/google-samples/gb-redisslave, name: gcr.io/kubernetes-e2e-test-images/redis, name: gcr.io/google-samples/gb-frontend] 
EOF
              
```

### use this  version

```
cat <<EOF | kubectl apply -f -
apiVersion: policy.mcm.ibm.com/v1alpha1
kind: Policy
metadata:
  name: guestbook-image-policy
spec:
  remediationAction: inform # inform or enforce
  namespaces:
    exclude:
      - kube*
    include:
      - '*'
  complianceType: musthave
  object-templates:
    - complianceType: musthave
      objectDefinition:
        apiVersion: securityenforcement.admission.cloud.ibm.com/v1beta1
        kind: ClusterImagePolicy
        metadata:
          name: guestbook
        spec:
          repositories:
            [name: gcr.io/google-samples/gb-redisslave, name: gcr.io/kubernetes-e2e-test-images/redis, name: gcr.io/google-samples/gb-frontend]

EOF
```

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: mcm.ibm.com/v1alpha1
kind: PlacementPolicy
metadata:
  name: guestbook-image-placement-policy # name of placement policy
spec:
  clusterLabels:
    matchLabels:
      environment: Prod # condition for my placements
---
apiVersion: mcm.ibm.com/v1alpha1
kind: PlacementBinding
metadata:
  name: guestbook-image-binding
placementRef:
  name: guestbook-image-placement-policy # refer to the placement policy name
  apiGroup: mcm.ibm.com
  kind: PlacementPolicy
subjects:
- name: guestbook-image-policy # refer to the compliance rules
  apiGroup: mcm.ibm.com
  kind: Policy
EOF

```





