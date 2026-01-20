# Changelog

Todas as mudancas notaveis neste projeto serao documentadas neste arquivo.

O formato e baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.0.0/),
e este projeto adere ao [Versionamento Semantico](https://semver.org/lang/pt-BR/).

## [0.1.4] - 2025-01-20

### Corrigido
- Comando `feedback` agora usa subcomando `send` para evitar conflito com Typer
  - Uso: `vectorgov feedback send <query_id> --like`

## [0.1.3] - 2025-01-20

### Corrigido
- Caractere box-drawing (─) em tabelas causava erro no Windows (cp1252)
- Substituido por hifen (-) para compatibilidade

## [0.1.2] - 2025-01-20

### Corrigido
- Caracteres Unicode incompativeis com Windows (cp1252)
  - ✓ → OK
  - ✗ → ERRO
  - 👍 → +1
  - 👎 → -1
- Removidos acentos de mensagens para evitar encoding issues

## [0.1.1] - 2025-01-20

### Adicionado
- Comando `tokens` para estimativa de tokens antes de usar LLM
  - Mostra contexto, system prompt e query tokens
  - Compara com limites de modelos populares (GPT-4o, Claude, Gemini)
  - Suporta saida JSON

## [0.1.0] - 2025-01-20

### Adicionado
- Primeira versao publica do VectorGov CLI
- Comando `search` para busca semantica em legislacao
- Comando `ask` para obter contexto para LLMs
- Comando `feedback send` para enviar like/dislike
- Comando `docs list` e `docs info` para listar documentos
- Comando `auth login/status/logout` para autenticacao
- Comando `config set/get/list/delete` para configuracoes
- Suporte a variaveis de ambiente (VECTORGOV_API_KEY)
- Arquivo de configuracao em ~/.vectorgov/config.yaml
- Saida em tabela, JSON ou raw
