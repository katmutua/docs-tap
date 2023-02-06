# Create an application accelerator

This is the first in a series of Getting started how-to guides for operators. It walks you through creating an application accelerator by using Tanzu Application Platform GUI and CLI.

For background information about application accelerators and for more advanced features like composition using fragments, see [Application Accelerator](about-application-accelerator.md).

## <a id="you-will"></a>What you will do

- Select a Git repository to create your accelerator and clone the repository.
- Create an `accelerator.yaml` file that defines your accelerator and commit it to your Git repository.
- Publish your application accelerator and view it in Tanzu Application Platform GUI.
- Begin to work with your accelerator.

## <a id="create-an-app-acc"></a>Create an application accelerator


For this example, use a publicly accessible Git repository and include a README file in the repository. You can configure these options when you create a repository on GitHub. You need the repository URL to create an accelerator.

To create a new application accelerator by using your Git repository, follow these steps:

1. Clone your Git repository:

    ```sh
    git clone https://github.com/path/to/repo
    ```

2. Create an empty file named `accelerator.yaml` in the root directory of this Git repository.

3. Add the following content to the `accelerator.yaml` file:

    ```yaml
    accelerator:
      displayName: Simple Accelerator
      description: Contains just a README
      iconUrl: https://images.freecreatives.com/wp-content/uploads/2015/05/smiley-559124_640.jpg
      tags:
      - simple
      - getting-started
    ```

    You can use any icon that has a reachable URL.

4. Add the new `accelerator.yaml` file, commit this change, and push it to your Git repository.

    ```sh
    git add accelerator.yaml
    git commit -m "Creating accelerator.yaml"
    git push
    ```

## <a id="publish-accelerator"></a>Publish the new accelerator

To publish the new application accelerator that is created in your Git repository, follow these steps:

1. Set up environment variables by running:

    ```console
    export GIT_REPOSITORY_URL=YOUR-GIT-REPOSITORY-URL
    export GIT_BRANCH=YOUR-GIT-BRANCH
    ```

    Where:

    - `YOUR-GIT-REPOSITORY-URL` is the URL of your Git repository.
    - `YOUR-GIT-BRANCH` is the name of the branch where you pushed the new `accelerator.yaml` file.

2. To publish the new application accelerator and name it `simple`, run:

    ```console
    tanzu accelerator create simple --git-repository ${GIT_REPOSITORY_URL} --git-branch ${GIT_BRANCH}
    ```

    The accelerator name, `simple`, is used when updating accelerators as well, _not_ the `displayName` parameter in the `accelerator.yaml`

3. Refresh Tanzu Application Platform GUI to reveal the newly published accelerator.

    ![Another accelerator appears in Tanzu Application Platform GUI](../images/new-accelerator-deployed-v1-1.png)

    It might take a few seconds for Tanzu Application Platform GUI to refresh the catalog and add an entry for your new accelerator.

## <a id="work-with-accelerators"></a>Working with accelerators

### <a id="accelerator-updates"></a>Updating an accelerator

After you push any changes to your Git repository, the accelerator is refreshed based on the `git.interval` setting for the Accelerator resource. The default value is 10 minutes. To force an immediate reconciliation, run:

```console
tanzu accelerator update ACCELERATOR-NAME --reconcile
```

Where `ACCELERATOR-NAME` is the name of your accelerator.

### <a id="accelerator-deletes"></a>Deleting an accelerator

When you no longer need your accelerator, you can delete it by using the Tanzu CLI:

```console
tanzu accelerator delete ACCELERATOR-NAME
```

### <a id="accelerator-manifest"></a>Using an accelerator manifest

You can also create a separate manifest file and apply it to the cluster by using the Tanzu CLI:

1. Create a `simple-manifest.yaml` file and add the following content:

    ```yaml
    apiVersion: accelerator.apps.tanzu.vmware.com/v1alpha1
    kind: Accelerator
    metadata:
      name: simple
      namespace: accelerator-system
    spec:
      git:
        url: YOUR-GIT-REPOSITORY-URL
        ref:
          branch: YOUR-GIT-BRANCH
    ```

    Where:

    - `YOUR-GIT-REPOSITORY-URL` is the URL of your Git repository.
    - `YOUR-GIT-BRANCH` is the name of the branch.

2. Apply the `simple-manifest.yaml` by running the following command in the directory where you created this file:

    ```console
    kubectl apply -f simple-manifest.yaml
    ```

Application Accelerator supports handling and management of accelerator manifests by using standard GitOps practices. For more information, see [Configure Application Accelerator](../application-accelerator/configuration.hbs.md#using-git-ops).

## Next steps

- [Add testing and scanning to your application](add-test-and-security.md)
