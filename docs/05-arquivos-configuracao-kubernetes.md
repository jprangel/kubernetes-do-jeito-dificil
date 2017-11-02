# Gerando Arquivos de Configuração do Kubernetes para Autenticação

Nesse lab você irá criar [arquivos de configuração do Kubernetes](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/), também conhecidos como kubeconfigs, que possibilitam clientes Kubernetes localizarem e autenticarem com os Servidores de API do Kubernetes.

## Configurações de Autenticação do Cliente

Nessa seção você irá criar arquivos kubeconfig para os clientes `kubelet` e `kube-proxy`.

> O `agendador` e o `gerenciador de controladora` acessam o Servidor de API do Kubernetes localmente através de uma porta de API insegura que não requer autenticação. A porta insegura do Servidor de API do Kubernetes só está habilitada para acesso local.

### Endereço de IP público do Kubernetes

Cada kubeconfig requer um Servidor de API do Kubernetes para se conectar. Para suportar alta disponibilidade o endereço de IP vinculado ao balanceador de carga externo fazendo frente ao Servidor de API do Kubernetes será utilizado.

Recupere o endereço de IP estático do `kubernetes-the-hard-way`:

```
ENDERECO_PUBLICO_KUBERNETES=$(gcloud compute addresses describe kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region) \
  --format 'value(address)')
```

### O Arquivo kubelet de Configuração do Kubernetes

Ao criar os arquivos kubeconfig, o certificado de cliente que corresponde com o nome do nó do Kubelet deve ser utilizado. Isso irá garantir que os Kubelets serão devidamente autorizados pelo [Autorizador de Nós](https://kubernetes.io/docs/admin/authorization/node/) do Kubernetes.

Crie um arquivo kubeconfig para cada nó _worker_:

```
for instance in worker-0 worker-1 worker-2; do
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${ENDERECO_PUBLICO_KUBERNETES}:6443 \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-credentials system:node:${instance} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:${instance} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=${instance}.kubeconfig
done
```

Resultados:

```
worker-0.kubeconfig
worker-1.kubeconfig
worker-2.kubeconfig
```

### O Arquivo de Configuração kube-proxy do Kubernetes

Crie um arquivo kubeconfig para o serviço `kube-proxy`:

```
kubectl config set-cluster kubernetes-the-hard-way \
  --certificate-authority=ca.pem \
  --embed-certs=true \
  --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443 \
  --kubeconfig=kube-proxy.kubeconfig
```

```
kubectl config set-credentials kube-proxy \
  --client-certificate=kube-proxy.pem \
  --client-key=kube-proxy-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-proxy.kubeconfig
```

```
kubectl config set-context default \
  --cluster=kubernetes-the-hard-way \
  --user=kube-proxy \
  --kubeconfig=kube-proxy.kubeconfig
```

```
kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
```

## Distribua os Arquivos de Configuração do Kubernetes

Copie os arquivos kubeconfig `kubelet` e `kube-proxy` apropriados para cada instância _worker_:

```
for instance in worker-0 worker-1 worker-2; do
  gcloud compute scp ${instance}.kubeconfig kube-proxy.kubeconfig ${instance}:~/
done
```

Próximo: [Gerando a Configuração e a Chave de Encriptação de Dados](06-chaves-encriptacao-dados.md)
