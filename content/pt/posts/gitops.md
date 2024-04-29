+++
title = "A tranquilidade do GitOps"
date = "2024-03-03T11:37:01-03:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["devops", "gitops", "kubernetes"]
+++

Recentemente, migramos todos os nossos serviços de um cluster DC/OS Marathon para um cluster Openshift. Isso representou um grande passo para nossa organização, pois tivemos que nos adaptar a uma plataforma completamente nova. Isso incluiu encontrar maneiras mais modernas e eficientes de arquitetar nossos pipelines de CI/CD. Enquanto pesquisava os muitos mecanismos de entrega contínua que podem ser implementados, me deparei com os pipelines baseados em pull e o paradigma GitOps, e foi amor à primeira vista.

Aqui, vou compartilhar minha experiência implementando o paradigma GitOps e usando o ArgoCD como a ferramenta de escolha para entrega contínua.

## GitOps, comparado à abordagem tradicional

Primeiro, é importante ver a diferença e as principais vantagens do GitOps em comparação com a abordagem tradicional, os pipelines baseados em push. Pipelines baseados em push, em termos simples, são pipelines que empurram para sua implantação na etapa de entrega do seu CI/CD. Como exemplo, um pipeline tradicional que implanta no Kubernetes pode terminar com o comando:

```bash

kubectl apply -f deployment.yaml
```
Em contraste, um pipeline baseado em pull depende de uma ferramenta que consulta um determinado recurso para o estado desejado e o aplica ao seu cluster. Em nosso caso, a ferramenta de escolha é o ArgoCD, e o recurso consultado é um repositório git. A prática de utilizar um repositório git como a única fonte de verdade para a configuração de implantação é o que chamamos de GitOps. Isso garante uma propriedade muito importante: cada configuração de implantação de cada aplicativo está sujeita a controle de versão. Isso nos permite acompanhar, gerenciar facilmente e, se necessário, retornar a estados anteriores.

## Adoção e implementação

Depois de configurar um pipeline de conceito de ArgoCD, apresentei-o aos meus colegas. No início, a etapa de entrega foi contra-intuitiva, no entanto, meus colegas rapidamente perceberam o valor de controlar as versões de nossas implantações e concordamos coletivamente que essa abordagem seria a escolhida. Este foi um passo importante, considerando que o DevOps é mais sobre cultura do que qualquer outra coisa, é importante ter toda a equipe na mesma página. Uma vez que nossa equipe decidiu ir com o ArgoCD, tivemos que decidir como realmente implementar nossos novos pipelines. Aqui está como fizemos:
Monorepositório

Nossa única fonte de verdade é um monorepositório com arquivos para cada aplicativo. Para cada branch de cada aplicativo, temos um helm chart para sua implantação correspondente, seguindo este formato:

```bash

/caminho/para/aplicacao
├── nome-da-branch
│   ├── Chart.yaml
│   ├── templates
│   └── values.yaml
```
Isso nos permite encontrar facilmente cada aplicativo, considerando que o caminho para ele dentro do monorepositório é igual ao caminho do aplicativo em nosso servidor git. No início, eu estava preocupado que o monorepositório ficasse muito grande e acabasse desacelerando nossos pipelines, mas acabou por não ser um problema.

## Pipelines

As etapas de "entrega" de nossos pipelines ficaram muito mais simples, e acabamos com algo semelhante a isso:

```bash

git clone monorepositório
yq -i '.image = "nome-da-nova-imagem"' values.yaml # atualizar o nome da imagem em values.yaml
git commit . -m "mensagem de commit"
git push

```
Como o ArgoCD é responsável apenas por monitorar um repositório git e implantar alterações, pudemos manter nossas etapas de CI exatamente as mesmas.
## Os resultados

Migrar para uma plataforma completamente nova e processo de implantação foi bastante trabalho, mas o processo foi relativamente simples. Os benefícios foram exatamente como esperado: não mais estresse com arquivos de configuração misteriosos em locais misteriosos; ou alterações perdidas e não registradas! Há uma tranquilidade no DevOps que apenas implantações controladas por versão e entregas simples podem trazer!

## Referências e leituras relacionadas
[Security benefits of pull-based pipelines, por alxk](https://alex.kaskaso.li/post/pull-based-pipelines)
