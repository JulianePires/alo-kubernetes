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

> https://localhost:30000

### Distribuindo cargas com *LoadBalancer*

É fácil chegar ao conceito do *Load Balancer* quando já entendemos *ClusterIP* e *NodePort*. O Load Balancer nada mais é do que um NodePort (permite acesso externo) que automaticamente se integra ao LoadBalancer do provedor de Cloud.

Isso facilita MUITO a conexão entre o nosso cluster e qualquer que seja a nuvem que queremos dar acesso a nossa estrutura.

Para configurarmos nosso serviço, criamos então um outro arquivo de configuração denominado *svc-pod-1-loadbalancer.yaml*, onde haverão propriedades que permitirão a criação de um serviço do tipo LoadBalancer vinculado ao *pod-1*, respondendo na porta 30000.

A mesma configuração deve ser aplicada no serviço de cloud. Acessando o terminal e copiando toda a informação num arquivo chamado *lb.yaml*:

```
cat > lb.yaml
```

E ativamos através do comando:

```
kubectl apply -f lb.yaml 
```
E... Mágica! Sem nenhuma configuração adicional, somos capazes de acessar nosso pod através de um link externo gerado pelo próprio serviço de nuvem.

### Definindo variáveis de ambiente (*env*)

Para que as notícias sejam cadastradas e trazidas do banco de dados, precisamos configurar um pod com um container do banco (mysql) e um *ClusterIP* para que esse banco possa ser acessado pelos demais pods dentro do cluster.

Só tem um problema com esse pod, o mysql necessita que sejam configuradas algumas variáveis de ambiente para que tenhamos acesso ao seu sistema.

Portanto, podemos definir variáveis de ambiente dentro do arquivo de identificação do próprio pod, dentro de uma propriedade chamada *env* nas especificações do container.

### Discriminando informações com o ConfigMap

No momento atual, temos as configurações de cada pod junto com a declaração de variáveis de ambiente, deixando o código acoplado, não sendo boa prática. Precisamos então desacoplar esse código distribuindo responsabilidades. Para tal, fazemos o uso de *ConfigMaps*.

Além do desacoplamento, os ConfigMaps têm a grande vantagem da reutilização de configurações entre diversos pods.

Para utilizar o ConfigMap dentro do nosso pod de banco de dados, definimos as variáveis de ambiente por referência ao arquivo db-configmap.

Também foi configurado um ConfigMap para o pod de sistem e um para o pod do portal, definindo as variáveis de ambiente necessárias para o funcionamento de cada um.

Porém, ainda tinhamos o problema de persistência desses pods, caso um fosse deletado por algum motivo específico, não seria mantido, interrompendo assim parte do sistema.

### Garantindo vitalidade com ReplicaSets

Para resolver isso, utilizamos os ReplicaSets, os ReplicaSets são encapsulamentos de pods, sendo capazes de criar e gerenciar um ou mais pods ao mesmo tempo.

Assim, caso algum pod venha a falhar, os ReplicaSets são capazes de criar um novo e/ou gerenciar o balanceamento de carga entre os demais nele presentes.

### Versionando imagens e pods com Deployments

Os *Deployments* conseguem executar a mesmíssimma função dos ReplicaSets, só que, na verdade, os Deployments acabam encapsulando os ReplicaSets, pois possuem a vantagem do controle do versionamento das imagens e pods.

A partir de alguns comandos extras, é possível ter acesso ao histórico de mudanças realizadas nos pods, com o comando:

```
kubectl rollout history deployment .\nome-deployment.yaml
```

A saída desse comando traz o número da revisão e a causa da mudança.

Caso eu queira deixar essa causa (um exemplo seria a alteração de versão para latest) um pouco mais declarativa, podemos utilizar o comando:

```
kubectl annotate deployment .\nome-deployment.yaml kubernetes.io/change-cause="Definindo a imagem com versão latest"
```

E o mais legal de todos, que é simplesmente poder voltar para um estado de revisão anterior com o comando:

```
kubectl rollout undo deployment .\nome-deployment.yaml --to-revision=2
```

Com esse comando, voltaríamos para a configuração presente na revisão 2.

## Garantindo persistência

### Volumes

Quando estudamos Docker, aprendemos que conseguimos compartilhar dados entre containers através dos volumes. No kubernetes, o volume tem basicamente a mesma função (nesse caso compartilha informação entre os containers presentes no pod), com a diferença que o ciclo de vida desses volumes depende do pod e não dos containers dentro dele. Então, eles continuam sendo efêmeros, a partir do momento em que o pod deixa de existir, o volume também não faz mais sentido, mas se um container deixa de funcionar, ainda assim temos um volume, pois o pod só deixa de existir no momento em que todos os containers presentes falham.

### PersistentVolume e PersistentVolumeClaim

Os *PersistentVolume's* são volumes que possuem um ciclo de vida independente dos pods, eles geralmente são criados nos serviços de nuvem e podem possuir detalhes de implementação do aramazenamento.

Para acessar um PersistentVolume com um Pod, precisamos de um outro recurso chamado *PersistentVolumeClaim*, como o termo *claim* (requisitar) já diz, o que esse volume faz, é solicitar recursos como memória e CPU para os PersistentVolume's. Por ter esse intermédio, permite-se que os admninistradores possam realizar diversas configurações padrão dentro do PersistentVolume, sem que mais detalhes sobre a estrutura sejam dados na requisição. Eles podem requisitar tamanhos e modos de acesso específicos (montados ReadWriteOnce, ReadOnlyMany or ReadWriteMany).

### Storage Classes

O *Storage Class* possui uma propriedade importante que é ao realizarmos um *bind* do pod com o *pvc* através do Storage Class, somos capazes de criar discos e volumes dinamicamente.

## Mantendo o estado dos pods com Stateful Sets

O *Stateful Set* tem a função de manter o arquivo e propriedade dos pods ainda que os mesmos sejam deletados e criados novamente. Para isso, precisamos definir que esse pod esteja ligado a um *Persistent Volume Claim* para acessar o *Persistent Volume*.