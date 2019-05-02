# Set Up an Nginx Ingress with Cert-Manager

We will use Helm to install Cert Manager to our Cluster. Cert-Manager is a Kubernetes native certificate manager. One of the most significant features that Cert-Manager provides is its ability to automatically provision TLS certificates. Based on the annotations in a Kubernetes ingress resource, the cert-manager will talk to Let’s Encrypt and acquire a certificate on your service’s behalf.

`Note` : Ensure that you are using Helm v2.12.1 or later.

`Prerequisites ` : 

* A Kubernetes cluster version 1.8+
* The kubectl CLI installed and configured
* Helm and Tiller should be installed.

### 1. Connect the cluster :

```
 gcloud container clusters get-credentials yourclustername --zone zonename --project projectname
 ```

 ### 2. Create role for accessing helm to the cluster :

```
 kubectl create clusterrolebinding tiller-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:default
 ```
```
helm  init
```

### 3. Install the CustomResourceDefinition resources separately :

```
 kubectl apply  -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.6/deploy/manifests/00-crds.yaml
```

### 4. Label the cert-manager namespace to disable resource validation :
Next, we’ll add a label to the kube-system namespace, where we’ll install cert-manager, to enable advanced resource validation using a webhook.


```
kubectl label namespace kube-system certmanager.k8s.io/disable-validation="true"
```

### 5. Install the cert-manager Helm chart :
Finally, we can install the cert-manager Helm chart into the kube-system namespace:

```
 helm install --name cert-manager --namespace kube-system stable/cert-manager
```

### 6. Create Production Issuer file : 
Begin by creating a yaml file named “prod_issuer.yaml” and add the following text to it:

```
apiVersion: certmanager.k8s.io/v1alpha1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: vishal.v@zuru.tech
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod
    # Enable the HTTP-01 challenge provider
    http01: {}
```

`email` = your email id.

We then specify an email address to register the certificate, and create a Kubernetes Secret called letsencrypt-staging to store the ACME account's private key. We also enable the HTTP-01 challenge mechanism.


### 7. Apply the issuer file : 

```
kubectl apply -f prod_issuer.yaml
```


### 8. Make the following changes in ingress file :

`certmanager.k8s.io/cluster-issuer: letsencrypt-prod`

```
spec:
  tls:
  - hosts:
    - vishalvyas.com
    - imvishalvyas.com
    secretName: letsencrypt-prod
```

We will now perform a test using curl to verify that HTTPS is working correctly.

```
curl https://example.com
```

We have successfully configured HTTPS using a Let's Encrypt certificate for our Nginx Ingress.
