[<img src="img/logo.png" alt="Logo do Markdown" width="10%">](https://www.instagram.com/cafecomlinux?igsh=NnppemZ3eDVrdjg5)

### 🚀 Guia de Instalação e Configuração do Dragonfly no Kubernetes

### 📝 Sobre o Dragonfly
#### O Dragonfly é uma alternativa moderna, in-memory e de alta performance ao Redis e Memcached, projetada para extrair o máximo do hardware atual através de uma arquitetura multi-threaded (shared-nothing).

[Site Oficial do Dragonfly](https://www.dragonflydb.io/)

[Repositório no GitHub](https://github.com/dragonflydb/dragonfly)

## ⚙️ Procedimento de Instalação

#### 1. Instalação do Operador do Dragonfly
##### O primeiro passo é instalar o operador que gerenciará o ciclo de vida das instâncias do Dragonfly no cluster.
```bash
kubectl apply -f https://raw.githubusercontent.com/dragonflydb/dragonfly-operator/main/manifests/dragonfly-operator.yaml
```

#### 2. Validando a Instalação do Operador
##### Verifique se o deployment do operador foi iniciado corretamente e está com o status Ready:
```bash
kubectl get deployment -n dragonfly-operator-system
```

### 📂 Configurando o Ambiente
#### 3. Criando o Namespace
##### Crie um namespace isolado para hospedar a sua instância do Dragonfly:
```bash
kubectl create namespace dragonfly
```
#### 4. Criação do Secret para a Senha do Dragonfly
##### Gere um Secret do Kubernetes para armazenar a senha de autenticação de forma segura. Substitua <senha> pela sua credencial.
```bash
kubectl create secret generic dragonfly-auth \
  --from-literal=password=<senha> \
  -n dragonfly
```
#### 5. Criando o yaml da instância do Dragonfly
##### Crie um arquivo chamado (dragonfly.yaml) com o seguinte conteúdo.
##### OBS: A configuração do request e limits vai de acordo com a sua necessidade
```bash
apiVersion: dragonflydb.io/v1alpha1
kind: Dragonfly
metadata:
  name: dragonfly
  namespace: dragonfly
spec:
  replicas: 1
  authentication:
    passwordFromSecret:
      name: dragonfly-auth
      key: password
  resources:
    requests:
      cpu: 1
      memory: 2Gi
    limits:
      cpu: 2
      memory: 4Gi
# --- BLOCO DE CONFIGURAÇÃO DA PERSISTÊNCIA CASO NECESSÁRIO (Opcional bloco abaixo) ---
  args:
    - "--dir=/data"
    - "--dbfilename=dump"
  snapshot:
    cron: "*/5 * * * *" # Roda o cron a cada 5 segundos para salvar a persistência
    persistentVolumeClaimSpec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 5Gi # Tamanho do PVC
      storageClassName: standard

```

### 🚀 Implantação da Instância
#### 6. Criando a Instância do Dragonfly
##### Aplique o seu arquivo de manifesto (dragonfly.yaml) que define as configurações da instância.
```bash
kubectl apply -f dragonfly.yaml
```
#### 6. Verificando o Status da Implantação
##### Monitore os recursos criados para garantir que tudo subiu sem erros:
```bash
kubectl get dragonfly -n dragonfly
kubectl get pods -n dragonfly
```
### 8. Detalhes de Conexão
##### Para descobrir o IP e detalhes do serviço criado, execute:
```bash
kubectl get service dragonfly -n dragonfly
```
##### 📌 Nota: A porta padrão utilizada pelo Dragonfly é a 6379.

### 🧪 Testando o Dragonfly
#### 9. Acessando o Terminal Interativo
##### O comando abaixo criará um Pod temporário rodando o redis-cli e abrirá o terminal diretamente conectado à sua instância do Dragonfly.
```bash
kubectl run -it --rm --restart=Never redis-cli \
  --image=redis:7.0.10 \
  -n dragonfly \
  -- redis-cli -h dragonfly
```

#### 10. Comandos de Validação interna
##### Assim que o terminal do redis-cli for aberto, realize a autenticação e os testes de leitura/escrita:
```bash
# 1. Autenticação (Substitua pela senha que você definiu no Secret)
dragonfly:6379> AUTH MinhaSenhaSuperSegura
OK

# 2. Teste de Conectividade
dragonfly:6379> PING
PONG

# 3. Teste de Escrita
dragonfly:6379> SET canal "teste bem sucedido"
OK

# 4. Teste de Leitura
dragonfly:6379> GET canal
"teste bem sucedido"

# 5. Sair do terminal
dragonfly:6379> exit
```
