#Story #Experiment
### Observations 

- The tests just calls creation of machine object, it's the mc controller responsibility to actually create the machine on the provider.
- There is a `mr` and a `secret` which deploys some scripts on the node and helps it join the cluster, so have to encorporate it.

### ToDo

- [o] try `local_integration_test script` with `provider-local`.
- [o] get `mcc` from gardener-local-setup.
- [ ] See how to apply the things the `mr` is applying.

### Issues
- The `mc binary` is crashing on `UpdateNodetoMachine` method.

### Conclusion
- It's not feasible to use mcm-local-provider for running IT.
- This is because the node agent working on gardener side is responsible for a node joining a cluster.
- So one way would be to run local gardener setup on local in the pipeline.