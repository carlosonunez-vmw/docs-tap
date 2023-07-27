# Debug workloads

## <a id="check-build-logs"></a> Verify build logs

After the workload is created, you can tail the workload to view the build and runtime logs. For more
information about the `tail` command, see [workload tail](command-reference/tanzu-apps-workload-tail.md)
in the command reference section.

- Verify logs by running:

    ```console
    tanzu apps workload tail pet-clinic --since 10m --timestamp
    ```

    Where:

  - `pet-clinic` is the name you gave the workload.
  - `--since` (optional) the amount of time to go back to begin streaming logs. The default is 1 second.
  - `--timestamp` (optional) prints the timestamp with each log entry.

## <a id="workload-status"></a> Get the workload status and details

After the workload build process is complete, create a Knative service to run the workload. You can
view workload details whenever in the process. Some details, such as the workload URL, are only
available after the workload is running.

1. To verify the workload details, run:

    ```console
    tanzu apps workload get pet-clinic
    ```

    Where:

    - `pet-clinic` is the name of the workload you want details about.

2. You can now see the running workload. When the workload is created, `tanzu apps workload get`
   includes the **URL** for the running workload. Some terminals allow you to `ctrl`+click the
   **URL** to view it. You can also copy and paste the URL into your web browser to see the
   workload.

## <a id="common-workload-errors"></a> Common workload errors

A workload can either be ready, on error or with an unknown status.

There are known errors that makes the workload enter an error or unknown status. The most common are:

- *Local Path Development Error Cases*
  - *Message*: Writing `registry/project/repo/workload:latest`: Writing image: Unexpected status
    code *401 Unauthorized* (HEAD responses have no body, use GET for details)
    - *Cause*: Apps plug-in cannot talk to the registry because the registry credentials are missing
      or invalid.
    - *Resolution*:
      - Run the `docker logout registry` and `docker login registry` commands and specify the valid
        credentials for the registry.
  - *Message*: Writing `registry/project/workload:latest`: Writing image: HEAD Unexpected status
    code *400 Bad Request* (HEAD responses have no body, use GET for details)
    - *Cause*: Certain registries like Harbor or GCR have a concept of `Project`. 400 Bad request is
      sent when either the project does not exists, the user does not have access to it, or the path
      in the `—source-image` flag is missing either project or repository.
    - *Resolution*:
      - Fix the path in the `—source-image` flag value to point to a valid repository path.

- *WorkloadLabelsMissing* / *SupplyChainNotFound*
  - *Message*: No supply chain found where full selector is satisfied by labels:
    map[app.kubernetes.io/part-of:spring-petclinic]
    - *Cause*: The labels and attributes in the workload object did not fully satisfy any installed
      supply chain on the cluster.
    - *Resolution*: Use the `tanzu apps csc list` and `tanzu apps csc get <supply-chain>` commands
      to see the drop-down menu criterias for the supply chains. You can apply the missing labels to
      a workload by using `tanzu apps workload apply`
    - For example,`tanzu apps workload apply workload-name —-type web`
    - For example,`tanzu apps workload apply workload-name --label apps.tanzu.vmware.com/workload-type=web`

- *MissingValueAtPath*
  - *Message*: Waiting to read value [.status.artifact.url] from resource
    gitrepository.source.toolkit.fluxcd.io  in namespace [ns]
  - *Possible Causes*:
    - The Git `url/tag/branch/commit` parameters passed in the workload are not valid.
      - *Resolution*: Fix the invalid Git parameter by using *tanzu apps workload apply*
    - The Git repository is not accessible from the cluster
      - *Resolution*: Configure your cluster networking or your Git repository networking so that
        they can communicate with each other.
    - The namespace is missing the Git secret for communicating with the private repository
      - *Resolution*: Checkout this topics on how to set up Git Authentication for Private
        repositories [Link to [Git
        authentication](https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/1.1/tap/GUID-scc-git-auth.html)]

- *TemplateRejectedByAPIServer*
  - *Message*: Unable to apply object [ns/workload-name] for resource [source-provider] in supply
    chain [source-to-url]: failed to get unstructured [ns/workload-name] from api server:
    imagerepositories.source.apps.tanzu.vmware.com "workload-name" is forbidden: User
    "system:serviceaccount:ns:default" cannot get resource "imagerepositories" in API group
    "source.apps.tanzu.vmware.com" in the namespace "ns"
  - *Cause*: This error happens when the service account in the workload object does not have
    permissions to create objects that are stamped out by the supply chain.
    - *Resolution*: This is fixed by setting up the [Set up developer namespaces to use your installed
      packages](../../install-online/set-up-namespaces.hbs.md)
      with the required service account and permissions.
