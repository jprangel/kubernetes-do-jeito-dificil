# Gerando a Configuração e a Chave de Encriptação de Dados

O Kubernetes armazena uma variedade de dados incluindo estado do cluster, configurações de aplicação e segredos. O Kubernetes suporta a habilidade de [encriptar](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data) dados em repouso no cluster.

Nesse lab você irá criar uma chave de encriptação e uma [configuração de encriptação](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#understanding-the-encryption-at-rest-configuration) adequada para encriptar segredos do Kubernetes. 

## A chave de Encriptação

Crie uma chave de encriptação:

```
CHAVE_ENCRIPTACAO=$(head -c 32 /dev/urandom | base64)
```

## O Arquivo de Configuração de Encriptação

Crie o arquivo de encriptação `encryption-config.yaml`:

```
cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${CHAVE_ENCRIPTACAO}
      - identity: {}
EOF
```

Copie o arquivo de encriptação `encryption-config.yaml` para cada instância de controladora:

```
for instance in controller-0 controller-1 controller-2; do
  gcloud compute scp encryption-config.yaml ${instance}:~/
done
```

Próximo: [Subindo o Cluster etcd](07-subindo-etcd.md)
