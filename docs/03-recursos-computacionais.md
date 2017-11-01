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

Crie uma regra de firewall que permita comunicação interna entre todos os protocolos:

```
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-internal \
  --allow tcp,udp,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 10.240.0.0/24,10.200.0.0/16
```

Crie uma regra de firewall que permita SSH, ICMP e HTTPS externos:

```
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-external \
  --allow tcp:22,tcp:6443,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 0.0.0.0/0
```


> Um [balanceador de carga externo](https://cloud.google.com/compute/docs/load-balancing/network/) será utilizado para expôr os Servidores de API do Kubernetes para clientes remotos.

Liste as regras de firewall na rede VPC `kubernetes-the-hard-way`:

```
gcloud compute firewall-rules list --filter "network: kubernetes-the-hard-way"
```

> saída

```
NAME                                         NETWORK                  DIRECTION  PRIORITY  ALLOW                 DENY
kubernetes-the-hard-way-allow-external       kubernetes-the-hard-way  INGRESS    1000      tcp:22,tcp:6443,icmp
kubernetes-the-hard-way-allow-internal       kubernetes-the-hard-way  INGRESS    1000      tcp,udp,icmp
```

### Endereço de IP Público do Kubernetes

Aloque um endereço de IP estático que será vinculado ao balanceador de carga externo que fará face aos Servidores de API do Kubernetes:

```
gcloud compute addresses create kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region)
```

Verifique se um endereço estático de IP do `kubernetes-the-hard-way` foi criado na sua zona computacional padrão:

```
gcloud compute addresses list --filter="name=('kubernetes-the-hard-way')"
```

> saída

```
NAME                     REGION    ADDRESS        STATUS
kubernetes-the-hard-way  us-west1  XX.XXX.XXX.XX  RESERVED
```

## Instâncias Computacionais

As instâncias computacionais nesse lab serão provisionadas utilizando o [Ubuntu Server](https://www.ubuntu.com/server) 16.04, que tem um bom suporte para o [_runtime_ de contêiner cri-containerd](https://github.com/kubernetes-incubator/cri-containerd). Cada instância computacional será provisionada com um endereço de IP fixo para simplificar o processo de colocar no ar o Kubernetes.

### Controladores do Kubernetes

Crie três instâncias computacionais que hospedarão a nave de controle do Kubernetes:

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

### Kubernetes _Workers_

Cada instância de _worker_ requer uma alocação de sub-rede de _pod_ da faixa CIDR do cluster Kubernetes. A alocação de sub-rede do _pod_ será utilizada para configurar a rede dos contêineres num exercício posterior. Os metadados da instância `pod-cidr` serão utilizados para expor as alocações de sub-rede de _pod_ para instâncias computacionais em tempo de execução.

> A faixa de CIDR do cluster Kubernetes é definida pela flag `--cluster-cidr`  do Gerenciador da Controladora. Nesse tutorial a faixa de CIDR será definida como `10.200.0.0/16`, que suporta 254 sub-redes.

Crie três instâncias computacionais que hospedarão os nós de _worker_ do Kubernetes:

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
