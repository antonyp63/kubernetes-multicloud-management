# kubernetes-multicloud-management
## Usage
0. Clone the repo ```git clone ....``` or ```l```
1. Package Guestbook 'app' into a charts with ```helm package gbapp```
2. log in MCM Hub Cluster ```cloudctl login -a <https:x.x.x.x:8443>```
1. Load application charts into ICP with ```cloudctl catalog load-chart --archive gbapp-0.1.x.tgz```
At this point you have upload the Guestbook Application helm to the Hub Cluster.

You will need to repair you Hub cluster to ensure you have a successful deployment.


2. Install application chart with GUI ```Catalog (top right), search for gbapp   ```
3. or CLI ```helm install gbapp -n <release-name> --tls ```
3. Update placement related values to redeploy application
4. Delete helm release to deregister application ```helm delete <release-name> --purge --tls```
5. Example of the Guestbook Application: