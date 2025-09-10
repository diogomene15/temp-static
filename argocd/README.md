# Playbook ArgoCD

Esse playbook Ansible permite a conexão com servidor remoto e a realização de:
- Instalação de `K3s` com `traefik` configurado
- Instalação de `kubectl`
- Criação de usuário para configuração de cluster
- Instalação de `ArgoCD` no kluster criado
- Configuração de `traefik` para funcionamento em portas externas personalizadas

# Como utilizar o playbook do ArgoCD?

1. Alterar hostname

No arquivo [`argo-ingress.yaml`](./argo-ingress.yaml) é necessária a alteração de **hostname** que será utilizado para redirecionamento via `traefik`

2. Alterar configurações de acesso aos hosts remotos

No arquivo [`hosts.ini`](./hosts.ini) deve ser realizada a inserção de informações válidas dos servidores remotos que deverão ser configurados

3. (opcional) Alterar portas externas do `traefik`

No arquivo [`traefik-config.yaml`](./traefik-config.yaml) pode ser realizada a alteração dos valores de `exposedPort`

4. Aplicar playbook

Executar comando
```shell
ansible-playbook -i hosts.ini playbook.yaml 
```
\* É necessário que o `Ansible` esteja instalado e corretamente configurado

# Roteiro manual

O roteiro a seguir constitui os passos necessários para realizar manualmente a configuração dos serviços:

1. Atualizar registries e pacotes do computador
apt-get update && apt-get upgrade -y

2. Instalar docker
apt-get install docker.io -y

3. Baixar e instalar kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl && rm kubectl

4. Baixar e instalar K3s
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable=traefik" sh -s -

5. Criar um usuário novo
adduser quati

6. Adicionar usuário ao grupo de docker e k3s
addgroup k3s

7. Configurar permissões k3s
chown -R root:k3s /etc/rancher
sudo chmod -R 770 /etc/rancher

usermod -aG docker quati && usermod -aG k3s quati

8. Trocar para usuário novo
su - quati

9. Configurar k3s no user
echo "export KUBECONFIG=/etc/rancher/k3s/k3s.yaml" >> ~/.bashrc && source ~/.bashrc

10. Deploy argocd
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

11. Realizar patch de serviço de argocd
kubectl apply -f argo-service.yaml

12. Delete o pod de server do argocd
kubectl delete pod/argocd-server-6d99648bc-z29dw -n argocd
