# PÓE — Banco de Mídias

Repositório que serve como **banco de dados de mídia** dos atendimentos de WhatsApp, Instagram e Facebook Messenger da PÓE (Escalada da Costura).

Consumido pelo MCP `Poé Mídias` no n8n, que por sua vez alimenta os agentes de Vendas e SAC na NexTags.

## Estrutura

```
catalogo-v3.3.1.json ← catálogo atual consumido pelo backend n8n
catalogo.json        ← catálogo legado
audio/              ← .m4a (AAC-LC); originais .ogg preservados
video/           ← .mp4
imagem/          ← .jpeg
```

## catalogo.json

Cada ativo tem:

| Campo | O que é |
|---|---|
| `id` | Identificador estável (`ativo01`…`ativo24`, `extra01`) |
| `nome` | Nome do ativo no Mapa de Ativos oficial |
| `tipo` | `audio` \| `video` \| `imagem` |
| `disponivel` | `true` se o arquivo existe no repo |
| `motivo_indisponivel` | Presente só quando `disponivel: false` |
| `agente` | `vendas`, `sac` ou ambos |
| `etapa` | Etapa do funil onde entra |
| `perfis` | Perfis a que se aplica (`1` nunca costurou, `2` já costura, `3` empreender) |
| `origens` | Origens de entrada onde vale |
| `objecoes` | Objeções que este ativo responde |
| `quando_usar` | Instrução operacional para o agente |
| `itens[]` | `url`, `bytes`, `mb`, `duracao_s` de cada arquivo |
| `aviso` | Divergência ou cuidado a observar |
| `uso_unico` | Não repetir na mesma conversa |
| `uso_interno` | Nunca enviar ao cliente |

## Entrega

As URLs em `catalogo.json` apontam para **jsDelivr** (`cdn.jsdelivr.net/gh/…`), que tem edge em São Paulo e serve `Content-Type` correto por extensão.

Fallback direto: `raw.githubusercontent.com` (mesmo caminho, ver `base_url_fallback`).

## Limites do WhatsApp

| Tipo | Limite | Formatos |
|---|---|---|
| Imagem | 5 MB | **apenas JPEG/PNG** — a NexTags não entrega WebP, SVG nem GIF |
| Vídeo | 16 MB | MP4 (H.264) |
| Áudio | 16 MB | OGG/Opus, MP3 |

Arquivos acima do limite são marcados com `excede_limite_whatsapp: true` no catálogo.

## Como adicionar ou trocar uma mídia

1. Coloque o arquivo na pasta certa (`audio/`, `video/`, `imagem/`)
2. Atualize a entrada correspondente em `catalogo.json` (`disponivel`, `itens[]`, remova `motivo_indisponivel`)
3. Suba o `versao` e o `atualizado_em`
4. Commit + push

O MCP relê o catálogo a cada chamada. Ao trocar o nome do catálogo, atualize a URL no nó `Ler Catalogo no GitHub` do backend n8n. Prompts com URLs fixas também precisam acompanhar a migração.

> jsDelivr faz cache por até 7 dias em URL sem tag de versão. Para forçar atualização imediata de um arquivo trocado *com o mesmo nome*, use uma URL com commit SHA em vez de `@main`, ou faça purge em `purge.jsdelivr.net`.

## Pendências de produção

11 dos 24 ativos ainda não foram produzidos — ver `indisponiveis` no `catalogo.json`.

O mais crítico é o **`ativo01` (Áudio-mãe de Boas-vindas)**: é usado em 5 das 7 origens de entrada, como 3ª mensagem da conversa.

## Áudios para múltiplos canais — 2026-10-08

- Catálogo atual: `catalogo-v3.3.1.json`; mesmos IDs e regras comerciais da versão 3.3.0.
- Os 20 áudios foram convertidos para M4A com AAC-LC, mono, 44,1 kHz e bitrate alvo de 96 kbps. Os originais OGG permanecem no repo para rollback.
- Arquivos abaixo de 16 MB; duração comparada ao original e decodificação completa validada. Resultados: `auditoria-audios-m4a.json`.
- Enviar como `attachment.type: audio`, usando a URL M4A. A conversão não recupera qualidade perdida no original.
- JSON de exemplo: `exemplos/resposta-audio-adriana.json`.
- Validação dos arquivos e HTTP não comprova entrega. Testar pela NexTags em WhatsApp, Instagram e Messenger. O campo `midias_enviadas_vendas` registra a tentativa da IA, sem confirmação de entrega.
