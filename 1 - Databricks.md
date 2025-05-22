Databricks nada mais é do que uma plataforma de processamento de dados na nuvem, ele não é um componente de cloud, nem uma cloud em si, nada mais que uma plataforma.

Nele, pode ser utilizado Spark, Python e SQL Para para trabalhar com Big Data de forma segura e otimizada. Sendo muito mais que apenas processar, ele automatiza ETLs, governa o acesso aos dados, e permite uma colaboração entre as equipes.

Eles se auto definem como uma plataforma Lake House! Com ele é possível trabalhar com dados, estruturados, semi-estruturados, não estruturados e streaming. Atende a demandas de engenharia de dados, BI e analise com SQL.

- [[#Como provisionar o Databricks?|Como provisionar o Databricks?]]
- [[#Como conectar o Databricks a recursos da nuvem?|Como conectar o Databricks a recursos da nuvem?]]
- [[#Linkar GIT, Bitbucket com Databricks|Linkar GIT, Bitbucket com Databricks]]
- [[#Workspace|Workspace]]
## Como provisionar o Databricks?

Para provisionar o Databricks especificamente na Azure, basta buscar pelo serviço dentro do portal. Se estiver algo já provisionado será listado, caso não, criar um novo.

![[Pasted image 20250111183305.png]]
![[Pasted image 20250111183317.png]]
![[Pasted image 20250111183924.png]]

No primeiro acesso, clicar em **LAUNCH WORKSPACE**:

![[Pasted image 20250111184623.png]]

Quando você criamos um **Databricks Workspace** na Azure, ele pode criar automaticamente um grupo de recursos separado porque o Azure Databricks utiliza um **grupo de recursos gerenciado** para gerenciar os recursos internos necessários para o funcionamento do workspace. Esse comportamento é padrão e projetado para isolar e organizar os recursos que o Databricks utiliza. Ao criar um cluster por exemplo, ele cria uma VM e outros serviços relacionados dentro do resource group gerenciado.


## Como conectar o Databricks a recursos da nuvem?

Necessitamos fazer as coisas "conversarem" entre si após as configurações. Para tudo que vamos usar integrado com outro recurso precisamos unir, configurar, e no Databricks não é diferente. Por exemplo, para usar ele e fazer uso das secrets geradas no key vault, precisamos integrar. Também precisamos fazer uma conexão onde o Databricks vai escrever o dado, se for na camada trusted por exemplo, preciso ter uma conectividade com o Data Lake no notebook (isso para standard, salve engano).

Para não ter a senha exposta em cada notebook vamos usar a Key Vault. Porém, a forma de fazer é diferente do ADF por exemplo, onde vamos no KV e criamos um access police para o ADF usar o KV.

Nesse caso vamos criar usando um **ESCOPO DE SEGREGO**, que é uma configuração onde vamos ter as secrets do KV. Como criar o escopo?

1 - [Gerenciamento de segredos – Azure Databricks | Microsoft Learn](https://learn.microsoft.com/pt-br/azure/databricks/security/secrets/)
2 - Dentro do link temos um tutorial de como fazer isso, há um link onde substituímos parte dele pela URL do workspace, que é encontrado lá na Azure Databricks.

![[Pasted image 20250111185851.png]]

3 - Após isso uma tela de criação de escopo será carregada, e devemos preencher com as informações da nessa Key Vault, elas são encontradas na aba **PROPERTIES**, DNS Name é o _Vault URI_ e Resource ID o _Resource ID_. 

![[Pasted image 20250111190043.png]]

Como o Databricks é standard, não podemos criar no formato creator, onde diz que apenas eu que criei o recurso posso mexer no escopo, isso a nível Databricks. Obrigatoriamente por ser standard, devo permitir que todo mundo consiga fazer isso (11/01/2025).

![[Pasted image 20250111190504.png]]

Após isso, podemos ver na key vault que o serviço do Databricks aparece:

![[Pasted image 20250111190746.png]]

Para acessar uma secret no notebook podemos usar o código abaixo, e por mais que tente printar por exemplo, ele não deixa ver a secret:
```python
key_datalake = dbutils.secrets.get(scope="scope_bricks", key="secret-db-azure")
```
![[Pasted image 20250111191841.png]]

---

A plataforma foi criada pelos mesmos criados do Spark, Delta Lake e MlFlow. Abaixo será apresentado alguns componentes do Databricks:
## Workspace

O workspace nada mais é o "ambiente de trabalho" onde vão estar os códigos e os repositórios clonados do GIT. Aqui conseguimos ver os famosos notebooks e os repos, também é possível ver o espaço de trabalho de outras pessoas, claro com as permissões devidas.

## [[Compute (Cluster policies, etc.)]]

Parte onde se gerencia os clusters, podemos criar vários clusters diferentes, com diferentes tamanhos, armazenamento, etc. E há diversos tipos como clusters individuais, trabalho, para carga grande em paralelo. Sem o cluster não é possível fazer nada no Databricks.

## [[Delta Tables]]

## [[Unity Catalog and metastore]]




