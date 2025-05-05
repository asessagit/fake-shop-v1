# Deployment do PostgreSQL
apiVersion: apps/v1  # Especifica a versão da API do Kubernetes para criar um Deployment.
kind: Deployment  # Define o tipo de recurso como Deployment para gerenciar pods.
metadata:
  name: postgre  # Nome do Deployment (pode ser usado para identificá-lo no cluster).
spec:
  selector:
    matchLabels:
      app: postgre  # Seleciona pods que possuem o rótulo "app: postgre".
  template:
    metadata:
      labels:
        app: postgre  # Define o rótulo dos pods criados por este Deployment.
    spec:
      containers:
        - name: postgre  # Nome do contêiner que será executado.
          image: postgres:13.16  # Imagem Docker do PostgreSQL na versão 13.16.
          ports:
            - containerPort: 5432  # Porta exposta dentro do contêiner (porta padrão do PostgreSQL).
          env:  # Variáveis de ambiente para configurar o banco de dados.
            - name: POSTGRES_DB
              value: fakeshop  # Nome do banco de dados que será criado.
            - name: POSTGRES_USER
              value: fakeshop  # Nome do usuário de acesso ao banco.
            - name: POSTGRES_PASSWORD
              value: Pg1234  # Senha para o usuário do banco de dados.

---
# Service para expor o PostgreSQL dentro do cluster
apiVersion: v1  # Define a API usada para criar o recurso (Service).
kind: Service  # Tipo do recurso é um Service, que conecta os pods à rede.
metadata:
  name: postgre  # Nome do Service (associado ao banco de dados PostgreSQL).
spec:
  selector:
    app: postgre  # Seleciona os pods com o rótulo "app: postgre".
  ports:
    - port: 5432  # Porta acessada pelo Service (dentro do cluster).
      targetPort: 5432  # Porta no pod para onde o tráfego será redirecionado.

---
# Deployment da aplicação web
apiVersion: apps/v1  # Especifica a versão da API para criar um Deployment.
kind: Deployment  # Define que este recurso é um Deployment.
metadata:
  name: fakeshop  # Nome do Deployment da aplicação web.
spec:
  selector:
    matchLabels:
      app: fakeshop  # Seleciona os pods com o rótulo "app: fakeshop".
  template:
    metadata:
      labels:
        app: fakeshop  # Rótulo usado para identificar os pods da aplicação.
    spec:
      containers:
        - name: fakeshop  # Nome do contêiner da aplicação.
          image: alexsessa/fake-shop:v1  # Imagem Docker da aplicação, na versão v1.
          ports:
            - containerPort: 5000  # Porta onde a aplicação Flask está escutando.
          env:  # Variáveis de ambiente para configurar a aplicação.
            - name: DB_HOST
              value: postgre  # Nome do serviço do PostgreSQL para conexão.
            - name: DB_USER
              value: fakeshop  # Nome do usuário do banco (igual ao configurado no PostgreSQL).
            - name: DB_PASSWORD
              value: Pg1234  # Senha do banco de dados.
            - name: DB_NAME
              value: fakeshop  # Nome do banco de dados.
            - name: FLASK_APP
              value: index.py  # Arquivo principal da aplicação Flask.

---
# Service para expor a aplicação web
apiVersion: v1  # Define a API usada para criar o recurso (Service).
kind: Service  # Tipo do recurso é um Service.
metadata:
  name: fakeshop  # Nome do Service para a aplicação web.
spec:
  selector:
    app: fakeshop  # Associa o Service aos pods com o rótulo "app: fakeshop".
  ports:
    - port: 80  # Porta pública do Service para acessos externos.
      targetPort: 5000  # Porta do contêiner (onde a aplicação Flask está rodando).
  type: LoadBalancer  # Expõe o Service na internet usando um balanceador de carga.
