# linkedin_finder (Brave-only) — Documentação Completa

Ferramenta de **linha de comando** (CLI) em Python para **encontrar URLs públicas** de perfis no LinkedIn usando a **Brave Search API**.
> **Não** fazemos scraping de páginas do LinkedIn; apenas usamos resultados de busca públicos (título/descrição).

---

## 1) Objetivo e funcionamento
Você informa **Cargo**, **Localidade**, **Tecnologias** e **Quantidade**. O app:
1. Monta a query `site:linkedin.com/in "<cargo>" "<localidade>" "tech1" ...`.
2. Consulta a **Brave Search API**.
3. Extrai **Nome** (do título do resultado), **URL do LinkedIn**, **Localidade** (informada) e **Tecnologias** (detectadas no snippet).
4. Exibe uma tabela e gera `output/resultados.csv`.

---

## 2) Pré-requisitos
- **Python 3.10+** (funciona bem no 3.13)
- **Chave da Brave Search API** (há plano gratuito)
- Windows/macOS/Linux
- (Opcional) VS Code

Dependências (no `requirements.txt`):
```
requests
python-dotenv
typer
rich
rapidfuzz
```

---

> **IMPORTANTE**: Rode o comando `python -m linkedin_finder.cli` **dentro da PASTA OUTER** (`...\linkedin_finder\`).  
> Se você entrar na PASTA INNER (`...\linkedin_finder\linkedin_finder\`), o comando muda para `python -m cli`.

---

## 3) Instalação (Windows PowerShell)
```powershell
cd "<pasta-raiz-do-projeto>"
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install --upgrade pip
pip install -r .\linkedin_finder\requirements.txt
```

macOS/Linux:
```bash
cd "<pasta-raiz-do-projeto>"
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r ./linkedin_finder/requirements.txt
```

---

## 4) Configurar `.env` (PASTA OUTER)
Crie/edite `linkedin_finder/.env` com:
```
BRAVE_API_KEY=SEU_TOKEN_DA_BRAVE_AQUI
```
Teste se foi lido:
```powershell
python -c "from dotenv import load_dotenv; load_dotenv(); import os; print('BRAVE=', (os.getenv('BRAVE_API_KEY') or '')[:6])"
```

---

## 5) Executar
Na **PASTA OUTER** (`...\linkedin_finder\`):
```powershell
python -m linkedin_finder.cli
```
Responda aos prompts. O CSV sai em `output\resultados.csv`.

> Se você estiver na **PASTA INNER** (`...\linkedin_finder\linkedin_finder\`), use:
> ```powershell
> python -m cli
> ```
> (e o `.env` precisa estar **nessa pasta** ou exporte a variável no shell.)

---

## 6) O que cada arquivo faz

### `cli.py`
- CLI com **Typer** + tabela com **Rich**.
- Pergunta filtros, monta a query, chama `search_brave()`.
- Extrai nome do título (`guess_name_from_title`), detecta tecnologias (`extract_technologies`), imprime tabela e salva CSV.

### `config.py`
- Lê `.env` (**python-dotenv**) e valida `BRAVE_API_KEY` com `require_env()`.

### `search/brave.py`
- Chama `https://api.search.brave.com/res/v1/web/search` com header `X-Subscription-Token`.
- Normaliza idioma (`pt` → `pt-br`), valida línguas aceitas.
- Retorna itens simplificados: `{title, link, snippet}` e mensagens de erro claras.

### `enrich/keywords.py`
- Vocabulário `BASE_TECHS` (Python, Node, AWS, Scrum, etc.).
- `extract_technologies(text, user_techs)`: normalização, *substring*, *fuzzy matching* e pequenas correções.

### `models.py`
- `Profile` (dataclass): `name`, `linkedin_url`, `location`, `technologies`.

### `output/export.py`
- Grava `output/resultados.csv` (módulo nativo `csv`).

---

## 7) Exemplos de uso
```
Qual cargo (ex.: Desenvolvedor Python): Desenvolvedor Python
Localidade (ex.: SP, São Paulo, Rio de Janeiro): SP
Tecnologias desejadas (separadas por vírgula) [Python]: Python, Django
Quantidade aproximada de perfis (1-50) [20]: 3
```

Geração:
```
Arquivo gerado: output/resultados.csv
```

---

## 8) Erros comuns & “ficheiro não encontrado”

### a) “No module named linkedin_finder.cli”
- Você está no diretório **errado**.
  - Solução: entre na **PASTA OUTER** (`...\linkedin_finder\`) e rode `python -m linkedin_finder.cli`.

### b) “file/ficheiro not found” ao rodar/ler `.env` ou `requirements.txt`
- O comando foi executado fora da pasta correta.
  - Para instalar dependências: `pip install -r .\linkedin_finder\requirements.txt` **a partir da raiz**; ou `pip install -r .\requirements.txt` **estando na OUTER**.
  - Para `.env`: ele deve estar **na pasta onde você roda o comando**.

### c) `RuntimeError: BRAVE_API_KEY ausente`
- Falta definir `BRAVE_API_KEY` no `.env` (na pasta onde você roda).

### d) `HTTP 401/403 (Brave)`
- Token inválido/quota/expirado. Verifique sua conta e a variável no `.env` corrento.

### e) `HTTP 422 (VALIDATION)` com `search_lang`
- A Brave não aceita `pt`; use `pt-br` (o provider já normaliza).

---

## 9) Personalizações
- **Idioma/país**: mude `lang="pt-br"` e `country="br"` em `cli.py` ou ajuste default em `brave.py`.
- **Paginação**: dá para implementar via `cursor` da Brave para mais resultados.
- **Vocabulário**: acrescente termos ao `BASE_TECHS`.
- **Exportar XLSX**: use `openpyxl` (ou mantenha CSV).

---

## 10) Comandos úteis (Windows)
```powershell
# Ativar venv
.\.venv\Scripts\Activate.ps1

# Instalar dependências
pip install -r .\linkedin_finder\requirements.txt

# Rodar CLI a partir da OUTER
cd .\linkedin_finder\
python -m linkedin_finder.cli

# Rodar CLI a partir da INNER
cd .\linkedin_finder\linkedin_finder\
python -m cli
```
