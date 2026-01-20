# 🗺️ MAPA DO CLI VECTORGOV

> **Versão**: 0.1.4
> **Data**: Janeiro 2025
> **Objetivo**: Documentação completa da arquitetura e funcionamento do CLI VectorGov

---

## 📋 Índice

1. [Visão Geral](#visão-geral)
2. [Arquitetura de Alto Nível](#arquitetura-de-alto-nível)
3. [Estrutura de Arquivos](#estrutura-de-arquivos)
4. [Comandos](#comandos)
5. [Fluxo de Dados](#fluxo-de-dados)
6. [Configurações](#configurações)
7. [Integração com SDK](#integração-com-sdk)
8. [Exemplos de Uso](#exemplos-de-uso)
9. [Links Úteis](#links-úteis)

---

## 📖 Visão Geral

O VectorGov CLI é uma ferramenta de linha de comando para interagir com a API VectorGov diretamente do terminal. Ideal para scripts, automações, pipes Unix e integração com outras ferramentas.

### Características Principais

| Característica | Descrição |
|----------------|-----------|
| **Baseado no SDK** | Usa o SDK Python internamente para chamadas à API |
| **Múltiplos Formatos** | Saída em tabela, JSON ou raw para pipes |
| **Autocomplete** | Suporte a shell completion (bash, zsh) |
| **Configuração Flexível** | Via variáveis de ambiente, arquivo YAML ou flags |
| **Windows/Linux/Mac** | Compatível com todos os sistemas operacionais |

### Quando Usar o CLI vs SDK

| Cenário | Recomendação |
|---------|--------------|
| Scripts shell/bash | **CLI** |
| Automações com pipes | **CLI** |
| Testes rápidos no terminal | **CLI** |
| Aplicações Python | **SDK** |
| Integração com LLMs | **SDK** |
| Aplicações web/API | **SDK** |

---

## 🏗️ Arquitetura de Alto Nível

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           ARQUITETURA DO CLI                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   Terminal                    CLI                         SDK               │
│   ┌─────────┐               ┌─────────┐                ┌─────────┐         │
│   │$ vectorgov│─────────────▶│  Typer  │───────────────▶│VectorGov│         │
│   │  search  │              │ (main.py)│               │(client) │         │
│   │  "query" │              │         │               │         │         │
│   └─────────┘               └────┬────┘               └────┬────┘         │
│                                  │                         │               │
│                                  │                         │ HTTPS         │
│                                  │                         ▼               │
│   ┌─────────┐               ┌────▼────┐          ┌─────────────────┐       │
│   │ stdout  │◀──────────────│ Output  │          │   API VectorGov │       │
│   │ (table/ │               │Formatter│          │vectorgov.io/api │       │
│   │  json)  │               │         │          │                 │       │
│   └─────────┘               └─────────┘          └─────────────────┘       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 📁 Estrutura de Arquivos

```
vectorgov-cli/
├── vectorgov_cli/
│   ├── __init__.py           # Versão e metadata
│   ├── main.py               # Entry point Typer (todos os comandos)
│   ├── config.py             # ConfigManager (YAML + env vars)
│   └── output.py             # OutputFormatter (tabela, JSON, raw)
├── pyproject.toml            # Configuração do pacote
├── README.md                 # Documentação de uso
├── CHANGELOG.md              # Histórico de versões
└── LICENSE                   # MIT License
```

### Arquivos Principais

| Arquivo | Responsabilidade |
|---------|------------------|
| `main.py` | Definição de todos os comandos Typer |
| `config.py` | Gerenciamento de configuração (arquivo + env) |
| `output.py` | Formatação de saída (tabela Rich, JSON, raw) |

---

## 🔧 Comandos

### Comandos de Busca

| Comando | Descrição | Exemplo |
|---------|-----------|---------|
| `search` | Busca semântica | `vectorgov search "O que é ETP?"` |
| `ask` | Contexto para LLMs | `vectorgov ask "Quando o ETP pode ser dispensado?"` |
| `tokens` | Estimativa de tokens | `vectorgov tokens "pesquisa de preços" --top-k 10` |

### Comandos de Feedback

| Comando | Descrição | Exemplo |
|---------|-----------|---------|
| `feedback send` | Envia like/dislike | `vectorgov feedback send <query_id> --like` |

### Comandos de Documentos

| Comando | Descrição | Exemplo |
|---------|-----------|---------|
| `docs list` | Lista documentos | `vectorgov docs list` |
| `docs info` | Detalhes de documento | `vectorgov docs info LEI-14133-2021` |

### Comandos de Autenticação

| Comando | Descrição | Exemplo |
|---------|-----------|---------|
| `auth login` | Configura API key | `vectorgov auth login` |
| `auth status` | Verifica autenticação | `vectorgov auth status` |
| `auth logout` | Remove API key | `vectorgov auth logout` |

### Comandos de Configuração

| Comando | Descrição | Exemplo |
|---------|-----------|---------|
| `config list` | Lista configurações | `vectorgov config list` |
| `config get` | Obtém valor | `vectorgov config get api_key` |
| `config set` | Define valor | `vectorgov config set default_top_k 10` |
| `config delete` | Remove valor | `vectorgov config delete default_mode` |

---

## 🔄 Fluxo de Dados

### Busca Simples

```
1. Usuário executa: vectorgov search "O que é ETP?"
2. main.py:search() parseia argumentos
3. ConfigManager carrega API key (env/arquivo)
4. VectorGov SDK faz chamada HTTP à API
5. API retorna resultados JSON
6. OutputFormatter formata para tabela/JSON
7. Resultado é impresso no stdout
```

### Integração com Pipes Unix

```bash
# Busca → jq → processamento
vectorgov search "ETP" --raw | jq '.hits[0].text'

# Busca → salvar arquivo
vectorgov search "licitação" --output json > resultados.json

# Busca → obter query_id → feedback
QUERY_ID=$(vectorgov search "ETP" --raw | jq -r '.query_id')
vectorgov feedback send $QUERY_ID --like
```

---

## ⚙️ Configurações

### Prioridade de Configuração

```
1. Flags de linha de comando (maior prioridade)
2. Variáveis de ambiente
3. Arquivo ~/.vectorgov/config.yaml
4. Valores padrão (menor prioridade)
```

### Variáveis de Ambiente

| Variável | Descrição | Exemplo |
|----------|-----------|---------|
| `VECTORGOV_API_KEY` | API key | `vg_xxx...` |
| `VECTORGOV_DEFAULT_MODE` | Modo de busca | `fast`, `balanced`, `precise` |
| `VECTORGOV_DEFAULT_TOP_K` | Resultados padrão | `5` |

### Arquivo de Configuração

**Localização**: `~/.vectorgov/config.yaml`

```yaml
api_key: vg_sua_chave
default_mode: balanced
default_top_k: 5
output_format: table
```

---

## 🔗 Integração com SDK

O CLI usa o SDK Python internamente:

```python
# Internamente no CLI (main.py)
from vectorgov import VectorGov

vg = VectorGov(api_key=config.api_key)
results = vg.search(query, top_k=top_k, mode=mode)
```

### Dependências

```toml
# pyproject.toml
dependencies = [
    "vectorgov>=0.10.0",  # SDK Python
    "typer>=0.9.0",       # Framework CLI
    "rich>=13.0.0",       # Tabelas e formatação
    "pyyaml>=6.0",        # Arquivo de configuração
]
```

---

## 📚 Exemplos de Uso

### Busca Básica

```bash
# Busca simples
vectorgov search "O que é ETP?"

# Com mais resultados
vectorgov search "pesquisa de preços" --top-k 10

# Modo preciso (mais lento, mais acurado)
vectorgov search "licitação" --mode precise
```

### Saída JSON para Scripts

```bash
# JSON formatado
vectorgov search "ETP" --output json

# JSON raw (para pipes)
vectorgov search "ETP" --raw | jq '.hits | length'
```

### Contexto para LLMs

```bash
# Obtém contexto e mostra código de exemplo
vectorgov ask "Quando o ETP pode ser dispensado?" --code

# Saída em formato messages (pronto para LLM)
vectorgov ask "critérios de julgamento" --output json
```

### Estimativa de Tokens

```bash
# Estima tokens antes de usar LLM
vectorgov tokens "O que é ETP?" --top-k 5

# Comparação com limites de modelos
vectorgov tokens "pesquisa de preços" --output json
```

---

## 🔗 Links Úteis

| Recurso | URL |
|---------|-----|
| **PyPI** | https://pypi.org/project/vectorgov-cli/ |
| **Documentação** | https://vectorgov.io/documentacao/cli-instalacao |
| **SDK Python** | https://pypi.org/project/vectorgov/ |
| **API Reference** | https://vectorgov.io/documentacao |
| **Playground** | https://vectorgov.io/playground |

---

## 📝 Histórico de Versões

| Versão | Data | Mudanças |
|--------|------|----------|
| 0.1.4 | 20/01/2025 | Comando `feedback send` para evitar conflito Typer |
| 0.1.3 | 20/01/2025 | Correção caracteres box-drawing no Windows |
| 0.1.2 | 20/01/2025 | Correção Unicode no Windows (cp1252) |
| 0.1.1 | 20/01/2025 | Comando `tokens` para estimativa |
| 0.1.0 | 20/01/2025 | Primeira versão pública |

---

*Este documento é atualizado a cada release do CLI.*
