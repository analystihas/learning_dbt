# 📘 LEARNING_DBT — dbt-core com Databricks (Guia de Autoaprendizado)

Este repositório existe para **aprender dbt-core na prática**, usando **Databricks** como data warehouse e **uv** para gerenciamento de ambiente Python.

O objetivo é:

* não depender de dbt Cloud
* entender **como o dbt realmente funciona**
* ter um setup reproduzível, local e limpo

---

## 🧱 Stack utilizada

* Python 3.12
* `uv` (gerenciamento de dependências e venv)
* `dbt-core`
* `dbt-databricks`
* Databricks SQL Warehouse
* `.env` para segredos
* `profiles.yml` versionado no projeto

---

## 1️⃣ Instalar dependências (dbt-core + Databricks)

Dentro do projeto:

```bash
uv add dbt-core dbt-databricks python-dotenv
```

Verifique:

```bash
uv run dbt --version
```

---

## 2️⃣ Inicializar o projeto dbt

```bash
uv run dbt init
```

> ⚠️ Observação
> Se você errar algo aqui (token, adapter etc), **não tem problema**.
> O importante é entender que:
>
> * o projeto dbt é uma coisa
> * o `profiles.yml` é outra
>
> Eles podem ser ajustados depois sem recriar tudo.

---

## 3️⃣ Databricks: onde pegar Host e HTTP Path

Ao usar Databricks, o dbt **não conecta direto no cluster**, e sim via **SQL Warehouse**.

### 📍 Onde encontrar

No Databricks UI:

```
Compute → SQL Warehouses → (seu warehouse)
```

Você vai precisar de:

* **Server Hostname**
* **HTTP Path**
* **Personal Access Token (PAT)**

⚠️ Durante o debug, pode aparecer o warning:

```
WARNING: thrift.transport.sslcompat: using legacy validation callback
```

👉 Isso **não é erro** e pode ser ignorado.

---

## 4️⃣ Configurar variáveis de ambiente (`.env`)

Crie um arquivo `.env` na raiz do projeto:

```env
DATABRICKS_HOST=https://dbc-XXXXXXXX.cloud.databricks.com
DATABRICKS_HTTP_PATH=/sql/1.0/warehouses/XXXXXXXX
DATABRICKS_TOKEN=dapiXXXXXXXXXXXXXXXX
DATABRICKS_CATALOG=dbt_tutorial_dev
DATABRICKS_SCHEMA=analytics
```

🔒 **Nunca commite esse arquivo**.

---

## 5️⃣ `profiles.yml` (dentro do projeto)

Estrutura recomendada:

```text
profiles/
└─ profiles.yml
```

Conteúdo:

```yaml
learning_dbt:
  outputs:
    dev:
      type: databricks
      host: "{{ env_var('DATABRICKS_HOST') }}"
      http_path: "{{ env_var('DATABRICKS_HTTP_PATH') }}"
      token: "{{ env_var('DATABRICKS_TOKEN') }}"
      catalog: "{{ env_var('DATABRICKS_CATALOG') }}"
      schema: "{{ env_var('DATABRICKS_SCHEMA') }}"
      threads: 4
  target: dev
```

⚠️ O nome `learning_dbt` **precisa ser igual** ao `name:` no `dbt_project.yml`.

---

## 6️⃣ Usando `profiles.yml` do projeto (não o global)

Sempre rode dbt assim:

```bash
uv run dbt debug --profiles-dir ./profiles
```

Isso ignora completamente `~/.dbt`.

---

## 7️⃣ Criar um alias para facilitar o uso (Git Bash)

Como o dbt **não carrega `.env` automaticamente**, criamos um alias que resolve isso.

### 📍 Editar o arquivo certo

```bash
code ~/.bashrc
```

### ✍️ Adicionar o alias

```bash
alias dbt='uv run python -c "from dotenv import load_dotenv; load_dotenv(); import subprocess, sys; subprocess.run([\"dbt\"] + sys.argv[1:])"'
```

Depois:

```bash
source ~/.bashrc
```

---

## 8️⃣ Rodar dbt normalmente 🚀

Agora você pode usar dbt como se fosse global:

```bash
dbt debug --profiles-dir ./profiles
dbt run --profiles-dir ./profiles
dbt test --profiles-dir ./profiles
```

Se aparecer:

```
Connection test: OK
```

🎉 Ambiente configurado com sucesso.

---

## 🧠 Conceitos importantes aprendidos aqui

* dbt-core **não depende de dbt Cloud**
* `profiles.yml` pode (e deve) ficar no projeto
* segredos vão para `.env`
* dbt **não carrega `.env` sozinho**
* Databricks usa **PAT + SQL Warehouse**
* `uv run` é o equivalente moderno do `venv + pip`

---

