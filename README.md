# ScriptForge AI — projeto Netlify

Este projeto inclui:
- `index.html`: interface do ScriptForge AI que chama a função `/ .netlify/functions/generate` (sem espaço na URL real).
- `netlify/functions/generate.mjs`: função de servidor que chama a API do Groq usando `GROQ_API_KEY`.
- `netlify.toml`: configuração de publicação e diretório das funções.

## Importante
Não coloque a chave de API em `index.html`, no GitHub ou neste ZIP. Configure `GROQ_API_KEY` como variável de ambiente no Netlify, com escopo para Functions.

## Deploy
Funções do Netlify precisam de um deploy com build/functions, não apenas de upload manual de um HTML isolado pelo Netlify Drop. O caminho mais simples é colocar estes arquivos num repositório Git e conectar esse repositório ao site no Netlify, depois disparar um novo deploy.
