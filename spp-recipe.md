# Deploy IBM Sterling Secure Proxy (SSP)

This recipe is for deploying the IBM Sterling Connect Direct (SC:D) in the `ssp` namespace. This recipe also assumes you've already deployed the [Sterling File Gateway recipe](sfg-recipe.md) - either `b2bi-nonprod` and `b2bi-prod`, or both. 

In particular, these infra resources are assumed to have already been deployed (aside from the B2Bi specific resources):

```yaml
- argocd/namespace-sealed-secrets.yaml
- argocd/daemonset-sync-global-pullsecret.yaml
```

### Infrastructure - kustomization.yaml (in **multi-tenancy-gitops** repository)
1. Edit the Infrastructure layer `${GITOPS_PROFILE}/1-infra/kustomization.yaml`, un-comment the following lines, commit and push the changes and synchronize the `infra` Application in the ArgoCD console.

    ```bash        
    cd multi-tenancy-gitops/0-bootstrap/single-cluster/1-infra
    ```

    In, `kustomization.yaml`:

    ```yaml
    - argocd/namespace-ssp.yaml
    ```

    >  💡 **NOTE**  
    > Commit and Push the changes for `multi-tenancy-gitops` 

### Services - instances folder (in **multi-tenancy-gitops-services** repository)
**NOTE:** This recipe can be implemented using a combination of storage classes. Not all combination will work, but the following table lists the storage classes that have been tested successfully:

    | Component | Access Mode | IBM Cloud | OCS/ODF |
    | --- | --- | --- | --- |
    | PVC | RWO | ibmc-file-gold-gid | ocs-storagecluster-cephfs |

1. Clone the services repo for GitOps: open a terminal window and clone the `multi-tenancy-gitops-services` repository under your Git Organization.
        
    ```bash
    git clone git@github.com:${GIT_ORG}/multi-tenancy-gitops-services.git
    ```
### Services - kustomization.yaml (in **multi-tenancy-gitops** repository)
1. Deploy all pre-requisite resources for SSP in the main `multi-tenancy-gitops` repository
    1. First you will have to run the script from `multi-tenancy-gitops-services/instances/sterling-secure-proxy-setup` 
    ```bash
    CM_SYS_PASS=password \
    CM_ADMIN_PASSWORD=password \
    CM_CERTSTORE_PASSWORD=password \
    CM_CERTENCRYPT_PASSWORD=password \
    CM_CUSTOMCERT_PASSWORD=password \
    ENGINE_SYS_PASS=Passw0rd@ \
    ENGINE_CERTSTORE_PASSWORD=password \
    ENGINE_CERTENCRYPT_PASSWORD=password \
    ENGINE_CUSTOMCERT_PASSWORD=password \
    RWX_STORAGECLASS=ibmc-file-gold-gid \
    ./run-setup.sh
    ```
    ```bash
    cd .. \
    oc apply -f sterling-secure-proxy-setup -n ssp
    ```
    1. Deploy SSP Configuration Manager and Engine instances
        ```yaml
        - argocd/instances/sterling-secure-proxy-cm-instance.yaml
        - argocd/instances/sterling-secure-proxy-engine-instance.yaml
        ```
    >  💡 **NOTE**  
    > Commit and Push the changes for `multi-tenancy-gitops` and
    > **Refresh** the ArgoCD application `services`.
### Possible erros:
1. `Imagebackoff` make sure that you have your `ibm-entitlement-key` on `ssp` namespace also visit all the `ServiceAccounts` assosiated with SSP and make sure that at the bottom you can see your `ibm-entitlement-key` included i.e:
```yaml
imagePullSecrets:
  - name: xxxxxxxxxxxxxxxxxxx
  - name: ibm-entitlement-key
```
---

### Validation

1.  Retrieve the Sterling File Gateway console URL.

    ```bash
    oc get route ibm-ssp-config-manager-ibm-ssp-cm -o jsonpath='https://{ .spec.host }'
    ```

2. Log in with the default credentials:  username:`admin` password: `Password@123` 