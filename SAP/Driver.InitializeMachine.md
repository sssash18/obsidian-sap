#Learn

Currently there are some steps required specific to each provider which needs to be performed after the `machine` creation (ex - src-dest check disabled on NAT EC2 instances). As of now that happens  in the `CreateMachine` function itself. So we segregate that in a seperate function `InitializeMachine`. It should be a one time call post VM creation.

### PRs:
- [MCM](https://github.com/gardener/machine-controller-manager/pull/898/files)
- [MCM-AWS](https://github.com/gardener/machine-controller-manager-provider-aws/pull/157/files)
