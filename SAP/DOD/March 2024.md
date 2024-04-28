
### Issue : 1

### 1. [Canary]-[Azure]-[Worker stuck as machine can't be deleted]-[Provider]
- Cluster: shoot--hc-dev--xf-az-c2f-haas

- Issue Description: Virtual Machine deletion operation on Azure stuck. MCM was not able to complete the machine deletion operation and run therefore constantly into timeouts. The corresponding `Machine` resource remains in state `Terminating`. MCM reported:

`VM deletion failed due to - machine codes error: code = [Internal] message = [Future#WaitForCompletion: context has been cancelled:`

- Issue Occurrence: once
- Actions Taken: Tried manually deleting the vm using azure cli.
- Resolved or not: Yes
- **Link to journal entry:** NA
- How was it resolved: `az vm delete --name shoot--hc-dev--xf-az-c2f-haas-hana-vm-g-z2-69b86-8r5ts --resource-group shoot--hc-dev--xf-az-c2f-haas`

### Issue : 2

### 2. [Live] - [GCP] - [Create Error in Shoot -(https://dashboard.garden.live.k8s.ondemand.com/namespace/garden-fglpreprod/shoots/fgppgcsa001)]

- Cluster - [shoot](https://dashboard.garden.live.k8s.ondemand.com/namespace/garden-fglpreprod/shoots/fgppgcsa001)

- Issue Description : 
```
* Error creating Subnetwork: googleapi: Error 409: The resource 'projects/sap-fgl-pp-sa/regions/me-central2/subnetworks/shoot--fglpreprod--fgppgcsa001-nodes' already exists, alreadyExists
  with google_compute_subnetwork.subnetwork-nodes,
  on main.tf line 19, in resource "google_compute_subnetwork" "subnetwork-nodes":
  19: resource "google_compute_subnetwork" "subnetwork-nodes" {] Operation will be retried.
```

- Resolved or Not : Yes (Shoot was deleted)
- **Link to journal entry:** [Issue](https://github.tools.sap/kubernetes-live/issues-live/issues/4568)
- How was it resolved: Delete the already existing subnet from provider side
`gcloud compute networks subnets delete shoot--fglpreprod--fgppgcsa001-nodes`
- Similar issue : https://github.tools.sap/kubernetes/ops-guide/blob/85a52c0a2d394f280c14d3f69b8cedd00271d97a/02_Developer_On_Duty/03_DoD_Log/2022/2022-05-02-APJ2.md#L7
- Error : 
```
The subnetwork resource 'projects/sap-fgl-pp-sa/regions/me-central2/subnetworks/shoot--fglpreprod--fgppgcsa001-nodes' is already being used by 'projects/sap-fgl-pp-sa/regions/me-central2/nats/sap-fgl-preprod-sa-cr-me-central2-000-Nat'
```
However the nat does not exist,


### Issue : 3

### 3. [Live] - [GCP] - [# [abap/abapcp-prod-us30] # Machine is not able to join the cluster in 20 mins]

- Issue Description : 
```
Worker extension (shoot--abap--abapcp-prod-us30/abapcp-prod-us30) reports failing health check: machine deployment "shoot--abap--abapcp-prod-us30-logmon-xt-z1" in namespace "shoot--abap--abapcp-prod-us30" is unhealthy: Deployment does not have minimum availability (MinimumReplicasUnavailable).
```

Machine failed to join the cluster in 20m0s minutes.

- ResolvedOrNot : No
- Link to Journal Entry: https://github.tools.sap/kubernetes-live/issues-live/issues/4575
- How was it resolved


### Issue : 4

### 4. [Canary] - [GCP] - [# [garden/gcp-eu2] Prometheus "garden/aggregate" is unhealthy]

### 5. [Canary] - [GCP] - [Prometheus "garden/cache" is unhealthy]

- How was it resolved - Scheduling the pod on other node along with deleting the old volume attachment.

### 6. https://github.tools.sap/kubernetes-live/issues-live/issues/4587

- there are expired orphan leases present in `kube-node-lease` ns
```
k get leases -n kube-node-lease ip-10-250-27-229.eu-central-1.compute.internal -o
yaml
apiVersion: coordination.k8s.io/v1
kind: Lease
metadata:
  creationTimestamp: "2024-03-20T06:08:00Z"
  name: ip-10-250-27-229.eu-central-1.compute.internal
  namespace: kube-node-lease
  ownerReferences:
  - apiVersion: v1
    kind: Node
    name: ip-10-250-27-229.eu-central-1.compute.internal
    uid: 7d93522d-8431-4209-aba1-3b9e452d065c
  resourceVersion: "263088273"
  uid: 72484f38-7fd4-4823-944b-166cd908e2d1
spec:
  holderIdentity: ip-10-250-27-229.eu-central-1.compute.internal
  leaseDurationSeconds: 40
  renewTime: "2024-03-20T11:33:38.525465Z"
  
```

- No nodes are present in the cluster.
- So DWD scaled down the `mcm` and `kcm`.
- Even after removing the orphaned lease, DWD will not perform scaleup because there are no other leases present.
	[code](https://github.com/gardener/dependency-watchdog/blob/55cc3697b3cb3d6547dadbedb8a48ae51283ee56/internal/prober/prober.go#L120)
- For mitigation, I have removed the orphan lease and manually scalled up mcm and kcm.