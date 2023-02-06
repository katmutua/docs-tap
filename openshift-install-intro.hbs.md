# Install Tanzu Application Platform (OpenShift)

To install Tanzu Application Platform on your OpenShift clusters with internet access:

|Step|Task|Link|
|----|----|----|
|1.| Review the prerequisites to ensure you have met all requirements before installing. |[Prerequisites](prerequisites.hbs.md)|
|2.| Accept Tanzu Application Platform EULAs and install the Tanzu CLI. |[Accept Tanzu Application Platform EULAs and installing the Tanzu CLI](install-tanzu-cli.hbs.md)|
|3.| Install Cluster Essentials for Tanzu. |[Deploy Cluster Essentials](https://docs.vmware.com/en/Cluster-Essentials-for-VMware-Tanzu/{{ vars.url_version }}/cluster-essentials/deploy.html)|
|4.| Add the Tanzu Application Platform package repository, prepare your Tanzu Application Platform profile, and install the profile to the cluster. |[Install the Tanzu Application Platform package and profiles](install-openshift.hbs.md)|
|5.| (Optional) Install any additional packages that were not in the profile. |[Install individual packages](install-components-openshift.hbs.md)|
|6.| Set up developer namespaces to use installed packages. |[Set up developer namespaces to use installed packages](set-up-namespaces-openshift.hbs.md)|
|7.| Install developer tools into your integrated development environment (IDE). |[Install Tanzu Developer Tools for VS Code](vscode-install-openshift.hbs.md)|

After installing Tanzu Application Platform on to your OpenShift clusters, proceed with [Get started with Tanzu Application Platform](getting-started.hbs.md).
