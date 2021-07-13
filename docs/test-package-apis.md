# Temporary Testing Doc for Packaging APIs

To validate the packaging APIs, TCE requires an alpha build of kapp-controller
[imgpkg bundle
format](https://github.com/vmware-tanzu/carvel-kapp-controller/issues/57). In
order to make this work, you need to **stop** the management cluster from
managing the guest clusters's kapp-controller. This enables you to mutate the
version of kapp-controller on the guest cluster.

## Setting up kapp-controller

1. Set your kube context to the **management cluster**.

    ```sh
    kubectl config use-context ${MGMT_CLUSTER_NAME}-admin@${MGMT_CLUSTER_NAME}
    ```

1. Set the `kapp-controller` App CR to pause reconciliation.

    ```sh
    kubectl patch app/${GUEST_CLUSTER_NAME}-kapp-controller --patch '{"spec":{"paused":true}}' --type=merge
    ```

1. Validate `kapp-controller` is not actively managed.

    ```sh
    $ kubectl get app -A
    NAMESPACE    NAME                        DESCRIPTION           SINCE-DEPLOY   AGE
    default      tce-guest-kapp-controller   Canceled/paused       128m           135m
    tkg-system   antrea                      Reconcile succeeded   2m40s          152m
    tkg-system   metrics-server              Reconcile succeeded   2m49s          149m
    tkg-system   tanzu-addons-manager        Reconcile succeeded   2m53s          153m
    ```

1. Set your kube context to the **workload/guest** cluster.

    ```sh
    kubectl config use-context ${MGMT_CLUSTER_NAME}-admin@${MGMT_CLUSTER_NAME}
    ```

1. Delete the existing `kapp-controller`.

   ```sh
   kubectl delete deploy -n tkg-system kapp-controller
   ```

1. Apply the alpha `kapp-controller` into the cluster.

   ```sh
   kubectl apply -f https://raw.githubusercontent.com/vmware-tanzu/carvel-kapp-controller/dev-packaging/alpha-releases/v0.17.0-alpha.1.yml
   ```

## Deploying the Packages

1. Deploy the PackageRepository

   ```sh
   kubectl apply -f addons/repos/main.yaml
   ```

1. Verify the package are available in the cluster.

   ```sh
   kubectl get packages
   NAME                              PUBLIC-NAME        VERSION          AGE
   contour-operator.1.11.0-vmware0   contour-operator   1.11.0-vmware0   112m
   ```

1. Deploy the service account and cluster rolebinding.

   ```sh
   kubectl apply -f addons/packages/contour-operator/serviceaccount.yaml &&\
     kubectl apply -f addons/packages/contour-operator/clusterrolebinding.yaml
   ```

1. Deploy the installed package.

   ```sh
   kubectl apply -f addons/packages/contour-operator/installedpackage.yaml
   ```

1. Validate all resources are available.

   ```sh
   kubectl get packagerepositories,packages,installedpackages -A
   ```