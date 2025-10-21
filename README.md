# Projeto-CI-CD-com-GitHub-Actions

> A documentação a seguir, descreve o passo a passo de como automatizar o ciclo completo de desenvolvimento, build, deploy e 
execução de uma aplicação FastAPI, usando o GitHub Actions e outros serviços.


### Ferramentas utilizadas 
* Python 3 e Docker
* Minikube, Kind ou Rancher
* ArgoCD instalado no Cluster local
* Git instalado



## Etapa 1: Criar aplicação FastAPI

### Passo 1

1. Crie um repositório público no GitHub como o nome `hello-app`, sem adicionar README.
2. No terminal execute:
```
git clone https://github.com/seu-usuario/hello-app.git
cd hello-app
```



