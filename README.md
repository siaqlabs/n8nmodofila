# 📌 **Como instalar o n8n em modo fila no EasyPanel (Corrigido)**
### 🔹 **Componentes:**
- **PostgreSQL** (banco de dados)
- **Redis** (fila para execução distribuída)
- **n8n-main** (interface e gerenciamento dos fluxos)
- **n8n-webhook** (responsável pelos webhooks)
- **n8n-worker** (executa os fluxos em segundo plano)

---

## 🔥 **Passo 1: Instalar PostgreSQL e Redis com persistência de dados**
### **Criação dos Serviços**
No EasyPanel, crie **dois serviços**: **PostgreSQL** e **Redis**.

📌 **Adicione persistência ao banco de dados PostgreSQL** e ao Redis.

```json
{
  "services": [
    {
      "type": "postgres",
      "data": {
        "projectName": "n8n-postgres",
        "serviceName": "n8n-postgres",
        "image": "bitnami/postgresql:16",
        "mounts": [
          {
            "type": "volume",
            "source": "n8n-postgres-data",
            "target": "/bitnami/postgresql"
          }
        ]
      }
    },
    {
      "type": "redis",
      "data": {
        "projectName": "n8n",
        "serviceName": "n8n-redis",
        "password": "senharedis",
        "env": "appendonly yes"
      }
    }
  ]
}
```
✅ **Isso garante que os dados do banco e da fila Redis não serão apagados ao reiniciar a máquina.**

---

## 🔥 **Passo 2: Instalar o n8n-main (Interface Principal)**
No EasyPanel, crie um **serviço do tipo "App"** para o **n8n-main**.

📌 **Configuração corrigida para evitar perda de dados:**
```json
{
  "services": [
    {
      "type": "app",
      "data": {
        "projectName": "n8n-main",
        "serviceName": "n8n-main",
        "source": {
          "type": "image",
          "image": "n8nio/n8n:latest"
        },
        "domains": [
          {
            "host": "$(EASYPANEL_DOMAIN)",
            "port": 5678
          }
        ],
        "env": "DB_TYPE=postgresdb \nDB_POSTGRESDB_DATABASE=n8n_database \nDB_POSTGRESDB_HOST=n8n-postgres \nDB_POSTGRESDB_PORT=5432 \nDB_POSTGRESDB_USER=postgres \nDB_POSTGRESDB_PASSWORD=password \nDB_POSTGRESDB_SCHEMA=public \nQUEUE_BULL_REDIS_HOST=n8n-redis \nQUEUE_BULL_REDIS_PORT=6379 \nQUEUE_BULL_REDIS_DB=2 \nQUEUE_BULL_REDIS_USER=default \nQUEUE_BULL_REDIS_PASSWORD=senharedis \nNODE_FUNCTION_ALLOW_EXTERNAL=moment,lodash,moment-with-locales \nEXECUTIONS_DATA_PRUNE=true \nEXECUTIONS_DATA_MAX_AGE=336 \nGENERIC_TIMEZONE=America/Sao_Paulo \nTZ=America/Sao_Paulo \nEXECUTIONS_MODE=queue \nN8N_ENCRYPTION_KEY=encryption_key \nN8N_HOST=https://$(PRIMARY_DOMAIN) \nN8N_EDITOR_BASE_URL=https://$(PRIMARY_DOMAIN) \nN8N_PROTOCOL=https \nNODE_ENV=production \nWEBHOOK_URL=https://n8nwebhook.seu.dominio \nN8N_DISABLE_PRODUCTION_MAIN_PROCESS=false \n",
        "mounts": [
          {
            "type": "volume",
            "source": "n8n-data",
            "target": "/home/node/.n8n"
          }
        ]
      }
    }
  ]
}
```

✅ **Isso garante que os dados e fluxos do n8n não sejam apagados ao reiniciar.**  

---

## 🔥 **Passo 3: Configurar as Credenciais no EasyPanel**
1. **Copie as credenciais** do PostgreSQL e Redis (usuário, senha, host, porta).  
2. **Cole essas informações nas Variáveis de Ambiente** do **n8n-main**.  

---

## 🔥 **Passo 4: Criar as instâncias webhook e worker**
Agora precisamos adicionar **duas instâncias adicionais** do n8n:
- **n8n-webhook** (processa requisições HTTP)
- **n8n-worker** (executa fluxos na fila BullMQ)

📌 **No EasyPanel, crie duas novas instâncias do n8n com a mesma configuração do `n8n-main` e apenas mude o nome do serviço.**

**n8n-webhook:**
```json
{
  "type": "app",
  "data": {
    "projectName": "n8n-webhook",
    "serviceName": "n8n-webhook",
    "source": {
      "type": "image",
      "image": "n8nio/n8n:latest"
    },
    "env": "COPIE TODAS AS VARIÁVEIS DO n8n-main",
    "command": "n8n webhook"
  }
}
```

**n8n-worker:**
```json
{
  "type": "app",
  "data": {
    "projectName": "n8n-worker",
    "serviceName": "n8n-worker",
    "source": {
      "type": "image",
      "image": "n8nio/n8n:latest"
    },
    "env": "COPIE TODAS AS VARIÁVEIS DO n8n-main",
    "command": "n8n worker --concurrency=5"
  }
}
```

✅ **Agora o n8n processa tarefas de forma distribuída.**

---

## 🔥 **Passo 5: Configurar a Encryption Key no EasyPanel**
⚠️ **É essencial usar a mesma chave de criptografia (`N8N_ENCRYPTION_KEY`) em todas as instâncias do n8n**.  

1. **Gere uma chave aleatória aqui:** [Random Key Generator](https://acte.ltd/utils/randomkeygen)  
2. **Acesse o console do servidor no EasyPanel:**  
   ```bash
   vi /home/node/.n8n/config
   ```
3. **Edite a chave de criptografia:**
   - Pressione `i` para entrar no modo de edição.  
   - Apague a chave antiga e cole a nova.  
   - Pressione `ESC`, digite `:wq` e pressione `Enter`.  

4. **Repita esse processo nas instâncias `n8n-webhook` e `n8n-worker`.**

---

## 🔥 **Passo 6: Configurar um Backup Automático**
⚠️ **Isso evita perda de dados mesmo se algo der errado.**  

1. Crie um arquivo de backup no servidor:
   ```bash
   nano /home/backup_postgres.sh
   ```
2. Adicione o seguinte código:
   ```bash
   #!/bin/bash
   PGPASSWORD="password" pg_dump -U postgres -h n8n-postgres -p 5432 n8n_database > /home/backup_n8n.sql
   ```
3. Salve (`CTRL+X`, `Y`, `Enter`).
4. Torne o script executável:
   ```bash
   chmod +x /home/backup_postgres.sh
   ```
5. Adicione um **cron job** para rodar automaticamente todos os dias às 3h da manhã:
   ```bash
   crontab -e
   ```
   Adicione esta linha:
   ```bash
   0 3 * * * /bin/bash /home/backup_postgres.sh
   ```

✅ **Agora, seu banco de dados será salvo automaticamente!**

---

## 🚀 **Finalizando**
Agora seu n8n está **seguro contra perdas de dados** e **otimizado para filas** no EasyPanel!  

Se precisar de ajustes, só avisar. 🚀
