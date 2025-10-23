   # Projeto-CI-CD-com-GitHub-Actions

> A documentação a seguir, descreve o passo a passo de como automatizar o ciclo completo de desenvolvimento, build, deploy e 
execução de uma aplicação FastAPI, usando o GitHub Actions e outros serviços.


## Sumário

- [Ferramentas utilizadas](#ferramentas-utilizadas)
- [Código da aplicação FastAPI](#código-da-aplicação-fastapi)
- [Etapa 1: Criar aplicação FastAPI](#etapa-1-criar-aplicação-fastapi)
- [Etapa 2: Criar o GitHub Actions (CI/CD)](#etapa-2--criar-o-github-actions-cicd)
- [Etapa 3: Repositório Git com os manifests do ArgoCD](#etapa-3--repositório-git-com-os-manifests-do-argocd)
- [Etapa 4: Criar o App no ArgoCD](#etapa-4--criar-o-app-no-argocd)
- [Alterando a mensagem da aplicação](#alterando-a-mensagem-dentro-do-código-de-python)
- [Conclusão](#conclusão)



### Ferramentas utilizadas 
* Python 3 e Docker
* Minikube, Kind ou Rancher
* ArgoCD instalado no Cluster local
* Git instalado




## Código da aplicação FastAPI

O repositório com o código-fonte da aplicação, Dockerfile e workflow do GitHub Actions está disponível aqui:

[Acesse o repositório hello-app no GitHub](https://github.com/Roberta-git/hello-app)




## Etapa 1: Criar aplicação FastAPI

#### Passo 1

1. Crie um repositório público no GitHub como o nome `hello-app`, sem adicionar README.
2. No terminal execute:
```
git clone https://github.com/seu-usuario/hello-app.git
cd hello-app
```

#### Passo 2

1. Dentro da pasta `hello-app`, no VS Code, crie um ambiente e instale o FastAPI e o Uvicorn para a aplicação.

```
python3 -m venv venv
source venv/bin/activate  # para Linux ou Mac
venv\Scripts\activate     # para Windows
pip install fastapi uvicorn
```

2. Crie o arquivo `requirements.txt` para registrar as dependências dos pacotes de Python.
```
pip freeze > requirements.txt
```

#### Passo 3

1. Crie um arquivo `main.py` com o conteúdo:
```
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}
```

2. Teste localmente, acessando no terminal:
```
uvicorn main:app --host 0.0.0.0 --port 8080
```

 * Abra o navegador  http://localhost:8080 e verifique se aparece a mensagem `{"message": "Hello World"}`.

<img src="https://github.com/user-attachments/assets/2c623e10-5b2f-4f73-8e25-2f97128c3fde"  alt="" width="700"/>
</p>



  * Teste no terminal via curl.
  ```
  curl http://localhost:8080
  ```

  <img src="https://github.com/user-attachments/assets/ba784636-27d7-4abe-8589-08792e4ce9f8"  alt="" width="700"/>
</p>

#### Passo 4

1. Para conteinerizar a aplicação, será preciso criar uma imagem Docker. Para isso, crie um arquivo chamado `Dockerfile` e inclua nele:
```
# Imagem base
FROM python:3.11-slim

# Diretório de trabalho
WORKDIR /app

# Copia os arquivos necessários
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copia o código da aplicação
COPY . .

# Define o comando de execução
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8080"]
```

2. Crie a imagem Docker, no terminal, através do comando:
```
docker build -t hello-app:latest .
```
  * Verifique no Docker Hub se a imagem foi construída e enviada, para isso, acesse  https://hub.docker.com/, entre na sua conta, cleque em repositórios e veja o seu repositório respectivo a imagem.

 <img src="https://github.com/user-attachments/assets/c4dd3b40-f236-444b-982e-f8aecc0572b3"  alt="" width="700"/>
</p>


  
3. Teste a  aplicação localmente.
```
docker run -p 8080:8080 hello-app:latest
```

4. Acesse http://localhost:8080, no navegador e verifique se apareece a mensagem `Hello Word`.

5. Se mensagem aparecer, volte ao terminal e faça o init e commit do arquivo.
```
git init
git add .
git commit -m "feat: primeira versão da API FastAPI e Dockerfile"

```

6. Conecte ao repostório remoto do GitHub por meio da URL para enviar os arquivos:
```
git remote add origin https://github.com/seu-usuario/hello-app.git
git branch -M main
git push -u origin main
```

Confira no GITHUb, se os arquivos pareceram.


## Etapa 2 – Criar o GitHub Actions (CI/CD) 

#### Passo 1

1. Crie o workflow do GitHub Actions.
2. No VSCode, na raiz de `hello-app`, gere a pasta  `.github ` e dentro dela, a pasta `workflows`; nela,  crie o arquivo `ci-cd.yml`, com o conteúdo:

```
name: CI/CD

on:
  push:
    branches:
      - main

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: --- exemplo
        uses: --- exemplo
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build da imagem
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: (coloque aqui o seu usuário do Docker Hub) seu-usuario-docker/hello-app:latest
```

3. No repositório do GitHub, clique em `Settings`, `Secrets and variables` e em `Actions`.

4. No botão `New repository secret `, adicione dois segredos:
 * Name: DOCKER_USERNAME. Value: seu nome no Docker Hub.
 * Name: DOCKER_PASSWORD. Value: o token (se usar o token, dê a permissão de ready, write e delete) ou senha do Docker Hub.


5. No terminal, rode o commit e push:
```
git add .github/workflows/ci-cd.yml
git commit -m "ci: adiciona pipeline GitHub Actions para build e push Docker Hub"
git push
```

6. Verifique, no repositório do GitHub, na aba `Actions`, se o workflow aparece verde, sinal de que tudo foi criado com sucesso.


<img src="https://github.com/user-attachments/assets/2c38bceb-d70f-4e8e-ba13-64b65c934b0d"  alt="" width="700"/>
</p>



## Etapa 3 – Repositório Git com os manifests do ArgoCD 

#### Passo 1

1. No VSCode, crie uma pasta chamada `deploy` e dentro dela, crie o arquivo `deployment.yaml`, com o conteúdo:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-app
  template:
    metadata:
      labels:
        app: hello-app
    spec:
      containers:
      - name: hello-app
        image: iasss/hello-app:latest
        ports:
        - containerPort: 8080
```

E o arquivo `service.yaml`, com o conteúdo:
```
apiVersion: v1
kind: Service
metadata:
  name: hello-app-service
  namespace: default
spec:
  selector:
    app: hello-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP
```

#### Passo 2

1. No terminal, suba os manifestos para o GitHub.
```
git add deploy/deployment.yaml deploy/service.yaml
git commit -m "infra: adiciona manifestos Kubernetes"
git push
```


## Etapa 4 – Criar o App no ArgoCD

#### Passo 1

1. No terminal, inicie o Minikube.
```
minikube start
```

2. Verifique se os pods então em "Running".
```
minikube status
kubectl get pods -A
```

<img src="https://github.com/user-attachments/assets/edba13e3-089b-4656-892e-f7613889ff3f"  alt="" width="700"/>
</p>



#### Passo 2

1. Instale o ArgoCD no Minekube, caso ainda não tenha.
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

2. Deixe o serviço na porta local do terminal aberto.
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```


3. Acesse o ArgoCD pelo navegador https://localhost:8080.

4. Faça o login, usando como usuário `admin` e senha.
  * Caso não saiba a senha, abra outra janela e use o comando `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | % { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }`


<img src="https://github.com/user-attachments/assets/1e54fe17-c0ab-4dd9-ae42-d37612e3c9be"  alt="" width="700"/>
</p>


#### Passo 3

1. Após entrar no ArgoCD, clique em "New App" e preencha os campos:

* Application Name: hello-app
* Project: default
* Sync Policy: "Manual" 
* Repository URL: link HTTPS do repositório que você criou no GitHub
* Revision: main
* Path: k8s
* Cluster URL: https://kubernetes.default.svc
* Namespace: default


<img src="https://github.com/user-attachments/assets/1709afe2-8d43-473b-b523-304835b0f151"  alt="" width="700"/>
</p>


2. Clique em `Create`.

3. Depois da aplicação criada, clique em `Sync`, para sincronizar os manifestos.

4. Espera-se que ao sincronizar,  a aplicação apareça com o status de saudável e com a cor verde.

5. Ao aparecer saudável, a aplicação terá sido criada com sucesso.


<img src="https://github.com/user-attachments/assets/9238c2c4-4860-4680-bbb4-29c7120b5760"  alt="" width="700"/>
</p>



## Alterando a mensagem dentro do código de python

1. Para alterar a mansagem do `main.py`, basta ir até o arquivo criado no VS Code e alterar a mensagem para qualquer outra. No meu caso, foi alterado de "Hello world" para "Olá, instrutores!".

2. Após a alteração, abra o terminal e faça add, commit e push do arquivo modificado.

```
git add main.py
git commit -m "feat: altera mensagem da API para Olá, instrutores!"
git push
```

3. Abra o navegador http://localhost:8081 e verifique se a mensagem realmente foi alterada.


<img src="https://github.com/user-attachments/assets/7ab62194-ce9f-4894-988a-5bceeb5d2467"  alt="" width="700"/>
</p>


  * Teste no terminal, via curl.
  ```
  curl http://localhost:8081
  ```

  <img src="https://github.com/user-attachments/assets/129602e4-d63b-4eaf-bad9-43bccd2fdd18"  alt="" width="700"/>
</p>

  
   


## Conclusão

Com a relização do projeto, foi possível aprimorar os conceitos a cerca de Kubernetes e assuntos relacionados, além de aprender sobre automatização de pipeline com GitHub Actions.
