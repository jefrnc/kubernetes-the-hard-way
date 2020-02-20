# Provisioning a CA and Generating TLS Certificates

In this lab you will provision a [PKI Infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure) using CloudFlare's PKI toolkit, [cfssl](https://github.com/cloudflare/cfssl), then use it to bootstrap a Certificate Authority, and generate TLS certificates for the following components: etcd, kube-apiserver, kube-controller-manager, kube-scheduler, kubelet, and kube-proxy.

## Certificate Authority

In this section you will provision a Certificate Authority that can be used to generate additional TLS certificates.

Generate the CA configuration file, certificate, and private key:

```
{

cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

}
```

```
PS C:\Repos\kubernetes-the-hard-way\deployments\certificates> cfssl gencert -initca ca-csr.json | cfssljson -bare ca
2020/02/20 12:07:02 [INFO] generating a new CA key and certificate from CSR
2020/02/20 12:07:02 [INFO] generate received request
2020/02/20 12:07:02 [INFO] received CSR
2020/02/20 12:07:02 [INFO] generating key: rsa-2048
2020/02/20 12:07:03 [INFO] encoded CSR
2020/02/20 12:07:03 [INFO] signed certificate with serial number 442797245969943793060276030632295796809356798029
```
Results:

```
$ ls


    Directory: C:\Repos\kubernetes-the-hard-way\deployments\certificates


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        2/20/2020  12:05 PM            267 ca-config.json
-a----        2/20/2020  12:06 PM            253 ca-csr.json
-a----        2/20/2020  12:07 PM           1679 ca-key.pem
-a----        2/20/2020  12:07 PM           1005 ca.csr
-a----        2/20/2020  12:07 PM           1318 ca.pem
```

## Client and Server Certificates

In this section you will generate client and server certificates for each Kubernetes component and a client certificate for the Kubernetes `admin` user.

### The Admin Client Certificate

Generate the `admin` client certificate and private key:

```
{

cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:masters",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin

}
```


En powershell
```
PS C:\Repos\kubernetes-the-hard-way\deployments\certificates> cfssl gencert -ca ca.pem -ca-key ca-key.pem -config ca-config.json -profile kubernetes  admin-csr.json  | cfssljson -bare admin
2020/02/20 12:15:33 [INFO] generate received request
2020/02/20 12:15:33 [INFO] received CSR
2020/02/20 12:15:33 [INFO] generating key: rsa-2048
2020/02/20 12:15:33 [INFO] encoded CSR
2020/02/20 12:15:33 [INFO] signed certificate with serial number 112712162943755742877086230686996897978870329831
2020/02/20 12:15:33 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").
```

 
Results:

```
admin-key.pem
admin.pem
```

### The Kubelet Client Certificates

Kubernetes uses a [special-purpose authorization mode](https://kubernetes.io/docs/admin/authorization/node/) called Node Authorizer, that specifically authorizes API requests made by [Kubelets](https://kubernetes.io/docs/concepts/overview/components/#kubelet). In order to be authorized by the Node Authorizer, Kubelets must use a credential that identifies them as being in the `system:nodes` group, with a username of `system:node:<nodeName>`. In this section you will create a certificate for each Kubernetes worker node that meets the Node Authorizer requirements.

Generate a certificate and private key for each Kubernetes worker node:

```
for instance in worker-0 worker-1; do
cat > ${instance}-csr.json <<EOF
{
  "CN": "system:node:${instance}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

EXTERNAL_IP=$(gcloud compute instances describe ${instance} \
  --format 'value(networkInterfaces[0].accessConfigs[0].natIP)')

INTERNAL_IP=$(gcloud compute instances describe ${instance} \
  --format 'value(networkInterfaces[0].networkIP)')

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${instance},${EXTERNAL_IP},${INTERNAL_IP} \
  -profile=kubernetes \
  ${instance}-csr.json | cfssljson -bare ${instance}
done
```

$instance = "worker-1"
$INTERNAL_IP=$(gcloud compute instances describe ${instance} --format 'value(networkInterfaces[0].accessConfigs[0].natIP)') 
$EXTERNAL_IP=$(gcloud compute instances describe ${i]nstance} --format 'value(networkInterfaces[0].networkIP)')
 
cfssl gencert -ca ca.pem -ca-key ca-key.pem -config ca-config.json -hostname ${instance},$EXTERNAL_IP,$INTERNAL_IP -profile kubernetes ${instance}-csr.json | cfssljson -bare ${instance}

```
PS C:\Repos\kubernetes-the-hard-way\deployments\certificates> cfssl gencert -ca ca.pem -ca-key ca-key.pem -config ca-config.json 
-hostname ${instance},$EXTERNAL_IP,$INTERNAL_IP -profile kubernetes ${instance}-csr.json | cfssljson -bare ${instance}
2020/02/20 12:37:26 [INFO] generate received request
2020/02/20 12:37:26 [INFO] received CSR
2020/02/20 12:37:26 [INFO] generating key: rsa-2048
2020/02/20 12:37:26 [INFO] encoded CSR
2020/02/20 12:37:26 [INFO] signed certificate with serial number 716200621200550963418401329895159922150473628077


$INTERNAL_IP=$(gcloud compute instances describe worker-1 --format 'value(networkInterfaces[0].accessConfigs[0].natIP)') 
$EXTERNAL_IP=$(gcloud compute instances describe worker-1 --format 'value(networkInterfaces[0].networkIP)')

cfssl gencert -ca ca.pem -ca-key ca-key.pem -config ca-config.json  -hostname worker-1,$EXTERNAL_IP,$INTERNAL_IP -profile kubernetes worker-1-csr.json | cfssljson -bare worker-1


```

Results:

```
worker-0-key.pem
worker-0.pem
worker-1-key.pem
worker-1.pem
worker-2-key.pem
worker-2.pem
```

### The Controller Manager Client Certificate

Generate the `kube-controller-manager` client certificate and private key:

```
{

cat > kube-controller-manager-csr.json <<EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-controller-manager",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager

}
```

Powershell
```
S C:\Repos\kubernetes-the-hard-way\deployments\certificates> cfssl gencert -ca ca.pem -ca-key ca-key.pem -config ca-config.json 
-profile kubernetes kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager
2020/02/20 12:40:18 [INFO] generate received request
2020/02/20 12:40:18 [INFO] received CSR
2020/02/20 12:40:18 [INFO] generating key: rsa-2048
2020/02/20 12:40:19 [INFO] encoded CSR
2020/02/20 12:40:19 [INFO] signed certificate with serial number 204848304204613875897381309497922704295430947869
2020/02/20 12:40:19 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").
```

Results:

```
kube-controller-manager-key.pem
kube-controller-manager.pem
```


### The Kube Proxy Client Certificate

Generate the `kube-proxy` client certificate and private key:

```
{

cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:node-proxier",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy

}
```

Powershell
```
PS C:\Repos\kubernetes-the-hard-way\deployments\certificates> cfssl gencert -ca ca.pem -ca-key ca-key.pem -config ca-config.json 
-profile kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy
2020/02/20 12:43:55 [INFO] generate received request
2020/02/20 12:43:55 [INFO] received CSR
2020/02/20 12:43:55 [INFO] generating key: rsa-2048
2020/02/20 12:43:55 [INFO] encoded CSR
2020/02/20 12:43:55 [INFO] signed certificate with serial number 601918434556005684405799074630388509260784944254
2020/02/20 12:43:55 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").
```


Results:

```
kube-proxy-key.pem
kube-proxy.pem
```

### The Scheduler Client Certificate

Generate the `kube-scheduler` client certificate and private key:

```
{

cat > kube-scheduler-csr.json <<EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-scheduler",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler

}
```

Powershell
```
PS C:\Repos\kubernetes-the-hard-way\deployments\certificates> cfssl gencert -ca ca.pem -ca-key ca-key.pem -config ca-config.json 
-profile kubernetes kube-scheduler-csr.json | cfssljson -bare kube-scheduler
2020/02/20 12:48:53 [INFO] generate received request
2020/02/20 12:48:53 [INFO] received CSR
2020/02/20 12:48:53 [INFO] generating key: rsa-2048
2020/02/20 12:48:53 [INFO] encoded CSR
2020/02/20 12:48:53 [INFO] signed certificate with serial number 178643106759854782720106517255774453007825702692
2020/02/20 12:48:53 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").
```


Results:

```
kube-scheduler-key.pem
kube-scheduler.pem
```


### The Kubernetes API Server Certificate

The `kubernetes-the-hard-way` static IP address will be included in the list of subject alternative names for the Kubernetes API Server certificate. This will ensure the certificate can be validated by remote clients.

Generate the Kubernetes API Server certificate and private key:

```
{

KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region) \
  --format 'value(address)')

KUBERNETES_HOSTNAMES=kubernetes,kubernetes.default,kubernetes.default.svc,kubernetes.default.svc.cluster,kubernetes.svc.cluster.local

cat > kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=10.32.0.1,10.240.0.10,10.240.0.11,10.240.0.12,${KUBERNETES_PUBLIC_ADDRESS},127.0.0.1,${KUBERNETES_HOSTNAMES} \
  -profile=kubernetes \
  kubernetes-csr.json | cfssljson -bare kubernetes

}
```


Powershell
```
{

$KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way --region $(gcloud config get-value compute/region) --format 'value(address)')

$KUBERNETES_HOSTNAMES="kubernetes,kubernetes.default,kubernetes.default.svc,kubernetes.default.svc.cluster,kubernetes.svc.cluster.local"
 
cfssl gencert -ca ca.pem -ca-key ca-key.pem -config ca-config.json 
-hostname 10.32.0.1,10.240.0.10,10.240.0.11,10.240.0.12,$KUBERNETES_PUBLIC_ADDRESS,127.0.0.1,$KUBERNETES_HOSTNAMES -profile kubernetes kubernetes-csr.json | cfssljson -bare kubernetes
2020/02/20 12:52:35 [INFO] generate received request
2020/02/20 12:52:35 [INFO] received CSR
2020/02/20 12:52:35 [INFO] generating key: rsa-2048
2020/02/20 12:52:36 [INFO] encoded CSR
2020/02/20 12:52:36 [INFO] signed certificate with serial number 143287199578473065350415820796734076238444439283

```

> The Kubernetes API server is automatically assigned the `kubernetes` internal dns name, which will be linked to the first IP address (`10.32.0.1`) from the address range (`10.32.0.0/24`) reserved for internal cluster services during the [control plane bootstrapping](08-bootstrapping-kubernetes-controllers.md#configure-the-kubernetes-api-server) lab.

Results:

```
kubernetes-key.pem
kubernetes.pem
```

## The Service Account Key Pair

The Kubernetes Controller Manager leverages a key pair to generate and sign service account tokens as described in the [managing service accounts](https://kubernetes.io/docs/admin/service-accounts-admin/) documentation.

Generate the `service-account` certificate and private key:

```
{

cat > service-account-csr.json <<EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account

}
```

Powershell
```
cfssl gencert -ca ca.pem -ca-key ca-key.pem -config ca-config.json 
-profile kubernetes service-account-csr.json | cfssljson -bare service-account
2020/02/20 12:54:28 [INFO] generate received request
2020/02/20 12:54:28 [INFO] received CSR
2020/02/20 12:54:28 [INFO] generating key: rsa-2048
2020/02/20 12:54:28 [INFO] encoded CSR
2020/02/20 12:54:28 [INFO] signed certificate with serial number 307095887903712464308280071040490174956389025702
2020/02/20 12:54:29 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").
```

Results:

```
service-account-key.pem
service-account.pem
```


## Distribute the Client and Server Certificates

Copy the appropriate certificates and private keys to each worker instance:

```
for instance in worker-0 worker-1 ; do
  gcloud compute scp ca.pem ${instance}-key.pem ${instance}.pem ${instance}:~/
done
```

Poweshell
```
PS C:\Repos\kubernetes-the-hard-way\deployments\certificates> gcloud compute scp ca.pem worker-0-key.pem worker-0.pem worker-0:
ca.pem                    | 1 kB |   1.3 kB/s | ETA: 00:00:00 | 100%
worker-0-key.pem          | 1 kB |   1.6 kB/s | ETA: 00:00:00 | 100%

PS C:\Repos\kubernetes-the-hard-way\deployments\certificates> gcloud compute scp ca.pem worker-1-key.pem worker-1.pem worker-1:
ca.pem                    | 1 kB |   1.3 kB/s | ETA: 00:00:00 | 100%
worker-1-key.pem          | 1 kB |   1.6 kB/s | ETA: 00:00:00 | 100%
worker-1.pem              | 1 kB |   1.5 kB/s | ETA: 00:00:00 | 100%
```

Copy the appropriate certificates and private keys to each controller instance:

```
for instance in controller-0 controller-1; do
  gcloud compute scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
    service-account-key.pem service-account.pem ${instance}:~/
done
```

Powershell
 
```
 
gcloud compute scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem service-account-key.pem service-account.pem controller-0:./
gcloud compute scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem service-account-key.pem service-account.pem controller-1:./
```

> The `kube-proxy`, `kube-controller-manager`, `kube-scheduler`, and `kubelet` client certificates will be used to generate client authentication configuration files in the next lab.

Next: [Generating Kubernetes Configuration Files for Authentication](05-kubernetes-configuration-files.md)
