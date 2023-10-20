# Aws Eks SecretsManager
Here are some examples on how to mount secrets from AWS Secrets manager in to your pod in EKS Cluster. Current documentation does not explain how to mount secrets as variable, also doe not detail how to export individual items

[AWS Secrets Manager and Config Provider](https://github.com/aws/secrets-store-csi-driver-provider-aws#usage)

> While installing CSI Driver, ***MUST*** ensure to enable sync if you like to create K8sSecret and assign to environment variables

