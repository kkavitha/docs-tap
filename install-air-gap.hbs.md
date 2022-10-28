# Install Tanzu Application Platform in an air-gapped environment

This topic describes how to install Tanzu Application Platform on your Kubernetes cluster and registry that are air-gapped from external traffic.

Before installing the packages, ensure that you have completed the following tasks:

- Review the [Prerequisites](prerequisites.html) to ensure that you have set up everything required before beginning the installation.
- [Accept Tanzu Application Platform EULA and install Tanzu CLI](install-tanzu-cli.html).
- [Deploy Cluster Essentials](https://docs.vmware.com/en/Cluster-Essentials-for-VMware-Tanzu/1.3/cluster-essentials/GUID-deploy.html). This step is optional if you are using VMware Tanzu Kubernetes Grid cluster.

## <a id='add-tap-package-repo'></a> Relocate images to a registry

To relocate images from the VMware Tanzu Network registry to your air-gapped registry:

1. Log in to your image registry by running:

    ```console
    docker login MY-REGISTRY
    ```

    Where `MY-REGISTRY` is your own container registry.

1. Log in to the VMware Tanzu Network registry with your VMware Tanzu Network credentials by running:

    ```console
    docker login registry.tanzu.vmware.com
    ```

1. Set up environment variables for installation use by running:

    ```console
    export IMGPKG_REGISTRY_HOSTNAME=MY-REGISTRY
    export IMGPKG_REGISTRY_USERNAME=MY-REGISTRY-USER
    export IMGPKG_REGISTRY_PASSWORD=MY-REGISTRY-PASSWORD
    export TAP_VERSION=VERSION-NUMBER
    export REGISTRY_CA_PATH=PATH-TO-CA
    ```

    Where:

    - `MY-REGISTRY` is your air-gapped container registry.
    - `MY-REGISTRY-USER` is the user with write access to `MY-REGISTRY`.
    - `MY-REGISTRY-PASSWORD` is the password for `MY-REGISTRY-USER`.
    - `VERSION-NUMBER` is your Tanzu Application Platform version. For example, `{{ vars.tap_version }}`.

1. Copy the images into a `.tar` file from the VMware Tanzu Network onto an external storage device with the Carvel tool imgpkg by running:

    ```console
    imgpkg copy \
      -b registry.tanzu.vmware.com/tanzu-application-platform/tap-packages:$TAP_VERSION \
      --to-tar tap-packages-$TAP_VERSION.tar \
      --include-non-distributable-layers
    ```

    Where:

    - `TANZUNET-REGISTRY-USERNAME` is your username of the VMware Tanzu Network.
    - `TANZUNET-REGISTRY-PASSWORD` is your password of the VMware Tanzu Network.

1. Relocate the images with the Carvel tool imgpkg by running:

    ```console
    imgpkg copy \
      --tar tap-packages-$TAP_VERSION.tar \
      --to-repo $IMGPKG_REGISTRY_HOSTNAME/tap-packages \
      --include-non-distributable-layers
      --registry-ca-cert-path $REGISTRY_CA_PATH
    ```

1. Create a namespace called `tap-install` for deploying any component packages by running:

    ```console
    kubectl create ns tap-install
    ```

    This namespace keeps the objects grouped together logically.

1. Create a registry secret by running:

    ```console
    tanzu secret registry add tap-registry \
        --server   $IMGPKG_REGISTRY_HOSTNAME \
        --username $IMGPKG_REGISTRY_USERNAME \
        --password $IMGPKG_REGISTRY_PASSWORD \
        --namespace tap-install \
        --export-to-all-namespaces \
        --yes
    ```

1. Add the Tanzu Application Platform package repository to the cluster by running:

    ```console
    tanzu package repository add tanzu-tap-repository \
      --url $IMGPKG_REGISTRY_HOSTNAME/tap-packages:$TAP_VERSION \
      --namespace tap-install
    ```

    Where:

    - `$TAP_VERSION` is the Tanzu Application Platform version environment variable you defined earlier.
    - `TARGET-REPOSITORY` is the necessary repository.

1. Get the status of the Tanzu Application Platform package repository, and ensure the status updates to `Reconcile succeeded` by running:

    ```console
    tanzu package repository get tanzu-tap-repository --namespace tap-install
    ```

    > **Note:** The `VERSION` and `TAG` numbers differ from the earlier example if you are on
    > Tanzu Application Platform v1.0.2 or earlier.

1. List the available packages by running:

    ```console
    tanzu package available list --namespace tap-install
    ```

    For example:

    ```console
    $ tanzu package available list --namespace tap-install
    / Retrieving available packages...
      NAME                                                 DISPLAY-NAME                                                              SHORT-DESCRIPTION
      accelerator.apps.tanzu.vmware.com                    Application Accelerator for VMware Tanzu                                  Used to create new projects and configurations.
      apis.apps.tanzu.vmware.com                           API Auto Registration for VMware Tanzu                                    A TAP component to automatically register API exposing workloads as API entities in TAP GUI.
      api-portal.tanzu.vmware.com                          API portal                                                                A unified user interface to enable search, discovery and try-out of API endpoints at ease.
      backend.appliveview.tanzu.vmware.com                 Application Live View for VMware Tanzu                                    App for monitoring and troubleshooting running apps
      connector.appliveview.tanzu.vmware.com               Application Live View Connector for VMware Tanzu                          App for discovering and registering running apps
      conventions.appliveview.tanzu.vmware.com             Application Live View Conventions for VMware Tanzu                        Application Live View convention server
      buildservice.tanzu.vmware.com                        Tanzu Build Service                                                       Tanzu Build Service enables the building and automation of containerized software workflows securely and at scale.
      cartographer.tanzu.vmware.com                        Cartographer                                                              Kubernetes native Supply Chain Choreographer.
      cnrs.tanzu.vmware.com                                Cloud Native Runtimes                                                     Cloud Native Runtimes is a serverless runtime based on Knative
      controller.conventions.apps.tanzu.vmware.com         Convention Service for VMware Tanzu                                       Convention Service enables app operators to consistently apply desired runtime configurations to fleets of workloads.
      controller.source.apps.tanzu.vmware.com              Tanzu Source Controller                                                   Tanzu Source Controller enables workload create/update from source code.
      developer-conventions.tanzu.vmware.com               Tanzu App Platform Developer Conventions                                  Developer Conventions
      grype.scanning.apps.tanzu.vmware.com                 Grype Scanner for Supply Chain Security Tools - Scan                      Default scan templates using Anchore Grype
      image-policy-webhook.signing.apps.tanzu.vmware.com   Image Policy Webhook                                                      The Image Policy Webhook allows platform operators to define a policy that will use cosign to verify signatures of container images
      learningcenter.tanzu.vmware.com                      Learning Center for Tanzu Application Platform                            Guided technical workshops
      ootb-supply-chain-basic.tanzu.vmware.com             Tanzu App Platform Out of The Box Supply Chain Basic                      Out of The Box Supply Chain Basic.
      ootb-supply-chain-testing-scanning.tanzu.vmware.com  Tanzu App Platform Out of The Box Supply Chain with Testing and Scanning  Out of The Box Supply Chain with Testing and Scanning.
      ootb-supply-chain-testing.tanzu.vmware.com           Tanzu App Platform Out of The Box Supply Chain with Testing               Out of The Box Supply Chain with Testing.
      ootb-templates.tanzu.vmware.com                      Tanzu App Platform Out of The Box Templates                               Out of The Box Templates.
      scanning.apps.tanzu.vmware.com                       Supply Chain Security Tools - Scan                                        Scan for vulnerabilities and enforce policies directly within Kubernetes native Supply Chains.
      metadata-store.apps.tanzu.vmware.com                 Tanzu Supply Chain Security Tools - Store                                 The Metadata Store enables saving and querying image, package, and vulnerability data.
      service-bindings.labs.vmware.com                     Service Bindings for Kubernetes                                           Service Bindings for Kubernetes implements the Service Binding Specification.
      services-toolkit.tanzu.vmware.com                    Services Toolkit                                                          The Services Toolkit enables the management, lifecycle, discoverability and connectivity of Service Resources (databases, message queues, DNS records, etc.).
      spring-boot-conventions.tanzu.vmware.com             Tanzu Spring Boot Conventions Server                                      Default Spring Boot convention server.
      sso.apps.tanzu.vmware.com                            AppSSO                                                                    Application Single Sign-On for Tanzu
      tap-gui.tanzu.vmware.com                             Tanzu Application Platform GUI                                            web app graphical user interface for Tanzu Application Platform
      tap.tanzu.vmware.com                                 Tanzu Application Platform                                                Package to install a set of TAP components to get you started based on your use case.
      workshops.learningcenter.tanzu.vmware.com            Workshop Building Tutorial                                                Workshop Building Tutorial
    ```

## <a id='air-gap-policy'></a> Prepare Sigstore TUF Stack for Air-gapped Policy Controller

Supply Chain Security Tools - Policy Controller requires access to a The Update Framework (TUF) server.
In an environment with public Internet access, the public official Sigstore TUF server is used.

The Sigstore Stack consists of:

- [Trillian](https://github.com/google/trillian)
- [Rekor](https://github.com/sigstore/rekor)
- [Fulcio](https://github.com/sigstore/fulcio)
- [Certificate Transparency Log (CTLog)](https://github.com/google/certificate-transparency-go)
- [The Update Framework (TUF)](https://theupdateframework.io/)

For an air-gapped environment, an internally accessible Sigstore stack is required. While installing, Policy Controller fails to reconcile and deploy in most cases. The Policy Controller must be configured with a TUF mirror and TUF root.

For more information about how to set up the Sigstore Stack, see [Install Sigstore Stack](scst-policy/install-sigstore-stack.html).

## <a id='install-profile'></a> Install your Tanzu Application Platform profile

The `tap.tanzu.vmware.com` package installs predefined sets of packages based on your profile settings.
This is done by using the package manager installed by Tanzu Cluster Essentials.

For more information about profiles, see [About Tanzu Application Platform components and profiles](about-package-profiles.md).

To prepare to install a profile:

1. List version information for the package by running:

    ```console
    tanzu package available list tap.tanzu.vmware.com --namespace tap-install
    ```

1. Create a `tap-values.yaml` file by using the
[Full Profile sample](#full-profile) as a guide.
These samples have the minimum configuration required to deploy Tanzu Application Platform.
The sample values file contains the necessary defaults for:

    - The meta-package, or parent Tanzu Application Platform package
    - Subordinate packages, or individual child packages

    >**Important:** Keep the values file for future configuration use.

    >**Important:** The policy controller `policy.apps.tanzu.vmware.com` has to be excluded in all
    TAP 1.3+ installations. See [Policy controller known issues section](scst-policy/known-issues.md)

### <a id='full-profile'></a> Full Profile

To install Tanzu Application Platform with Supply Chain Basic,
you must retrieve your cluster’s base64 encoded ca certificate from `$HOME/.kube/config`.
Retrieve the `certificate-authority-data` from the respective cluster section
and input it as `B64_ENCODED_CA` in the `tap-values.yaml`.

The following is the YAML file sample for the full-profile:

```yaml
shared:
  ingress_domain: "INGRESS-DOMAIN"
  image_registry:
    project_path: "SERVER-NAME/REPO-NAME"
    username: "REGISTRY-USERNAME"
    password: "REGISTRY-PASSWORD"
    ca_cert_data: |
      -----BEGIN CERTIFICATE-----
      MIIFXzCCA0egAwIBAgIJAJYm37SFocjlMA0GCSqGSIb3DQEBDQUAMEY...
      -----END CERTIFICATE-----
profile: full
excluded_packages:
- policy.apps.tanzu.vmware.com
ceip_policy_disclosed: true
buildservice:
  kp_default_repository: "REPOSITORY"
  kp_default_repository_username: "REGISTRY-USERNAME" # Takes the value from the shared section above by default, but can be overridden by setting a different value.
  kp_default_repository_password: "REGISTRY-PASSWORD" # Takes the value from the shared section above by default, but can be overridden by setting a different value.
  exclude_dependencies: true
supply_chain: basic
scanning:
  metadataStore:
    url: ""
contour:
  infrastructure_provider: aws
  envoy:
    service:
      type: LoadBalancer
      annotations:
      # This annotation is for air-gapped AWS only.
          service.kubernetes.io/aws-load-balancer-internal: "true"

ootb_supply_chain_basic:
  registry:
      server: "SERVER-NAME" # Takes the value from the shared section above by default, but can be overridden by setting a different value.
      repository: "REPO-NAME" # Takes the value from the shared section above by default, but can be overridden by setting a different value.
  gitops:
      ssh_secret: "SSH-SECRET"
  maven:
      repository:
         url: https://MAVEN-URL
         secret_name: "MAVEN-CREDENTIALS"

accelerator:
  ingress:
    include: true
    enable_tls: false
  git_credentials:
    secret_name: git-credentials
    username: GITLAB-USER
    password: GITLAB-PASSWORD

appliveview:
  ingressEnabled: true
  tls:
    secretName: "SECRET-NAME"
    namespace: "APP-LIVE-VIEW-NAMESPACE"

appliveview_connector:
  backend:
    ingressEnabled: true
    sslDisabled: false

tap_gui:
  service_type: ClusterIP
  app_config:
    kubernetes:
      serviceLocatorMethod:
        type: multiTenant
      clusterLocatorMethods:
        - type: config
          clusters:
            - url: https://${KUBERNETES_SERVICE_HOST}:${KUBERNETES_SERVICE_PORT}
              name: host
              authProvider: serviceAccount
              serviceAccountToken: ${KUBERNETES_SERVICE_ACCOUNT_TOKEN}
              skipTLSVerify: false
              caData: B64_ENCODED_CA
    catalog:
      locations:
        - type: url
          target: https://GIT-CATALOG-URL/catalog-info.yaml
    #Example Integration for custom GitLab:
    integrations:
      gitlab:
        - host: GITLABURL
          token: <GITLAB-TOKEN>
          apiBaseUrl: https://GITLABURL/api/v4/
    backend:
      reading:
        allow:
          - host: https://GIT-CATALOG-URL/catalog-info.yaml

metadata_store:
  ns_for_export_app_cert: "MY-DEV-NAMESPACE"
  app_service_type: ClusterIP # Defaults to `LoadBalancer`. If `shared.ingress_domain` is set as above, this must be set to `ClusterIP`.

grype:
  namespace: "MY-DEV-NAMESPACE"
  targetImagePullSecret: "TARGET-REGISTRY-CREDENTIALS-SECRET"
```

Where:

- `INGRESS-DOMAIN` is the subdomain for the host name that you point at the `tanzu-shared-ingress`
service's External IP address.
- `REPOSITORY` is the fully qualified path to the Tanzu Build Service repository. This path must be writable. For example:
    * Harbor: `harbor.io/my-project/build-service`.
    * Artifactory: `artifactory.com/my-project/build-service`.
- `REGISTRY-USERNAME` and `REGISTRY-PASSWORD` are the user name and password for the internal registry.
- `SERVER-NAME` is the host name of the registry server. Examples:
    * Harbor has the form `server: "my-harbor.io"`.
    * Docker Hub has the form `server: "index.docker.io"`.
    * Google Cloud Registry has the form `server: "gcr.io"`.
- `REPO-NAME` is where workload images are stored in the registry. If this key is passed through the shared section earlier and AWS ECR registry is used, you must ensure that the `SERVER-NAME/REPO-NAME/buildservice` and `SERVER-NAME/REPO-NAME/workloads` exist. AWS ECR expects the paths to be pre-created.
Images are written to `SERVER-NAME/REPO-NAME/workload-name`. Examples:
    * Harbor has the form `repository: "my-project/supply-chain"`.
    * Docker Hub has the form `repository: "my-dockerhub-user"`.
    * Google Cloud Registry has the form `repository: "my-project/supply-chain"`.
- `SSH-SECRET` is the secret name for https authentication, certificate authority, and SSH authentication.
- `MAVEN-CREDENTIALS` is the name of [the secret with maven creds](scc/building-from-source.hbs.md#a-idmaven-repository-secreta-maven-repository-secret). This secret must be in the developer namespace. You can create it after the fact.
- `GIT-CATALOG-URL` is the path to the `catalog-info.yaml` catalog definition file. You can download either a blank or populated catalog file from the [Tanzu Application Platform product page](https://network.pivotal.io/products/tanzu-application-platform/#/releases/1043418/file_groups/6091). Otherwise, you can use a Backstage-compliant catalog you've already built and posted on the Git infrastructure.
- `GITLABURL` is the host name of your GitLab instance.
- `GITLAB-USER` is the user name of your GitLab instance.
- `GITLAB-PASSWORD` is the password for the `GITLAB-USER` of your GitLab instance. This can also be the `GITLAB-TOKEN`.
- `GITLAB-TOKEN` is the API token for your GitLab instance.
- `MY-DEV-NAMESPACE` is the name of the developer namespace. SCST - Store exports secrets to the namespace, and SCST - Scan deploys the `ScanTemplates` there. This allows the scanning feature to run in this namespace.
  - If there are multiple developer namespaces, use `ns_for_export_app_cert: "*"` to export the SCST - Store CA cert to all namespaces.
- `TARGET-REGISTRY-CREDENTIALS-SECRET` is the name of the secret that contains the
credentials to pull an image from the registry for scanning.
- `SECRET-NAME` is the name of the TLS secret for the domain consumed by HTTPProxy.
- `APP-LIVE-VIEW-NAMESPACE` is the targeted namespace for the TLS secret for the domain.

If you use custom CA certificates, you must provide one or more PEM-encoded CA certificates under the `ca_cert_data` key. If you configured `shared.ca_cert_data`, Tanzu Application Platform component packages inherits that value by default.

Create the app-live-view namespace and the TLS secret for the domain before installing the Tanzu Application Platform packages in the cluster. This ensures the HTTPProxy is updated with the TLS secret.

To create a TLS secret for app-live-view, run:

```console
kubectl create -n app-live-view secret tls alv-cert --cert=<.crt file> --key=<.key file>
```

To verify the HTTPProxy object with the TLS secret, run:

```console
kubectl get httpproxy -A
NAMESPACE            NAME                                                              FQDN                                                             TLS SECRET               STATUS   STATUS DESCRIPTION
app-live-view        appliveview                                                       appliveview.192.168.42.55.nip.io                                 app-live-view/alv-cert   valid    Valid HTTPProxy
```

## <a id="install-package"></a>Install your Tanzu Application Platform package

Follow these steps to install the Tanzu Application Platform package:

1. Install the package by running:

    ```console
    tanzu package install tap -p tap.tanzu.vmware.com -v $TAP_VERSION --values-file tap-values.yaml -n tap-install
    ```

    Where `$TAP_VERSION` is the Tanzu Application Platform version environment variable
    you defined earlier.

1. Verify the package install by running:

    ```console
    tanzu package installed get tap -n tap-install
    ```

    This may take 5-10 minutes because it installs several packages on your cluster.

1. Verify that all the necessary packages in the profile are installed by running:

    ```console
    tanzu package installed list -A
    ```

## <a id='next-steps'></a>Next steps

- [Install the Tanzu Build Service dependencies](tbs-offline-install-deps.html)
