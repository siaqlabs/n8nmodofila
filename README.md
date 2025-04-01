# n8nmodofila
Como instalar o N8N em modo fila

## 1. Instalar Postgres e Redis
```
{
  "services": [
    {
      "type": "postgres",
      "data": {
        "projectName": "n8n-postgres",
        "serviceName": "n8n-postgres",
        "image": "bitnami/postgresql:17.4.0"
      }
    },
    {
      "type": "redis",
      "data": {
        "projectName": "n8n",
        "serviceName": "n8n-redis",
        "password": "senharedis"
      }
    }
  ]
}
```

## 2. Instalar n8n-main
```
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
        "env": "DB_TYPE=postgresdb \nDB_POSTGRESDB_DATABASE=databasename \nDB_POSTGRESDB_HOST=internalhost \nDB_POSTGRESDB_PORT=5432 \nDB_POSTGRESDB_USER=postgres \nDB_POSTGRESDB_PASSWORD=password \nQUEUE_BULL_REDIS_HOST=internalhost \nQUEUE_BULL_REDIS_PORT=6379 \nQUEUE_BULL_REDIS_DB=2 \nQUEUE_BULL_REDIS_USER=default \nQUEUE_BULL_REDIS_PASSWORD=senharedis \nNODE_FUNCTION_ALLOW_EXTERNAL=moment,lodash,moment-with-locales \nEXECUTIONS_DATA_PRUNE=true \nEXECUTIONS_DATA_MAX_AGE=336 \nGENERIC_TIMEZONE=America/Sao_Paulo \nTZ=America/Sao_Paulo \nEXECUTIONS_MODE=queue \nN8N_ENCRYPTION_KEY=encriptionkey \nN8N_HOST=https://$(PRIMARY_DOMAIN) \nN8N_EDITOR_BASE_URL=https://$(PRIMARY_DOMAIN) \nN8N_PROTOCOL=https \nNODE_ENV=production \nWEBHOOK_URL=https://n8nwebhook.seu.dominio \n",
        "mounts": []
      }
    }
  ]
}
```
## 3. Configurar as variáveis
Copiar e colar as Credentials do Postgres e Redis para dentro da Environment Variables do n8n-main

## 4. Duplicar em n8n-hook e n8n-worker
#### A) Criar mais duas N8N através da Template do EasyPanel
#### B) Nomear n8n-hook e n8n-worker
#### C) Copiar e colar as Environment Variables do passo 3

## 5. Configurar a Encryption Key no EasyPanel

1. **Acesse o console do EasyPanel**:
   - Digite:
     ```bash
     vi /home/node/.n8n/config
     ```

2. **Gere uma chave no site:** https://acte.ltd/utils/randomkeygen

3. **Configure a chave**:
   - Abra o arquivo de configuração (`vi /home/node/.n8n/config`).
   - Pressione **`i`** para editar, apague a chave antiga e cole a copiada.
   - Pressione **`Esc`** e digite `:wq`, depois pressione Enter para salvar e sair.

4. **Repita nas demais instâncias**:
   - Certifique-se de usar a mesma chave em todas.
  
5. **Atualize a Encryption Key nas Environment Variables**:
   - Encontre a linha com a chave (`N8N_ENCRYPTION_KEY`).
   - Repita para as 3 instâncias

## 6. Configure os dominios
   - Va na aba "Domains"
   - Configure os domínios igual no seu DNS.

## 7. Configure os comandos

1. **Configurar a instância main**:
   - Acesse a aba **Advanced**.
   - No campo **Command**, insira:
     ```bash
     n8n start
     ```

2. **Configurar a instância webhook**:
   - Acesse a aba **Advanced**.
   - No campo **Command**, insira:
     ```bash
     n8n webhook
     ```

3. **Configurar a instância worker**:
   - Acesse a aba **Advanced**.
   - No campo **Command**, insira:
     ```bash
     n8n worker --concurrency=5
     ```



