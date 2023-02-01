# Alô, Kubernetes!

Esse repositório é pra dar uma sacada em como o kubernetes interage com a api de forma declarativa (comandos).

Os arquivos criados estarão em _yaml_ por serem mais simples, mas também podem ser escritos em _json_, por exemplo.

O comando utilizado para _aplicar_ o conte trazidos no arquivo de criação do pod é:

```
kubectl apply -f ./primeiro-pod.yaml
``` 