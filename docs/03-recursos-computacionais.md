# Provisionando Recursos Computacionais

O Kubernetes requer um conjunto de máquinas para hospedar o nave de comando e os nós _worker_ onde os contêineres são efetivamente executados. Nesse lab você provisionará os recursos computacionais necessários para executar um cluster Kubernetes seguro e altamente disponível por uma única [zona computacional](https://cloud.google.com/compute/docs/regions-zones/regions-zones).

> Garanta que uma zona computacional e região padrões foram definidas como descrito no lab [Pré-requisitos](01-pre-requisitos.md#defina-uma-zona-computacional-e-região-padrões).

## Rede

O [modelo de rede](https://kubernetes.io/docs/concepts/cluster-administration/networking/#kubernetes-model)  do Kubernetes assume uma rede plana na qual contêineres e nós podem comunicar-se. Em casos onde isso não é desejado [políticas de rede](https://kubernetes.io/docs/concepts/services-networking/network-policies/) podem limitar como grupos de contêineres são autorizados a comunicar entre si e com _endpoints_ de rede externos.

> Configurar políticas de rede está fora do escopo desse tutorial.

### Rede de Nuvem Privada Virtual

Nessa seção uma rede de [Nuvem Privada Virtual](https://cloud.google.com/compute/docs/networks-and-firewalls#networks) (VPC) será configurada para hospedar o cluster do Kubernetes.

Crie a rede VPC customizada `kubernetes-the-hard-way`:

```
gcloud compute networks create kubernetes-the-hard-way --mode custom
```


Uma [sub-rede](https://cloud.google.com/compute/docs/vpc/#vpc_networks_and_subnets) deve ser provisionada dentro de uma faixa de IP grande o suficiente para atribuir um endereço de IP privado para cada nó no cluster do Kubernetes.

Crie a sub-rede `kubernetes` na rede VPC `kubernetes-the-hard-way`:

```
gcloud compute networks subnets create kubernetes \
  --network kubernetes-the-hard-way \
  --range 10.240.0.0/24
```

> O faixa de endereço de IP `10.240.0.0/24`  pode hospedar até 254 instâncias computacionais

### Regras de Firewall

Create a firewall rule that allows internal communication across all protocols:

```
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-internal \
  --allow tcp,udp,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 10.240.0.0/24,10.200.0.0/16
```

Create a firewall rule that allows external SSH, ICMP, and HTTPS:

```
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-external \
  --allow tcp:22,tcp:6443,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 0.0.0.0/0
```

> An [external load balancer](https://cloud.google.com/compute/docs/load-balancing/network/) will be used to expose the Kubernetes API Servers to remote clients.

List the firewall rules in the `kubernetes-the-hard-way` VPC network:

```
gcloud compute firewall-rules list --filter "network: kubernetes-the-hard-way"
```

> output

```
NAME                                         NETWORK                  DIRECTION  PRIORITY  ALLOW                 DENY
kubernetes-the-hard-way-allow-external       kubernetes-the-hard-way  INGRESS    1000      tcp:22,tcp:6443,icmp
kubernetes-the-hard-way-allow-internal       kubernetes-the-hard-way  INGRESS    1000      tcp,udp,icmp
```

### Endereço de IP Público do Kubernetes

Allocate a static IP address that will be attached to the external load balancer fronting the Kubernetes API Servers:

```
gcloud compute addresses create kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region)
```

Verify the `kubernetes-the-hard-way` static IP address was created in your default compute region:

```
gcloud compute addresses list --filter="name=('kubernetes-the-hard-way')"
```

> output

```
NAME                     REGION    ADDRESS        STATUS
kubernetes-the-hard-way  us-west1  XX.XXX.XXX.XX  RESERVED
```

## Instâncias Computacionais

The compute instances in this lab will be provisioned using [Ubuntu Server](https://www.ubuntu.com/server) 16.04, which has good support for the [cri-containerd container runtime](https://github.com/kubernetes-incubator/cri-containerd). Each compute instance will be provisioned with a fixed private IP address to simplify the Kubernetes bootstrapping process.

### Controladores do Kubernetes

Create three compute instances which will host the Kubernetes control plane:

```
for i in 0 1 2; do
  gcloud compute instances create controller-${i} \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-1604-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-1 \
    --private-network-ip 10.240.0.1${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-hard-way,controller
done
```

### Kubernetes Workers

Each worker instance requires a pod subnet allocation from the Kubernetes cluster CIDR range. The pod subnet allocation will be used to configure container networking in a later exercise. The `pod-cidr` instance metadata will be used to expose pod subnet allocations to compute instances at runtime.

> The Kubernetes cluster CIDR range is defined by the Controller Manager's `--cluster-cidr` flag. In this tutorial the cluster CIDR range will be set to `10.200.0.0/16`, which supports 254 subnets.

Create three compute instances which will host the Kubernetes worker nodes:

```
for i in 0 1 2; do
  gcloud compute instances create worker-${i} \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-1604-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-1 \
    --metadata pod-cidr=10.200.${i}.0/24 \
    --private-network-ip 10.240.0.2${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-hard-way,worker
done
```

### Verificação

Liste as instâncias computacionais na sua zona computacional padrão:

```
gcloud compute instances list
```

> saída

```
NAME          ZONE        MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
controller-0  us-west1-c  n1-standard-1               10.240.0.10  XX.XXX.XXX.XXX  RUNNING
controller-1  us-west1-c  n1-standard-1               10.240.0.11  XX.XXX.X.XX     RUNNING
controller-2  us-west1-c  n1-standard-1               10.240.0.12  XX.XXX.XXX.XX   RUNNING
worker-0      us-west1-c  n1-standard-1               10.240.0.20  XXX.XXX.XXX.XX  RUNNING
worker-1      us-west1-c  n1-standard-1               10.240.0.21  XX.XXX.XX.XXX   RUNNING
worker-2      us-west1-c  n1-standard-1               10.240.0.22  XXX.XXX.XX.XX   RUNNING
```

Próximo: [Provisionando a CA e Gerando Certificados TLS](04-autoridade-certificadora.md)
