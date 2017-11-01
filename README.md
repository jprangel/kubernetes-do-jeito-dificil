ATENÇÃO: Essa é uma tradução livre, sem fins lucrativos, buscando apenas facilitar o acesso a material em Português sobre o Kubernetes diretamente de quem realmente entende do assunto.

# Kubernetes no Modo Difícil

Esse tutorial irá acompanhá-lo na configuração do Kubernetes no Modo Difícil. Esse guia não é para pessoas buscando um comando totalmente automatizado para colocar no ar um cluster de Kubernetes. Se você é um desses então confira [Google Container Engine](https://cloud.google.com/container-engine), ou então [Getting Started Guides](http://kubernetes.io/docs/getting-started-guides/) (em Inglês).

Kubernetes no Modo Difícil está otimizado para o aprendizado, significando tomar o caminho mais longo para garantir que você entenda cada tarefa necessária para colocar no ar um cluster de Kubernetes.

> Os resultados desse tutorial não devem ser tidos como aptos à produção, e podem ter suporte limitado da comunidade, mas não deixe que isso o impeça de aprender!

## Público Alvo

O público alvo para esse tutorial é alguem planejando suportar um cluster de Kubernetes em produção e quer entender como tudo se encaixa.

## Detalhes do Cluster

O Kubernetes no Modo Difícil irá guiá-lo para colocar no ar um cluster Kubernetes de alta disponibilidade com encriptação fim-a-fim entre componentes e autenticação RBAC.

* [Kubernetes](https://github.com/kubernetes/kubernetes) 1.8.0
* [cri-containerd Container Runtime](https://github.com/kubernetes-incubator/cri-containerd) 1.0.0-alpha.0
* [CNI Container Networking](https://github.com/containernetworking/cni) 0.6.0
* [etcd](https://github.com/coreos/etcd) 3.2.8

## Laboratórios

Esse tutorial parte do princípio que você tenha acesso ao [Google Cloud Platform](https://cloud.google.com). Ainda que o GCP é utilizado para requisitos básicos de infraestrutura, as lições aprendidas nesse tutorial podem ser aplicadas a outras plataformas.

* [Pré-requisitos](docs/01-pre-requisitos.md)
* [Instalando as Ferramentas Cliente](docs/02-ferramentas-cliente.md)
* [Provisionando Recursos Computacionais](docs/03-recursos-computacionais.md)
* _PENDENTE_[Provisionando a CA e Gerando Certificados TLS](docs/04-certificate-authority.md)
* _PENDENTE_[Gerando Arquivos de Configuração do Kubernetes para autenticação](docs/05-kubernetes-configuration-files.md)
* _PENDENTE_[Gerando a Configuração de Encriptação de Dados e a Chave](docs/06-data-encryption-keys.md)
* [Subindo o Cluster etcd](docs/07-subindo-etcd.md)
* [Subindo o Plano de Controle](docs/08-subindo-controladoras-kubernetes.md)
* _PENDENTE_[Subindo os Nós Worker do Kubernetes](docs/09-bootstrapping-kubernetes-workers.md)
* [Configurando o kubectl para Acesso Remoto](docs/10-configurando-kubectl.md)
* [Provisionando Rotas de Rede dos Pods](docs/11-rotas-de-rede-de-pod.md)
* [Implantando o Add-on de DNS do Cluster](docs/12-dns-addon.md)
* [Smoke Test (Teste de Fumaça)](docs/13-smoke-test.md)
* [Limpeza](docs/14-limpeza.md)
