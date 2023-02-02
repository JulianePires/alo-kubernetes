# Alô, Kubernetes!

Esse repositório é pra dar uma sacada em como o kubernetes interage com a api de forma declarativa (comandos).

Os arquivos criados estarão em *yaml* por serem mais simples, mas também podem ser escritos em *json*, por exemplo.

O projeto a seguir foi feito em acompanhamento ao curso de Kubernetes na Alura.

O comando utilizado para *aplicar* as informações trazidas no arquivo de criação do pod é:

```
kubectl apply -f .\portal-noticias.yaml
``` 

Para remoção do pod através do arquivo, utilizamos o comando:

```
kubectl delete -f .\portal-noticias.yaml
```

Para execução do pod criado (-it para modo iterativo, *bash* como comando a ser executado):
```
kubectl exec -it portal-noticiasv -- bash
```

## Acesso com Services

Para que haja comunicação efetiva entre os pods, de maneira que não hajam problemas de comunicação mesmo que os pods sejam encerrados e alterem o IP ao serem distribuidos, utilizamos um recurso chamado **Service**.

Services são abstrações que são utlizadas para expor aplicações executando em um ou mais pods.

Existem 3 tipos de services:

- ClusterIP -> Realiza comunicação entre diferentes pods dentro de um container através de um IP fixo
- NodePort -> Permite a comunicação do pod com o mundo externo
- LoadBalancer

### Construindo pontes com o ClusterIP

O ClusterIP nada mais é do que um conector, ele funciona como uma ponte de um pod para ser acessado pelos demais pods dentro de um cluster. O que ele faz é fornecer um IP fixo com mapeamento de porta para que, dessa maneira, outros componentes consigam acessar esse pod apontado, mesmo que ele tenha que ser substituido e, consequentemente, tenha seu IP alterado.

O arquivo *svc-pod-2.yaml* traz configurações de um ClusterIP utilizado para comunicar o pod-2 com os demais recursos do cluster.

Nele podemos observar que, além de definirmos o tipo nas especificações, temos também o seletor apontando para o pod-2 e a porta de destino 80.

Para acessarmos então o pod-2 com o pod-1, estando os pods inicializados, seguimos os passos:

1. Criação do service a partir do arquivo

```
kubectl apply -f .\svc-pod-2.yaml 
```
2. Execução do pod-1 em modo iterativo

```
kubectl exec -it pod-1 -- bash
```
3. Acesso ao IP do service, na porta indicada, através do terminal do pod-1

```
curl 10.106.130.115:80
```

### Abrindo horizontes com o NodePort

O NodePort aqui funciona como uma porta para o exterior do cluster, podendo então acessar com qualquer máquina externamente através do kubectl. Ele também exerce uma função de *ClusterIP* dentro do cluster.

O arquivo *svc-pod-1.yaml* traz as configurações necessárias para a configuração de um NodePort que irá realizar um *bind* no pod-1. Nele, é especificado o tipo de serviço como NodePort, porém, você pode observar aque as configurações são bem similares ao *ClusterIP*.

Atuando como *ClusterIP*, podemos acessar então o conteúdo do pod-1 através do portal-noticias, que não está vinculado a nenhum serviço. Assim, temos:

```
kubectl exec -it portal-noticias -- bash
```
E então, através do IP do NodePort, na porta mapeada, com o curl:

```
curl 10.104.108.232:80
```
Para que o serviço seja acessado externamente (do nosso navegador, por exemplo), devemos definir uma porta para o nó, através da propriedade nodePort, no arquivo de configurações.

Assim, por padrão o ip designado pelo docker desktop no Windows, para que acessemos o cluster na nossa máquina, é o localhost. Acessamos então através da porta definida (3030):

> https://localhost:3030
