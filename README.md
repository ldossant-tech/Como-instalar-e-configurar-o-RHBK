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

Um banco de dados deve estar disponível e acessível a partir do namespace do cluster onde a versão do Keycloak da Red Hat está instalada. Para obter uma lista dos bancos de dados compatíveis, consulte [Configurando o banco de dados](https://docs.redhat.com/en/documentation/red_hat_build_of_keycloak/24.0/html-single/server_guide/#db-) . A versão do Keycloak da Red Hat não gerencia o banco de dados e você precisa provisioná-lo por conta própria. 

Para fins de desenvolvimento, você pode usar uma instalação efêmera do PostgreSQL em formato de pod. Para provisioná-la, siga a abordagem abaixo:

```jsx
➜  rhbk cat postgres.yaml 
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgresql-db
spec:
  serviceName: postgresql-db-service
  selector:
    matchLabels:
      app: postgresql-db
  replicas: 1
  template:
    metadata:
      labels:
        app: postgresql-db
    spec:
      containers:
        - name: postgresql-db
          image: postgres:15
          volumeMounts:
            - mountPath: /data
              name: cache-volume
          env:
            - name: POSTGRES_USER
              value: testuser
            - name: POSTGRES_PASSWORD
              value: testpassword
            - name: PGDATA
              value: /data/pgdata
            - name: POSTGRES_DB
              value: keycloak
      volumes:
        - name: cache-volume
          emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: postgres-db
spec:
  selector:
    app: postgresql-db
  type: LoadBalancer
  ports:
  - port: 5432
    targetPort: 5432
```

### Implantação do CR Red Hat do Keycloak

Para implantar a versão do Keycloak para Red Hat, você cria um Recurso Personalizado (CR) com base na Definição de Recurso Personalizado (CRD) do Keycloak.

Considere armazenar as credenciais do banco de dados em um segredo separado. Digite os seguintes comandos:

```jsx
oc create secret generic keycloak-db-secret \
  --from-literal=username=testuser \
  --from-literal=password=testpassword
```

Você pode personalizar diversos campos usando o CRD do Keycloak. Para uma implantação básica, você pode seguir a seguinte abordagem:

```jsx
apiVersion: k8s.keycloak.org/v2alpha1
kind: Keycloak
metadata:
  name: keycloak
spec:
  instances: 1

  http:
    httpEnabled: true

  db:
    vendor: postgres
    host: postgres-db
    database: keycloak
    usernameSecret:
      name: keycloak-db-secret
      key: username
    passwordSecret:
      name: keycloak-db-secret
      key: password

  hostname:
    hostname: keycloak-service-kafka.apps.ldossant.vmware.tamlab.rdu2.redhat.com

  proxy:
    headers: xforwarded
```
