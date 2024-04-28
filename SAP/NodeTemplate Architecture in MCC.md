#Issue

#### Observations
- Sometimes the `architecture` field in `machineclass` just flashes and disappears - this indicates may be a webhook is mutating the resource.
---------
##### Experiment 1
###### Conditions to test
- [o] Print the logs in the `RenderEmbeddedFS` function and see the value of `release.Manifest()`
###### **Result**s 
- Printed logs at multiple places , `architecture` appears everywhere as expected . Refer logs in */tmp/gepaws3.log 

>> **VERDICT : NEGATIVE**

-------------
##### Experiment 2  
###### Conditions to test
- [o] `omitempty` removed from `architecture` field in `NodeTemplate` of mcm/v1alpha1 and run `make generate`
- [o] MCM version to be updated in gardener and gep-aws
- [o] Update `crd` in gardener
###### Results
- `architecture` is absent in `NodeTemplate` , though it was present in CRD and CREATION object. Refer */tmp/gepaws4.log*  
- 
>> **VERDICT** : **NEGATIVE**

-----
##### Experiment 3
###### Conditions to test
- [o] make the `architecture` field as required and assign a default value `amd64` in the CRD.
###### Results
 - The deployed mcc has the default value for architecture until second reconciliation.

>> **VERDICT : NEGAGTIVE**

-----
##### Experiment 4

###### Conditions to test
- [ ] Try to print the `kubeconfig` used by `gepaws` to apply the `mcc`.
- [ ] Then use that `kubeconfig` from a mock program and try to apply mcc on that cluster and observe the `architecture` field. 