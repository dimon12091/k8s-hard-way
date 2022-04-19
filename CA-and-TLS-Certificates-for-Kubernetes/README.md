# Creating a Certificate Authority and TLS Certificates for Kubernetes
## Introduction
The various components of Kubernetes require certificates in order to authenticate with one another. Provisioning a certificate authority and using it to generate those certificates is a necessary step in bootstrapping a Kubernetes cluster from scratch.

In this hands-on lab, we will provision a certificate authority and generate the certificates Kubernetes needs. We will complete the following tasks:

* Provision a certificate authority (CA)
* Generate Kubernetes client certificates and kubelet client certificates for two worker nodes
* Generate a Kubernetes API server certificate
* Generate a Kubernetes service account key pair

## Solution
Navigate to the lab instructions page, and copy the Workspace Public IP address to your clipboard.

Open your terminal application and run the following command:

```ssh cloud_user@<PUBLIC_IP_ADDRESS>```

Type `yes` at the prompt.

Enter your password.

## Provision the Certificate Authority (CA)

```aidl
cat > ca-config.json << EOF
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

cat > ca-csr.json << EOF
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
```
## Generate the Kubernetes Client Certificates and Kubelet Client Certificates for Two Worker Nodes
### Generate the Admin Client Certificate

```aidl
cat > admin-csr.json << EOF
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
```

### Generate the Kubelet Client Certificates

```aidl
cat > worker0.mylabserver.com-csr.json << EOF
{
  "CN": "system:node:worker0.mylabserver.com",
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

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=172.34.1.0,worker0.mylabserver.com \
  -profile=kubernetes \
  worker0.mylabserver.com-csr.json | cfssljson -bare worker0.mylabserver.com

cat > worker1.mylabserver.com-csr.json << EOF
{
  "CN": "system:node:worker1.mylabserver.com",
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

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=172.34.1.1,worker1.mylabserver.com \
  -profile=kubernetes \
  worker1.mylabserver.com-csr.json | cfssljson -bare worker1.mylabserver.com
```

### Generate the Kube-Controller-Manager Client Certificate

```aidl
cat > kube-controller-manager-csr.json << EOF
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
```

### Generate the Kube-Proxy Client Certificate

```aidl
cat > kube-proxy-csr.json << EOF
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
```

### Generate the Kube-Scheduler Client Certificate

```aidl
cat > kube-scheduler-csr.json << EOF
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
```

### Generate the Kubernetes API Server Certificate

```aidl
cat > kubernetes-csr.json << EOF
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
  -hostname=10.32.0.1,172.34.0.0,controller0.mylabserver.com,172.34.0.1,controller1.mylabserver.com,172.34.2.0,kubernetes.mylabserver.com,127.0.0.1,localhost,kubernetes.default \
  -profile=kubernetes \
  kubernetes-csr.json | cfssljson -bare kubernetes
```

### Generate a Kubernetes Service Account Key Pair

```aidl
cat > service-account-csr.json << EOF
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
```

## Additional Resources

Your team is working on setting up a Kubernetes cluster with two controllers and two worker nodes. To enable all the components of Kubernetes to securely authenticate with each other, your team needs to provision a certificate authority and generate several certificates using that authority. Your task is to create the certificate authority and the necessary certificates.

You will need to log into the learning activity server using the Workspace Public IP. This server already has cfssl installed, so there is no need to install it.
In order to accomplish this, you need to:
* Provision the certificate authority (CA)
* Generate the necessary Kubernetes client certs, as well as kubelet client certs for two worker nodes.
* Generate the Kubernetes API server certificate.
* Generate a Kubernetes service account key pair.

Here is the cluster architecture for which you will need to generate certificates. Note that these are not real servers, just values that we will use for the purposes of this activity.
Controllers and Workers Nodes.