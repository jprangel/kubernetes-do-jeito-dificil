# Limpeza

Nesse lab você irá deletar os recursos computacionais criados durante esse tutorial.

## Instâncias Computacionais

Delete o controlador e a instância computacional dos _workers_:

```
gcloud -q compute instances delete \
  controller-0 controller-1 controller-2 \
  worker-0 worker-1 worker-2
```

## Rede

Remova os recursos de rede do balanceador de carga externo:

```
gcloud -q compute forwarding-rules delete kubernetes-forwarding-rule \
  --region $(gcloud config get-value compute/region)
```

```
gcloud -q compute target-pools delete kubernetes-target-pool
```

Remova os endereço de IP estático `kubernetes-the-hard-way`:

```
gcloud -q compute addresses delete kubernetes-the-hard-way
```

Remova as regras de firewall `kubernetes-the-hard-way`:

```
gcloud -q compute firewall-rules delete \
  kubernetes-the-hard-way-allow-nginx-service \
  kubernetes-the-hard-way-allow-internal \
  kubernetes-the-hard-way-allow-external
```

Remova as rodas de rede de Pod:

```
gcloud -q compute routes delete \
  kubernetes-route-10-200-0-0-24 \
  kubernetes-route-10-200-1-0-24 \
  kubernetes-route-10-200-2-0-24
```

Remova a sub-rede `kubernetes`:

```
gcloud -q compute networks subnets delete kubernetes
```

Remova a rede VPC `kubernetes-the-hard-way`:

```
gcloud -q compute networks delete kubernetes-the-hard-way
```
