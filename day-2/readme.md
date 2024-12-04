## Applications

Tal como um pod é a menor forma de conteúdo no Kubernetes, dentro do ArgoCD o Application é a menor forma.Nessa aula, abordamos como funcionam os Applications de dentro pra fora e de cabo a rabo. A melhor parte é que é só isso mesmo. Os Applications são apenas links, que indicam para o Argo Server onde está o Helm ou Kustomization e indicam onde eles devem ser renderizados.


Os Applications podem receber overrides ou value files de fontes locais (o mesmo repositório) ou de fontes separadas (fazer deploy de um repositório público, como um Grafana; mas usar um values local). Para isso, usamos dois sources - ou seja, estamos indicando para o Argo que nossa aplicação possui 2 repositórios git. Funciona assim:
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: grafana-app
  namespace: argocd
spec:
  project: default
  sources:
  - repoURL: 'https://grafana.github.io/helm-charts'
    chart: grafana
    targetRevision: 7.0.19
    helm:
      valueFiles: 
      - $values/grafana/values.yaml
  - repoURL: 'https://github.com/bernardolsp/descomplicando-gitops-no-kubernetes-argocd.git'
    targetRevision: feat/day2
    ref: values
  destination:
    namespace: monitoring
    server: 'https://kubernetes.default.svc'
  syncPolicy:
    automated:
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```


Notem que inserimos o valueFiles como uma variável a partir do ref do segundo repoURL. Isso é possível e muito prático!Essa sintaxe nem sempre foi permitida. Antes dela, usávamos o conceito de child charts, onde criamos Charts que utilizam dependências externas, tipo assim:

```
apiVersion: v2
name: wordpress
description: A Helm chart for Kubernetes
dependencies:
- name: wordpress
  version: 9.0.3
  repository: https://charts.helm.sh/stable
```

E o values utilizaria wordpress: como primeira linha, para que todo o conteúdo que estamos modificando faça override do chart dependência, dessa forma:
```
wordpress:
  wordpressPassword: foo
  mariadb:
    db:
      password: bar
    rootUser:
      password: baz
```


Na aula de hoje nós comentamos sobre como expandir o conceito de Applications utilizando o App of Apps - o nome indica ser uma gambiarra, mas na verdade é um pattern bastante utilizado no Argo - até tem sua própria descrição na documentação ( https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/). O app of Apps funciona de forma bem simples:


você possui um Application
o Application olha para um diretório, em um repositório
esse diretório possui um helm chart
(até aqui, exatamente igual a um app normal)
esse helm chart é composto por outros Applications (ao invés de Deployments, Services, Configmaps...)
esses Applications possuem seus próprios links para seus repositórios e diretórios
