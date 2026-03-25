# Como instalar e configurar o RHBK
- [Como instalar e configurar o RHBK](#como-instalar-e-configurar-o-rhbk)
  - [Banco de dados](#banco-de-dados)
  - [Implantação do CR Red Hat do Keycloak](#implantação-do-cr-red-hat-do-keycloak)

1. Abra o console web da plataforma OpenShift Container Platform.
2. Na coluna da esquerda, clique em Início , Operadores , OperatorHub .
3. Procure por "Keycloak" na caixa de pesquisa.
4. Selecione a operadora na lista de resultados.
5. Siga as instruções na tela.

Após a instalação e execução da versão do Keycloak Operator da Red Hat no namespace do cluster, você poderá configurar os demais pré-requisitos de implantação.

### Banco de dados

Para fins de desenvolvimento, você pode usar uma instalação efêmera do PostgreSQL em formato de pod. Para provisioná-la, vamos aplicar os yamls localizado no diretório infra/01-rhbk/. Vamos prosseguir via terminal usando a oc CLI.

```jsx
cd infra/01-rhbk/
```
Faça login no OpenShift usando o comando:

```jsx
oc login -u <USER> -p <PASSWORD> <HOST>:6443
```

entre no namespace do projeto:

```jsx
oc project <PROJECT>
```

Agora precisamos ter um banco de dados para o RHBK.

Para fins de desenvolvimento, você pode usar uma instalação efêmera do PostgreSQL em formato de pod.

aplicar o yaml infra/01-rhbk/postgresql.yaml:

```jsx
oc apply -f infra/01-rhbk/postgresql.yaml -n <PROJECT>
```

Verifique se o pod foram criados em pods dentro de workload:
<IMAGEM POSTGRESQL>

### Implantação do CR Red Hat do Keycloak

Para implantar a versão do Keycloak para Red Hat, você cria um Recurso Personalizado (CR) com base na Definição de Recurso Personalizado (CRD) do Keycloak.

Considere armazenar as credenciais do banco de dados em um segredo separado. Digite os seguintes comandos:

Execute dentro do CLI:

```jsx
oc create secret generic keycloak-db-secret \
  --from-literal=username=testuser \
  --from-literal=password=testpassword
```

Você pode personalizar diversos campos usando o CRD do Keycloak. Para uma implantação básica, vamos utilizar o recurso dentro de 01-infra/keycloak.yaml

Execute dentro do CLI:

```jsx
oc apply -f keycloak.yaml -n <PROJECT>
```
