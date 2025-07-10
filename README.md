# Projeto Kubernetes com GitOps e ArgoCD - Online Boutique 🚀

Este projeto demonstra a implantação de uma aplicação de microserviços (Online Boutique) em um cluster Kubernetes local (Minikube) utilizando o princípio de GitOps com a ferramenta ArgoCD.

---

## 📝 Entregas Esperadas

As seguintes entregas foram realizadas com sucesso neste projeto:

* ✅ **Criação de Repositório Git Público:** Configuração de um repositório Git público contendo a estrutura de manifests YAML para a aplicação.
* ✅ **Deploy do ArgoCD:** Implantação e configuração correta da ferramenta ArgoCD no cluster Kubernetes.
* ✅ **Criação do App no ArgoCD:** Configuração da aplicação no ArgoCD, apontando para o repositório Git.
* ✅ **Sincronização da Aplicação:** Sincronização bem-sucedida da aplicação, garantindo que todos os pods estejam rodando e saudáveis.
* ✅ **Acesso ao Frontend:** Acesso ao frontend da aplicação Online Boutique via `kubectl port-forward`.

---

## ⚙️ Pré-requisitos

Para replicar este projeto, você precisará ter os seguintes softwares instalados e configurados em seu ambiente:

* **Git:** Ferramenta de controle de versão.
* **Docker Desktop com WSL2:** Ambiente de contêineres Docker com o subsistema Linux para Windows.
* **Minikube:** Ferramenta para rodar um cluster Kubernetes localmente.
* **`kubectl`:** Ferramenta de linha de comando para interagir com clusters Kubernetes. (Geralmente vem com o Minikube ou Docker Desktop).
* **Conta GitHub:** Para hospedar o repositório do projeto.
* **Personal Access Token (PAT) do GitHub:** Com permissões de `repo` para o ArgoCD acessar o repositório.
* **Visual Studio Code (VS Code):** Editor de código (opcional, mas recomendado para o WSL).

---

## 🚀 Passo a Passo para Executar o Projeto

Passos para configurar e implantar a aplicação Online Boutique usando GitOps.

### **1. Configuração Inicial do Ambiente**

1.  **Inicie o Minikube:**
    Certifique-se de que o Docker Desktop e o WSL2 estão rodando. Inicie o Minikube:
    ```bash
    minikube start
    ```
    (Este comando também configurará o `kubectl` para usar o contexto do Minikube).
![minikube-start](https://github.com/user-attachments/assets/b53d2aa4-17f3-4aa3-af22-d7905eac3fac)

    **Clone o Repositório:**
    Abra seu terminal WSL e clone este repositório:
    ```bash
    git clone https://github.com/fabsakae/Projeto-k8s-GitOps.git
    ```
![gitclone](https://github.com/user-attachments/assets/22581c17-a501-4ccf-be95-94c88d075eac)
2.  **Etapa 1 – Fork e repositório GitHub:**
    Fork do repositório oficial:
        a. https://github.com/GoogleCloudPlatform/microservices-demo
![fork](https://github.com/user-attachments/assets/c7860645-4732-4b5f-b2fd-0d0d2c495061)

    Criar um repositório no GitHub com:
        b. Apenas o arquivo release/kubernetes-manifests.yaml
        c. Estrutura recomendada:
        ```
        gitops-microservices/
        └── k8s/
            └── online-boutique.yaml
        ```
    **Repositório chamado: Projeto-k8s-GitOps. Dentro dele, existe uma pasta: k8s. Dentro da pasta k8s, existe um arquivo chamado: online-boutique.yaml.**
      
    

### **2. Deploy do ArgoCD**

1.  **Criei o Namespace do ArgoCD:**
    ```bash
    minikube kubectl -- create namespace argocd
    ```
![namespace](https://github.com/user-attachments/assets/8862a091-55cd-4b21-b49e-e48cf07e5f1b)

2.  **Aplique o Manifesto de Instalação do ArgoCD (Instrui o Kubernetes a aplicar todos os manifestos (arquivos YAML) definidos no link fornecido):**
    ```bash
    minikube kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    ```
![apply](https://github.com/user-attachments/assets/ea1f8f81-fcf3-48a2-9091-0295d75daa7e)

3.  **Fazer Port-Forward para o ArgoCD Server ("encaminhar" uma porta do seu computador local (WSL) para a porta do serviço do ArgoCD no cluster.):**
    **Mantener este comando rodando em um terminal WSL separado:**
    ```bash
    minikube kubectl -- port-forward svc/argocd-server -n argocd 8080:443
    ```

### **3. Acessar e Logar no ArgoCD**

1.  **Obtenher a Senha Inicial do Admin:**
    No terminal WSL, execute:
    ```bash
    minikube kubectl -- -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
    ```
    Anotei esta senha.
2.  **Acesse o ArgoCD no Navegador:**
    Abri meu navegador web e fui para: `https://localhost:8080`.
    (Ignore o aviso de segurança, se aparecer.)
![argo1](https://github.com/user-attachments/assets/593411e3-d1cc-4cae-b591-e3267caece00)
4.  **Faça Login:**
    * **Username:** `admin`
    * **Password:** Usei a senha que obiteve no passo anterior.

### **4. Configurar Repositório e Criar Aplicação no ArgoCD**

1.  **Adicione seu Repositório Git:**
    * No painel do ArgoCD, naveguei para `Settings` (Configurações) > `Repositories` (Repositórios).
    * Cliquei em `+ CONNECT REPO` (ou `+ NEW REPO`).
    * Em `Choose your connection method`, selecionei `VIA HTTP/HTTPS` e `Type: git`.
    * Preenchi:
        * `Repository URL`: `https://github.com/fabsakae/Projeto-k8s-GitOps.git`
        * `Username (optional)`: `fabsakae` (seu usuário do GitHub)
        * `Password (optional)`: Cole seu **Personal Access Token (PAT)** do GitHub aqui.
    * Rolei para cima e clique em `CONNECT`. (Confirmei que o status aparece como `Successful`).
2.  **Crie a Nova Aplicação:**
    * Naveguei para `Applications` (Aplicações).
    * Cliquei em `+ NEW APP` (Novo Aplicativo).
    * Preenchi os detalhes:
        * **GENERAL:**
            * `Application Name`: `online-boutique`
            * `Project Name`: `default`
            * `Sync Policy`: `Manual`
        * **REPOSITORY:**
            * `Repository URL`: Selecione `https://github.com/fabsakae/Projeto-k8s-GitOps.git` na lista suspensa.
            * `Revision`: `HEAD` ou `main`
            * `Path`: `k8s` (esta é a pasta dentro do seu repositório que contém os manifests)
        * **DESTINATION:**
            * `Cluster Name`: `in-cluster`
            * `Namespace`: `default`
    * Cliquei em `CREATE` no canto superior esquerdo da tela.
3.  **Sincronize a Aplicação:**
    * No dashboard de `Applications`, cliquei na nova aplicação `online-boutique`.
    * No canto superior esquerdo da tela de detalhes, cliquei em `SYNC`.
    * Na janela de confirmação que aparece, cliquei em `SYNCHRONIZE`.
    * Aguardei o ArgoCD implantar os recursos. (O status da aplicação deve mudar de `Missing` para `Progressing` e, finalmente, para `Healthy` e `Synced`. Issoleva alguns minutos.)

### **5. Acessar o Frontend da Aplicação Online Boutique**

1.  **Faça Port-Forward para o Serviço Frontend:**
    Abri um **novo terminal WSL** (deixe os outros terminais do Minikube e ArgoCD rodando) e executei:
    ```bash
    minikube kubectl -- port-forward svc/frontend -n default 8080:80
    ```
    * O comando ficará rodando no terminal.
2.  **Acesse a Aplicação no Navegador:**
    Com o `port-forward` rodando, abri o navegador web e acessei:
    `http://localhost:8080` .

    A aplicação Online Boutique está carregada e funcional!

---

## 🎉 Conclusão

Este projeto demonstra a automação da implantação de uma aplicação complexa utilizando a metodologia GitOps e a ferramenta ArgoCD, garantindo a rastreabilidade e a consistência das implantações através de um repositório Git.
