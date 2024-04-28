# C -> One by One scaleups of each WP

### Simulation Result

> wp1 -> m5.large; wp2 -> m5.2xlarge

```
NAME                         READY   STATUS    RESTARTS   AGE     IP       NODE                                          NOMINATED NODE   READINESS GATES
pod/scenario-c-small-422n7   0/1     Pending   0          6m49s   <none>   ip-10-180-14-180.eu-west-1.compute.internal   <none>           <none>
pod/scenario-c-small-5kv6b   0/1     Pending   0          6m49s   <none>   ip-10-180-4-61.eu-west-1.compute.internal     <none>           <none>
pod/scenario-c-small-6qc9z   0/1     Pending   0          6m49s   <none>   ip-10-180-14-180.eu-west-1.compute.internal   <none>           <none>
pod/scenario-c-small-758d6   0/1     Pending   0          6m49s   <none>   ip-10-180-14-180.eu-west-1.compute.internal   <none>           <none>
pod/scenario-c-small-8zb8g   0/1     Pending   0          6m49s   <none>   ip-10-180-14-180.eu-west-1.compute.internal   <none>           <none>
pod/scenario-c-small-csfq2   0/1     Pending   0          6m49s   <none>   wg-wp-2-rqnsq                                 <none>           <none>
pod/scenario-c-small-j57p8   0/1     Pending   0          6m49s   <none>   wg-wp-2-rqnsq                                 <none>           <none>
pod/scenario-c-small-jggjr   0/1     Pending   0          6m49s   <none>   ip-10-180-14-180.eu-west-1.compute.internal   <none>           <none>
pod/scenario-c-small-k74bn   0/1     Pending   0          6m49s   <none>   wg-wp-2-rqnsq                                 <none>           <none>
pod/scenario-c-small-qj8ss   0/1     Pending   0          6m49s   <none>   ip-10-180-14-180.eu-west-1.compute.internal   <none>           <none>
pod/scenario-c-small-scnzf   0/1     Pending   0          6m49s   <none>   wg-wp-1-xdk5g                                 <none>           <none>
pod/scenario-c-small-wbxvv   0/1     Pending   0          6m49s   <none>   wg-wp-2-rqnsq                                 <none>           <none>
pod/scenario-c-small-zjf5d   0/1     Pending   0          6m49s   <none>   ip-10-180-14-180.eu-west-1.compute.internal   <none>           <none>

NAME                                               STATUS    ROLES    AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE              KERNEL-VERSION                   CONTAINER-RUNTIME
node/ip-10-180-14-180.eu-west-1.compute.internal   Ready     <none>   6m49s   v1.28.4   10.180.14.180   <none>        Garden Linux 1312.1   6.1.63-gardenlinux-cloud-amd64   containerd://1.6.24
node/ip-10-180-4-61.eu-west-1.compute.internal     Ready     <none>   6m49s   v1.28.4   10.180.4.61     <none>        Garden Linux 1312.1   6.1.63-gardenlinux-cloud-amd64   containerd://1.6.24
node/wg-wp-1-xdk5g                                 Unknown   <none>   6m32s             <none>          <none>        <unknown>             <unknown>                        <unknown>
node/wg-wp-2-rqnsq                                 Unknown   <none>   6m17s             <none>          <none>        <unknown>             <unknown>                        <unknown>
```

### Actual CA behaviour

```
I0216 11:30:39.864149       1 waste.go:55] Expanding Node Group shoot--i544000--scenario-c-wp-1-z1 would waste 95.00% CPU, 47.09% Memory, 71.05% Blended
I0216 11:30:39.864168       1 waste.go:55] Expanding Node Group shoot--i544000--scenario-c-wp-2-z1 would waste 93.75% CPU, 35.28% Memory, 64.52% Blended
I0216 11:30:39.864178       1 orchestrator.go:185] Best option to resize: shoot--i544000--scenario-c-wp-2-z1
I0216 11:30:39.864185       1 orchestrator.go:189] Estimated 1 nodes needed in shoot--i544000--scenario-c-wp-2-z1
I0216 11:30:39.864208       1 orchestrator.go:253] No similar node groups found
I0216 11:30:39.864228       1 orchestrator.go:295] Final scale-up plan: [{shoot--i544000--scenario-c-wp-2-z1 1->2 (max: 5)}]
I0216 11:30:39.864246       1 executor.go:147] Scale-up: setting group shoot--i544000--scenario-c-wp-2-z1 size to 2

```

#### Actual pod assignment

```
NAME                     READY   STATUS    RESTARTS   AGE     IP            NODE                                          NOMINATED NODE   READINESS GATES
scenario-c-small-6znld   1/1     Running   0          5m32s   100.64.3.5    ip-10-180-21-222.eu-west-1.compute.internal   <none>           <none>
scenario-c-small-dgkhl   1/1     Running   0          5m37s   100.64.0.27   ip-10-180-14-180.eu-west-1.compute.internal   <none>           <none>
scenario-c-small-flbmz   1/1     Running   0          5m43s   100.64.0.24   ip-10-180-14-180.eu-west-1.compute.internal   <none>           <none>
scenario-c-small-hhgzn   1/1     Running   0          5m33s   100.64.3.2    ip-10-180-21-222.eu-west-1.compute.internal   <none>           <none>
scenario-c-small-jxfcw   1/1     Running   0          5m30s   100.64.3.6    ip-10-180-21-222.eu-west-1.compute.internal   <none>           <none>
scenario-c-small-mltkn   1/1     Running   0          5m47s   100.64.0.22   ip-10-180-14-180.eu-west-1.compute.internal   <none>           <none>
scenario-c-small-ng6t6   1/1     Running   0          5m45s   100.64.0.23   ip-10-180-14-180.eu-west-1.compute.internal   <none>           <none>
scenario-c-small-p6pjs   1/1     Running   0          5m35s   100.64.0.28   ip-10-180-14-180.eu-west-1.compute.internal   <none>           <none>
scenario-c-small-tgxq4   1/1     Running   0          5m27s   100.64.3.4    ip-10-180-21-222.eu-west-1.compute.internal   <none>           <none>
scenario-c-small-wgmcw   1/1     Running   0          5m42s   100.64.0.25   ip-10-180-14-180.eu-west-1.compute.internal   <none>           <none>
scenario-c-small-x9kcx   1/1     Running   0          5m40s   100.64.0.26   ip-10-180-14-180.eu-west-1.compute.internal   <none>           <none>
scenario-c-small-xdn9t   1/1     Running   0          5m29s   100.64.3.3    ip-10-180-21-222.eu-west-1.compute.internal   <none>           <none>
scenario-c-small-z24vk   1/1     Running   0          5m38s   100.64.2.8    ip-10-180-4-61.eu-west-1.compute.internal     <none>           <none>
```

#### Actual nodes

> (14-180,21-222 are m5.2xlarge; 4-61 is m5.large)
```
NAME                                          STATUS   ROLES    AGE     VERSION
ip-10-180-14-180.eu-west-1.compute.internal   Ready    <none>   69m     v1.28.4
ip-10-180-21-222.eu-west-1.compute.internal   Ready    <none>   5m44s   v1.28.4
ip-10-180-4-61.eu-west-1.compute.internal     Ready    <none>   67m     v1.28.4
```