# Alô, Kubernetes!

Esse repositório é pra dar uma sacada em como o kubernetes interage com a api de forma declarativa (comandos).

Os arquivos criados estarão em _yaml_ por serem mais simples, mas também podem ser escritos em _json_, por exemplo.

O projeto a seguir foi feito em acompanhamento ao curso de Kubernetes na Alura.

O comando utilizado para _aplicar_ as informações trazidas no arquivo de criação do pod é:

```
kubectl apply -f .\portal-noticias.yaml
``` 

Para remoção do pod através do arquivo, utilizamos o comando:

```
kubectl delete -f .\portal-noticias.yaml
```

Para execução do pod criado (-it para modo iterativo, _bash_ como comando a ser executado):
```
kubectl exec -it portal-noticiasv -- bash
```

### Acesso com Services

Para que haja comunicação efetiva entre os pods, de maneira que não hajam problemas de comunicação mesmo que os pods sejam encerrados e alterem o IP ao serem distribuidos, utilizamos um recurso chamado **Service**.

Services são abstrações que são utlizadas para expor aplicações executando em um ou mais pods.

Existem 3 tipos de services:

- ClusterIP -> Realiza comunicação entre diferentes pods dentro de um container através de um IP fixo
- NodePort
- LoadBalancer

O arquivo _svc-pod-2.yaml_ traz configurações de um ClusterIP utilizado para comunicar o pod-2 com os demais recursos do cluster.

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