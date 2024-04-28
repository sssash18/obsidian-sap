#Resources 

- [Operation Guide](https://pages.github.tools.sap/kubernetes/onboarding-website/learn/operationsguide/developer-on-duty/07_onboarding/)
- [Duties](https://pages.github.tools.sap/kubernetes/onboarding-website/learn/operationsguide/developer-on-duty/02_tasks/)
- [Splunk On Call](https://portal.victorops.com/ui/sap-ti-ce/incidents) (Code sap-ti-ce)
- [DOD Schedule](https://sap.sharepoint.com/teams/gardener-blr/_layouts/15/Doc.aspx?sourcedoc={a0af4660-8236-42f2-bd83-6306dbf61e12}&action=edit&wd=target%28DOD.one%7Cbc6a8e17-c942-4f11-9848-254819cb28ef%2FSchedule%202024%7C761fec08-7443-2b4c-99cd-24d0ba10da3f%2F%29&wdorigin=703&wdpreservelink=1)
- Repositories:
	- [Live](https://github.tools.sap/kubernetes-live)
	- [Staging](https://github.tools.sap/kubernetes-staging)
	- [Canary](https://github.tools.sap/kubernetes-canary)
	- [Dev](https://github.tools.sap/kubernetes-dev)
- Dashboards
	- [Live](https://dashboard.garden.live.k8s.ondemand.com/)
	- [Staging](https://dashboard.garden.staging.k8s.ondemand.com/)
	- [Canary](https://dashboard.garden.canary.k8s.ondemand.com/)
	- [Dev](https://dashboard.garden.dev.k8s.ondemand.com/)
### Points to Note

1. Proiority for landscaped -> live, staging, canary, dev.
2. Never change the `shoot yaml` or things affected from the values in it.
3. Never delete any thing without consultaion.
4. Remember to update `oidc-kubeconfig` before the shift once.

### Roadmap to solve a issue

1. Try to search the issue by log in `ops guide` and resolve it.
2. Try to find specific details that can point to a `specific component` or a `misconfiguration from the owner's side`. You can get logs from `Prometheus` or `Vali`.
3. Try to solve the issue by doing some quick hotfixes.
4. If above does not work , create a `ticket` from the cluster dashboard in the relevant repository, and tag the party whose action is required i.e. /ping component or /owner action.