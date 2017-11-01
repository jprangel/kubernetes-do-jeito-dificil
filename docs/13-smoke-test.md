# Smoke Test (Teste de Fumaça)

Nesse lab você irá completar uma série de tarefas para garantir que seu cluster Kubernetes está funcionando corretamente.

## Encriptação de Dados

Nessa seção você irá verificar a habilidade de [encriptar dados secretos em repouso](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#verifying-that-data-is-encrypted).

Crie um _secret_ (segredo) genérico:

```
kubectl create secret generic kubernetes-the-hard-way \
  --from-literal="mykey=mydata"
```

Imprima um hexdump da _secret_ `kubernetes-the-hard-way` armazenada no etcd:

```
gcloud compute ssh controller-0 \
  --command "ETCDCTL_API=3 etcdctl get /registry/secrets/default/kubernetes-the-hard-way | hexdump -C"
```

> saída

```
00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
00000010  73 2f 64 65 66 61 75 6c  74 2f 6b 75 62 65 72 6e  |s/default/kubern|
00000020  65 74 65 73 2d 74 68 65  2d 68 61 72 64 2d 77 61  |etes-the-hard-wa|
00000030  79 0a 6b 38 73 3a 65 6e  63 3a 61 65 73 63 62 63  |y.k8s:enc:aescbc|
00000040  3a 76 31 3a 6b 65 79 31  3a 70 88 d8 52 83 b7 96  |:v1:key1:p..R...|
00000050  04 a3 bd 7e 42 9e 8a 77  2f 97 24 a7 68 3f c5 ec  |...~B..w/.$.h?..|
00000060  9e f7 66 e8 a3 81 fc c8  3c df 63 71 33 0a 87 8f  |..f.....<.cq3...|
00000070  0e c7 0a 0a f2 04 46 85  33 92 9a 4b 61 b2 10 c0  |......F.3..Ka...|
00000080  0b 00 05 dd c3 c2 d0 6b  ff ff f2 32 3b e0 ec a0  |.......k...2;...|
00000090  63 d3 8b 1c 29 84 88 71  a7 88 e2 26 4b 65 95 14  |c...)..q...&Ke..|
000000a0  dc 8d 59 63 11 e5 f3 4e  b4 94 cc 3d 75 52 c7 07  |..Yc...N...=uR..|
000000b0  73 f5 b4 b0 63 aa f9 9d  29 f8 d6 88 aa 33 c4 24  |s...c...)....3.$|
000000c0  ac c6 71 2b 45 98 9e 5f  c6 a4 9d a2 26 3c 24 41  |..q+E.._....&<$A|
000000d0  95 5b d3 2c 4b 1e 4a 47  c8 47 c8 f3 ac d6 e8 cb  |.[.,K.JG.G......|
000000e0  5f a9 09 93 91 d7 5d c9  c2 68 f8 cf 3c 7e 3b a3  |_.....]..h..<~;.|
000000f0  db d8 d5 9e 0c bf 2a 2f  58 0a                    |......*/X.|
000000fa
```

A chave etcd deveria estar prefixada com `k8s:enc:aescbc:v1:key1`, o que indica que o provedor `aescbc` foi utilizado para encriptar os dados com a chave de encriptação `key1`.

## Implantações

Nessa seção você irá verificar a abilidade de criar e gerenciar [Implantações](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

Crie uma implantação para o servidor web [nginx](https://nginx.org/en/):

```
kubectl run nginx --image=nginx
```

Liste os _pods_ criados pela implantação `nginx`:

```
kubectl get pods -l run=nginx
```

> saída

```
NAME                     READY     STATUS    RESTARTS   AGE
nginx-4217019353-b5gzn   1/1       Running   0          15s
```

### Encaminhamento de Porta (_Forwarding_)

Nessa seção você irá verificar a habilidade de acessar aplicações remotamente utilizando [encaminhamento de porta](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/).

Recupere o nome completo do pod `nginx`:

```
POD_NAME=$(kubectl get pods -l run=nginx -o jsonpath="{.items[0].metadata.name}")
```

Encaminhe a porta `8080` na sua máquina local para a porta `80` do pod do `nginx`:

```
kubectl port-forward $POD_NAME 8080:80
```

> saída

```
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

Em um novo terminal, faça uma solicitação HTTP utilizando o endereço de encaminhamento:

```
curl --head http://127.0.0.1:8080
```

> saída

```
HTTP/1.1 200 OK
Server: nginx/1.13.5
Date: Mon, 02 Oct 2017 01:04:20 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 08 Aug 2017 15:25:00 GMT
Connection: keep-alive
ETag: "5989d7cc-264"
Accept-Ranges: bytes
```

Retorne ao terminal anterior e pare o encaminhamento de porta para o pod `nginx`:

```
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
^C
```

### Logs

Nessa seção você irá verificar a abilitade de [recuperar logs de contêineres](https://kubernetes.io/docs/concepts/cluster-administration/logging/).

Imprima os logs do pod `nginx`:

```
kubectl logs $POD_NAME
```

> saída

```
127.0.0.1 - - [02/Oct/2017:01:04:20 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.54.0" "-"
```

### Executar

Nessa seção você irá verificar a habilidade de [executar comandos em um contêiner](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/#running-individual-commands-in-a-container).

Imprima a versão do nginx executando o comando `nginx -v` no contêiner `nginx`:

```
kubectl exec -ti $POD_NAME -- nginx -v
```

> saída

```
nginx version: nginx/1.13.5
```

## Serviços

Nessa seção você irá verificar a habilidade de expôr aplicações utilizando um [Serviço](https://kubernetes.io/docs/concepts/services-networking/service/).

Exponha a implantação `nginx` utilizando um serviço de [NodePort (Porta de Nó)](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport):

```
kubectl expose deployment nginx --port 80 --type NodePort
```

> O tipo de serviço Balanceador de Carga não pode ser utilizado porque seu cluster não está configurado com [integração de provedor de nuvem](https://kubernetes.io/docs/getting-started-guides/scratch/#cloud-provider). Configurar uma integração de provedor de nuvem está fora do escopo desse tutorial.

Recupere a porta do nó atribuída ao serviço do `nginx`:

```
NODE_PORT=$(kubectl get svc nginx \
  --output=jsonpath='{range .spec.ports[0]}{.nodePort}')
```

Crie uma regra de firewall que permita acesso remoto à porta de nó do `nginx`:

```
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-nginx-service \
  --allow=tcp:${NODE_PORT} \
  --network kubernetes-the-hard-way
```

Recupere o endereço de IP externo de uma instância _worker_:

```
EXTERNAL_IP=$(gcloud compute instances describe worker-0 \
  --format 'value(networkInterfaces[0].accessConfigs[0].natIP)')
```

Faça uma requisição HTTP utilizando o endereço de IP externo e a porta de nó do `nginx`:

```
curl -I http://${EXTERNAL_IP}:${NODE_PORT}
```

> saída

```
HTTP/1.1 200 OK
Server: nginx/1.13.5
Date: Mon, 02 Oct 2017 01:06:11 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 08 Aug 2017 15:25:00 GMT
Connection: keep-alive
ETag: "5989d7cc-264"
Accept-Ranges: bytes
```

Próximo: [Limpeza](14-limpeza.md)
