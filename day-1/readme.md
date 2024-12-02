O **ArgoCD** (Continuous Delivery) é uma ferramenta moderna para **deploy contínuo** em Kubernetes. Ele permite que equipes de desenvolvimento automatizem e sincronizem a entrega de aplicativos em clusters Kubernetes, garantindo que o estado do cluster permaneça consistente com o estado desejado descrito nos repositórios Git (GitOps).

### **Por que usar o ArgoCD?**
- **GitOps como base**: O repositório Git é a única fonte de verdade.
- **Automação**: Verifica e sincroniza automaticamente o estado dos aplicativos no cluster.
- **Observabilidade**: Interface web e CLI para visualizar e gerenciar o estado dos aplicativos.
- **Rollback facilitado**: Reverter mudanças é tão simples quanto retornar para um commit anterior no Git.

---

### **Como funciona o ArgoCD?**
1. **Configuração no repositório Git**: O estado desejado do aplicativo é descrito em arquivos YAML/Helm Charts/Kustomize.
2. **Sincronização**: O ArgoCD monitora o repositório e compara o estado real (no cluster) com o estado desejado (no Git).
3. **Ação**: 
   - Sincroniza automaticamente (opcional) ou manualmente.
   - Atualiza o cluster para que fique alinhado com o estado do Git.
4. **Visualização e gerenciamento**: O painel do ArgoCD mostra as diferenças e permite ações corretivas.

---

### **Exemplo simples**

#### **1. Pré-requisitos**
- Kubernetes funcionando.
- Repositório Git com os manifestos da aplicação (ex.: `deployment.yaml`).
- ArgoCD instalado no cluster.

#### **2. Instalação do ArgoCD**
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

#### **3. Configuração do ArgoCD**
- Faça login no painel do ArgoCD:
  ```bash
  kubectl port-forward svc/argocd-server -n argocd 8080:443
  ```
  - Acesse: [https://localhost:8080](https://localhost:8080).
  - Credenciais padrão:
    - Usuário: `admin`.
    - Senha: saída do comando:
      ```bash
      kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
      ```

#### **4. Configuração de um aplicativo**
Crie um repositório Git com a seguinte estrutura:

```plaintext
app-config/
│
├── deployment.yaml
├── service.yaml
```

#### **5. Criação de um aplicativo**
No painel do ArgoCD ou via CLI:
```bash
argocd app create my-app \
  --repo https://github.com/seu-usuario/app-config.git \
  --path ./ \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

#### **6. Sincronização**
O ArgoCD agora começa a monitorar o repositório. Para aplicar as configurações no cluster:
```bash
argocd app sync my-app
```

---

### **Recursos adicionais do ArgoCD**
1. **Hooks customizados**:
   - Para executar tarefas antes ou depois dos deploys, como migrações.
2. **Saúde do aplicativo**:
   - Verifica se os recursos do Kubernetes estão em um estado saudável.
3. **Integrações**:
   - Funciona com Helm Charts, Kustomize, Jsonnet, etc.
4. **Controle de acesso baseado em RBAC**:
   - Permite configurar permissões detalhadas.

---

### **Exemplo prático de GitOps**

1. Atualize um manifesto no repositório:
   - Modifique `replicas` de 2 para 3 em `deployment.yaml`:
     ```yaml
     replicas: 3
     ```

2. Commit e push:
   ```bash
   git add deployment.yaml
   git commit -m "Atualizar replicas para 3"
   git push origin main
   ```

3. O ArgoCD detecta a mudança automaticamente e aplica ao cluster (se configurado para auto-sync).

