# Commerce Standalone Helm Chart

This Helm chart alllows you to install a standalone instance of SAP Commerce (solr, hsqldb + hybris) on Kubernetes.

Some important things we do in the Chart configuration:

- use a persistant volume so that changes are not lost when the instance restarts (e.g. creating new customers, orders etc)
- local.properties is managed as a ConfigMap so changes can easily be made to the configuration without needing to touch the base image
- license file is also managed as a ConfigMap so you can easily add your own license without needing to touch the base image
- The system comes pre-initialized so once the server starts, it's ready to be used

## Installation

- Clone this repository
- Goto the support lauchpad and get a license key for SAP Commerce
- Create the config map containing the license.

  ```shell
  kubectl create configmap hybris-license --from-file=<path-to-CPS.txt> --dry-run -o yaml > templates/cm-hybris-license.yaml
  ```

- Create Kubernetes secret required for pulling the image.

  ```shell
  kubectl create secret docker-registry ycommerce-reg --docker-server=docker.io --docker-username=<your-name --docker-password=<your-pword> --docker-email=<your-email> --dry-run -o yaml > templates/secret.yaml
  ```

- Change the `kymaCluster` in `values.yaml` to match your cluster.
- Change the `HYBRIS_DATA_DIR` in `values.yaml` to match your data.zip packaging.
- Execute these commands to get the helm certificates from your cluster

  ```shell
  kubectl get -n kyma-installer secret helm-secret -o jsonpath="{.data['global\.helm\.ca\.crt']}" | base64 --decode > "$(helm home)/ca.pem";
  kubectl get -n kyma-installer secret helm-secret -o jsonpath="{.data['global\.helm\.tls\.crt']}" | base64 --decode > "$(helm home)/cert.pem";
  kubectl get -n kyma-installer secret helm-secret -o jsonpath="{.data['global\.helm\.tls\.key']}" | base64 --decode > "$(helm home)/key.pem";
  ```

- Execute `helm install . --tls`

  > Note: It will probably take about 20-30 minutes before the server is available. Check the logs of the container to determine when the server is started.

The storefront will be accessible at:
`https://electronics.<kymaCluster>/yacceleratorstorefront`

For the backoffice:
`https://backoffice.<kymaCluster>/backoffice` (admin/nimda)

For the admin console
`https://<name>.<kymaCluster>/hac` (admin/nimda)

## Post Installation

The Commerce system is already pre-configured to be paired with Kyma. The post installation steps are simply to 

1. Create the Application in Kyma 
2. Copy the one-click integration URL
3. Login to the backoffice
4. Goto System -> API -> Credential -> Consumed Certificate Credential and use the one click URL to retrieve the certificate for the 'kyma-cert' crendential








