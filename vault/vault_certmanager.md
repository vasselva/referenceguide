# Vault Cert Manager Integration

## Vault
One of the Vault feature is PKI secret engine.  The PKI secrets engine generates dynamic X.509 certificate. With this secrets engine, services can get certificates without going through the usual manual process of generating a private key and CSR, submitting to a CA, and waiting for a verification and signing process to complete

## Cert-Manager
cert-manager is a Kubernetes add-on to automate the management and issuance of TLS certificates from various issuing sources.

It will ensure certificates are valid and up to date periodically, and attempt to renew certificates at an appropriate time before expiry.
Details on [cert manager](https://github.com/jetstack/cert-manager)

## Pre-requisties

- Kuberenetes e.g. Minikube
- Vault instance

### Steps

1. Vault PKI setup
2. Debug steps for Vault PKI setup
3. Deploy sample app in Kubernetes
    + Deployment
    + Service 
    + Ingress 
4. Test the Deploy App
5. Install Cert-manager
6. Enable cert-manager
6. Test TLS connection
7. Debug

### Vault PKI Setup

#### Enable Vault PKI as CA
```
vault secrets enable pki 
vault secrets tune -default-lease-ttl=2160h -max-lease-ttl=87600h pki
vault write pki/root/generate/internal common_name=test.com ttl=87600h
```
#### Create Vault Policies
```
path "pki/issue/cert-manager" {
  capabilities = ["read", "list", "create", "update"]
}

path "pki/sign/cert-manager" {
  capabilities = ["read", "list", "create", "update"]
}
```
#### Creat AppRole for Vault Authentication

Kubernetes authenticate to Vault before signing the CSR from cert manager

```
vault policy write cert-manager-pki cert-manager-pki.hcl
vault write pki/roles/cert-manager allowed_domains=test.com allow_subdomains=true max_ttl=8760
vault read auth/approle/role/cert-manager-role/role-id
vault write -f auth/approle/role/cert-manager-role/secret-id
```
Capture the Role ID and Secret ID for later use

### Debug steps for Vault PKI

#### Validate permission able to generate the certs
```
vault policy read cert-manager-pki
```
```
Capture Vault Role ID and Secret ID to test Authentication
vault write auth/approle/login role_id=298bc94c-30f9-5c71-32e5-9ac5c3429eb4  secret_id=f3de7834-6a23-a67f-38ff-35710ae9444c
```
```
vault write pki/issue/cert-manager common_name=hello.test.com
vault write pki/sign/cert-manager common_name=hello.test.com
```

### Deploy Sample App in Kubernetes

```
Kubectl create ns hello
```
#### Deploytment file
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
      - image: gcr.io/google-samples/node-hello:1.0
        name: hello-world
        ports:
        - containerPort: 8080
```
#### Deploy Service 
```
apiVersion: v1
kind: Service
metadata:
  name: hello
spec:
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
  selector:
    app: hello
```
#### Deploy ingress
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hello
  annotations:
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/issuer: "vault-issuer"
spec:
  tls:
  - hosts:
    - a.test.com
    secretName: hello-tls
  rules:
  - host: a.test.com
    http:
      paths:
      - path: /
        backend:
          serviceName: hello
          servicePort: 80
```
### Test the Deployment App 

```
kubectl get pods -n hello
NAME                    READY   STATUS    RESTARTS   AGE
hello-77646f9c8-slcqx   1/1     Running   0          13s
```
```
kubectl get svc -n hello
NAME    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
hello   ClusterIP   10.99.234.66   <none>        80/TCP    35s
```
```
kubectl get ingress -n hello
NAME    HOSTS        ADDRESS          PORTS     AGE
hello   a.test.com   192.168.99.101   80, 443   80s
```
```
Update /etc/hosts if required to test
192.168.99.101  a.test.com

openssl s_client -connect a.test.com:443 | grep CN=
subject=/O=Acme Co/CN=Kubernetes Ingress Controller Fake Certificate
issuer=/O=Acme Co/CN=Kubernetes Ingress Controller Fake Certificate
```

### Install Cert Manager
Refer Cert Manager Documentation

https://cert-manager.io/docs/installation/kubernetes/

Verify the installation as per below documentation

https://cert-manager.io/docs/installation/kubernetes/#verifying-the-installation

** Note - apiVersion: cert-manager.io/v1alpha2 - Kubectl may not able to display components correctly when queried

### Enable Cert Manager

Enabling cert Manager requires three manager components on testing namespace
+ Create Secret to connect to vault
+ Create Issuer to create CSR and signed by vault
+ Certificate itself

#### Create Kubernetes Secret
```
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: cert-manager-vault-approle
  namespace: hello
data:
  secretId: "ZjNkZTc4MzQtNmEyMy1hNjdmLTM4ZmYtMzU3MTBhZTk0NDRj"
```
#### Create Issuer
```
apiVersion: cert-manager.io/v1alpha2
kind: Issuer
metadata:
  name: vault-issuer
  namespace: hello
spec:
  vault:
    path: pki/sign/cert-manager
    server: http://10.0.2.2:8200 # Vault Address - Local Vault Address
    auth:
      appRole:
        path: approle
        roleId: "298bc94c-30f9-5c71-32e5-9ac5c3429eb4"
        secretRef:
          name: cert-manager-vault-approle
          key: secretId
```
#### Create certificate
```
apiVersion: cert-manager.io/v1alpha3
kind: Certificate
metadata:
  name: hello
  namespace: hello
spec:
  secretName: hello-tls ### should be the same as in ingress manifest
  issuerRef:
    name: vault-issuer
  commonName: a.test.com
  dnsNames:
  - a.test.com
```
### Testing and Debugging steps
```
kubectl get secrets -n hello -o wide
NAME                         TYPE                                  DATA   AGE
cert-manager-vault-approle   Opaque                                1      6m29s
default-token-xlnmg          kubernetes.io/service-account-token   3      44m
hello-tls                    kubernetes.io/tls                     3      5s
```

```
kubectl describe secrets hello-tls -n hello
Name:         hello-tls
Namespace:    hello
Labels:       <none>
Annotations:  cert-manager.io/alt-names: a.test.com
              cert-manager.io/certificate-name: hello
              cert-manager.io/common-name: a.test.com
              cert-manager.io/ip-sans:
              cert-manager.io/issuer-kind: Issuer
              cert-manager.io/issuer-name: vault-issuer
              cert-manager.io/uri-sans:

Type:  kubernetes.io/tls

Data
====
ca.crt:   1158 bytes
tls.crt:  2346 bytes
tls.key:  1679 bytes
```

```
kubectl logs -n cert-manager -lapp=cert-manager -f
```
### References

https://cert-manager.io/docs/installation/kubernetes/#verifying-the-installation

https://cert-manager.io/docs/configuration/vault/

https://www.arctiq.ca/our-blog/2019/4/1/how-to-use-vault-pki-engine-for-dynamic-tls-certificates-on-gke/

https://www.vaultproject.io/docs/secrets/pki/
