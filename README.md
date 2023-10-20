# Aws Eks mount secrets as Environment Variables
Here are some examples on how to mount secrets from AWS Secrets manager in to your pod in EKS Cluster. Current documentation does not explain how to mount secrets as variable, also doe not detail how to export individual items


> While installing CSI Driver, ***MUST*** ensure to enable sync if you like to create K8sSecret and assign to environment variables


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

### Exporting secrets as file
SecretProviderClass spec:
```yaml
  parameters:
    objects: |
        - objectName: "MySecret"  # Name of the secret in AWS secretsmanager
          objectType: "secretsmanager"
          objectAlias: secret-token      # Optional Alias name. This will be the file name on the pod
          jmesPath:                      # Use jmesPath to create file with individual secret value
            - path: "username"           # Name of the secret object in MySecret
              objectAlias: "Username"    # Name of the file that will contain the value
            - path: "password"
              objectAlias: "MyPassword"

```

### K8s Secrets for AWS secret / mounting as environment variable
SecretProviderClass spec:
'parameters' section lets you mount secrts as file. In oder to create K8s secrets and to mount as variable, add 'secretsObjects section'

```yaml
  secretObjects:
    - secretName: username					# Name of the k8s secret to create	
      type: Opaque
      data: 
        - objectName: "Username"          # This is the ObjectName or obJectAlias defined in the parameters section
          key: "myusername"				  # With in k8s secret data will be in this key
 # Below example creates k8s secret names 'mysecret' and all the values will be stored as 'secretData'
    - secretName: mysecret
      type: Opaque
      data: 
        - objectName: "secret-token"
          key: "secretData"
```