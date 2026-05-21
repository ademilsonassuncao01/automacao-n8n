# 🚨 Monitor Logístico Brasil — Automação com n8n

> Sistema de monitoramento de incidentes operacionais em tempo real para operações logísticas no Brasil, utilizando automação com n8n, Google News RSS, INMET e Telegram.

---

## 📋 Índice

- [Sobre o Projeto](#sobre-o-projeto)
- [Como Funciona](#como-funciona)
- [Arquitetura do Workflow](#arquitetura-do-workflow)
- [Fontes de Dados](#fontes-de-dados)
- [Classificação de Incidentes](#classificação-de-incidentes)
- [Exemplo de Alerta](#exemplo-de-alerta)
- [Pré-requisitos](#pré-requisitos)
- [Configuração Passo a Passo](#configuração-passo-a-passo)
- [Estrutura de Dados](#estrutura-de-dados)
- [Próximos Passos](#próximos-passos)
- [Tecnologias Utilizadas](#tecnologias-utilizadas)

---

## 📌 Sobre o Projeto

Operações logísticas dependem de informação rápida para tomar decisões. Um bloqueio de rodovia, uma manifestação ou um evento climático extremo pode impactar entregas, rotas e toda a cadeia de distribuição.

Este projeto automatiza o monitoramento de incidentes operacionais no Brasil, coletando dados de múltiplas fontes, classificando por severidade, identificando regiões afetadas e enviando relatórios consolidados automaticamente via Telegram — sem nenhuma intervenção humana.

**Problema resolvido:** Gestores de logística gastam tempo monitorando manualmente notícias, grupos de WhatsApp e redes sociais para identificar riscos operacionais. Esta automação centraliza tudo em um único canal com classificação inteligente.

---

## ⚙️ Como Funciona

O workflow executa automaticamente a cada **30 minutos** seguindo este fluxo:

```
1. COLETA     → 6 feeds RSS do Google News (temas específicos de logística)
2. MERGE      → Unifica todos os itens em uma única fila
3. FILTRAGEM  → Mantém apenas notícias publicadas HOJE
4. CLASSIFICAÇÃO → Determina severidade (CRÍTICO / ALTO / MÉDIO / BAIXO)
5. LOCALIZAÇÃO   → Identifica região, UF, rodovia e coordenadas GPS
6. CONSOLIDAÇÃO  → Monta relatório agrupado por severidade e região
7. ALERTA        → Envia relatório ao canal Telegram
8. HISTÓRICO     → Salva todos os incidentes no Google Sheets
```

---

## 🏗️ Arquitetura do Workflow

```
⏰ Trigger 30min
    │
    ├──► 📰 RSS: Bloqueios de Rodovias
    ├──► 📰 RSS: Manifestações e Protestos
    ├──► 📰 RSS: Acidentes (caminhões/carretas)
    ├──► 🌧️ RSS: Desastres Climáticos
    ├──► ✊ RSS: Greves e Paralisações
    └──► 🚔 RSS: Ocorrências PRF
              │
         [Merge 1→2→3]
              │
    🧠 Filtrar Hoje + Classificar
    (filtra por data, deduplica,
     classifica severidade,
     extrai localização)
              │
        ┌─────┴─────┐
        │           │
   📋 Consolidar   📊 Google Sheets
    Relatório       Salvar Incidente
        │
   📲 Telegram
   Enviar Relatório
```

---

## 📡 Fontes de Dados

| Fonte | Tipo | Frequência | Cobertura |
|-------|------|-----------|-----------|
| Google News RSS — Bloqueios | RSS Feed | ~15-30min | Brasil |
| Google News RSS — Manifestações | RSS Feed | ~15-30min | Brasil |
| Google News RSS — Acidentes | RSS Feed | ~15-30min | Brasil |
| Google News RSS — Clima | RSS Feed | ~15-30min | Brasil |
| Google News RSS — Greves | RSS Feed | ~15-30min | Brasil |
| Google News RSS — PRF | RSS Feed | ~15-30min | Brasil |

**Filtro de frescor:** Apenas notícias com `pubDate` do dia atual são processadas, garantindo que somente informações do dia cheguem ao relatório.

---

## 🎯 Classificação de Incidentes

### Severidade

| Nível | Emoji | Exemplos de Keywords |
|-------|-------|---------------------|
| CRÍTICO | 🔴 | bloqueio total, manifestação, deslizamento, acidente fatal, tombamento de caminhão |
| ALTO | 🟠 | interdição, alagamento, acidente, congestionamento, ciclone |
| MÉDIO | 🟡 | lentidão, desvio, obra, chuva, alerta |
| BAIXO | 🟢 | demais notícias relacionadas |

### Tipos de Incidente

- **Manifestação** — protestos, greves, paralisações
- **Acidente** — colisões, tombamentos, capotamentos
- **Evento Climático** — alagamentos, enchentes, deslizamentos
- **Interdição** — bloqueios e fechamentos de vias
- **Incêndio** — queimadas e incêndios em vias
- **Obra** — manutenção e obras em rodovias

### Regiões Monitoradas

O sistema identifica automaticamente mais de **35 localidades**, incluindo:

- Rodovias federais: BR-101, BR-116, BR-153, BR-163, BR-364, BR-381, BR-376, BR-277, BR-290 e outras
- Capitais: São Paulo, Rio de Janeiro, Belo Horizonte, Curitiba, Porto Alegre, Salvador, Fortaleza, Recife, Manaus, Belém, Brasília, Goiânia e outras
- Regiões específicas: Marginal Tietê, Via Dutra, Rodovia Bandeirantes, Castello Branco, ABC Paulista, Guarulhos, Santos e outras

---

## 📲 Exemplo de Alerta

```
🗺️ RELATÓRIO OPERACIONAL — BRASIL
📅 21/05/2026 às 08:30
━━━━━━━━━━━━━━━━━━━━

🔴 CRÍTICO — 3 ocorrências
  ├ PR — BR-376 — Evento Climático
  ├ SC — Florianópolis — Evento Climático
  └ MG — BR-116 — Manifestação

🟠 ALTO — 5 ocorrências
  ├ SP — Via Dutra — Acidente +2
  ├ DF — Brasília — Manifestação
  └ BA — BR-101 — Interdição

🟡 MÉDIO — 8 ocorrências
  ├ SP — São Paulo — Obra +3
  └ RS — Porto Alegre — Evento Climático +2

━━━━━━━━━━━━━━━━━━━━
📊 Total hoje: 16 incidentes
⏱️ Próxima atualização: 09:00
Fonte: Google News Brasil
```

---

## 📦 Pré-requisitos

- [n8n](https://n8n.io) — self-hosted ou cloud (gratuito para uso pessoal)
- Conta Google — para Google Sheets (OAuth2)
- Bot Telegram — criado via [@BotFather](https://t.me/BotFather)
- Canal ou grupo no Telegram — onde o bot é administrador

---

## 🚀 Configuração Passo a Passo

### 1. Criar o Bot no Telegram

1. Acesse [@BotFather](https://t.me/BotFather) no Telegram
2. Envie `/newbot` e siga as instruções
3. Guarde o **token** gerado (ex: `7123456789:AAFxxx...`)
4. Crie um canal no Telegram e adicione o bot como **administrador**
5. Obtenha o Chat ID do canal acessando no navegador:
```
https://api.telegram.org/bot<SEU_TOKEN>/getUpdates
```
6. Guarde o `"id"` do canal (número negativo, ex: `-1003911792440`)

### 2. Criar a Planilha no Google Sheets

1. Acesse [sheets.google.com](https://sheets.google.com) e crie uma nova planilha
2. Renomeie a aba para **`Incidentes`**
3. Adicione estes cabeçalhos na linha 1:

```
timestamp | data_hora | titulo | tipo | severidade | regiao | uf_estado | 
latitude | longitude | maps_link | fonte | link | descricao | alerta_enviado
```

4. Copie o **ID da planilha** da URL:
```
https://docs.google.com/spreadsheets/d/[ESTE_É_O_ID]/edit
```

### 3. Importar o Workflow no n8n

1. Acesse seu n8n → **Create new workflow**
2. Clique em **⋮ (três pontos)** → **Import from File**
3. Selecione o arquivo `n8n_monitor_BRASIL_MVP_final.json`

### 4. Configurar as Credenciais

**Telegram:**
1. Clique no nó **📲 Telegram: Relatório**
2. Em *Credencial* → **+ Create New**
3. Cole o token do BotFather no campo **Access Token**
4. No campo **Chat ID** → cole o ID do canal (ex: `-1003911792440`)

**Google Sheets:**
1. Clique no nó **📊 Google Sheets: Salvar**
2. Em *Credencial* → **+ Create New** → OAuth2
3. Faça login com sua conta Google e autorize
4. No campo **Document ID** → cole o ID da sua planilha

### 5. Ativar o Workflow

1. Clique em **Save** (Ctrl+S)
2. Clique no toggle **Inactive → Active**
3. Teste clicando em **▶ Test workflow**

---

## 🗂️ Estrutura de Dados

Cada incidente é salvo no Google Sheets com os seguintes campos:

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `timestamp` | DateTime | Data/hora de publicação da notícia |
| `data_hora` | String | Data/hora formatada em pt-BR |
| `titulo` | String | Título da notícia |
| `tipo` | String | Categoria do incidente |
| `severidade` | String | CRÍTICO / ALTO / MÉDIO / BAIXO |
| `regiao` | String | Rodovia, cidade ou região identificada |
| `uf_estado` | String | Sigla do estado (ex: SP, RJ, MG) |
| `latitude` | Float | Coordenada geográfica |
| `longitude` | Float | Coordenada geográfica |
| `maps_link` | URL | Link direto para Google Maps |
| `fonte` | String | Origem da informação |
| `link` | URL | Link para a notícia original |
| `descricao` | String | Resumo da notícia |
| `alerta_enviado` | Boolean | Se gerou alerta no Telegram |

---

## 🔮 Próximos Passos

- [ ] Integrar **HERE Traffic API** para dados de tráfego em tempo real (2-5 min de atraso)
- [ ] Integrar **TomTom Traffic API** para redundância em regiões específicas
- [ ] Adicionar monitoramento de **grupos de Telegram** de caminhoneiros
- [ ] Criar **dashboard no Looker Studio** conectado ao Google Sheets com mapa de incidentes
- [ ] Implementar **filtro por rotas específicas** da operação
- [ ] Adicionar **webhook** para integração com sistemas TMS/WMS

---

## 🛠️ Tecnologias Utilizadas

| Tecnologia | Função |
|-----------|--------|
| [n8n](https://n8n.io) | Plataforma de automação de workflows |
| Google News RSS | Coleta de notícias em tempo real |
| [INMET](https://portal.inmet.gov.br) | Alertas climáticos oficiais do Brasil |
| Telegram Bot API | Canal de alertas operacionais |
| Google Sheets | Base de dados e histórico de incidentes |
| JavaScript (n8n Code Node) | Lógica de classificação e processamento |

---

## 👤 Autor

Desenvolvido por **Adi** — Gestor de Operações Logísticas  
Projeto construído como parte do aprendizado em automação, dados e inteligência operacional.

---

## 📄 Licença

MIT License — fique à vontade para usar, modificar e distribuir.
