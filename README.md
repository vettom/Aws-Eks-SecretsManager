# Aws Eks: K8s secret from Secrets manager and mount as Environment Variables

There are 3 parts to creating K8s secrets from AWS Secrets Manager objects

- Install CSI driver with --set syncSecret.enabled=true
- Configure necessary objects in SecretProviderClass.parametes
- Create Secret Objects to create k8s secrets SecretProviderClass.secretObject

> CSI Driver ***MUST*** be installed with ***--set syncSecret.enabled=true***


## Install CSI Driver and AWS Provider with Helm chart
```bash
# Add CSI driver and AWS provider repo and update
helm repo add secrets-store-csi-driver https://kubernetes-sigs.github.io/secrets-store-csi-driver/charts
helm repo add aws-secrets-manager https://aws.github.io/secrets-store-csi-driver-provider-aws
helm repo update secrets-store-csi-driver aws-secrets-manager

# Install CSI driver with option --set syncSecret.enabled=true
helm install -n kube-system csi-secrets-store secrets-store-csi-driver/secrets-store-csi-driver  --set syncSecret.enabled=true
helm install -n kube-system secrets-provider-aws aws-secrets-manager/secrets-store-csi-driver-provider-aws

```

Follow instructions here [AWS Secrets Manager and Config Provider](https://github.com/aws/secrets-store-csi-driver-provider-aws#usage) to create secret, policy and create IAM Service Account.

## Scenario
- You have configured Secret called MySecret with data username and password
- Necessary policy created in AWS to allow access to Secret
- Iamservice account called 'nginx-deployment-sa' created and policy attached

## Creating K8s Secrets from AWS Secrets

```bash
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: aws-secret-to-k8s-secret
  namespace: default
spec:
  provider: aws
  parameters:
    objects: |
        - objectName: "MySecrets"
          objectType: "secretsmanager"
          objectAlias: mysecrets
          jmesPath:
            - path: "username"
              objectAlias: "Username"
  secretObjects:
    - secretName: myusername
      type: Opaque
      data: 
        - objectName: "Username"
          key: "username"
    - secretName: myk8ssecrets
      type: Opaque
      data: 
        - objectName: "mysecrets"
          key: "mysecrets"

```
#### spec.parameters
  - objectName: Name of the secret object in secretStore
  - objectAlias: Optional Alias name for secretObject.
  - jmesPath.path : Name of specific secret to be exposed
  - jmesPath.objectAlias: Alias name for the seccret to be used.

#### spec.secretObjects
  - secretName: Name of the secret to be created in k8s
  - data.objectName: Name of the secretObject/Alias to retrieve data from
  - key: Name of the key with in k8s secret to be used for storing retrieved data.
