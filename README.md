<div align="center">

<img src="docs/images/logo-new.png" alt="Meus Prazos" width="180" />

# Meus Prazos

### Gestão unificada de prazos e tarefas de todas as suas plataformas

[![Laravel](https://img.shields.io/badge/Laravel-13-FF2D20?style=for-the-badge&logo=laravel&logoColor=white)](https://laravel.com)
[![PHP](https://img.shields.io/badge/PHP-8.5-777BB4?style=for-the-badge&logo=php&logoColor=white)](https://php.net)
[![Livewire](https://img.shields.io/badge/Livewire-3-FB70A9?style=for-the-badge&logo=livewire&logoColor=white)](https://livewire.laravel.com)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind-4-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white)](https://tailwindcss.com)
[![PWA](https://img.shields.io/badge/PWA-Ready-5A0FC8?style=for-the-badge&logo=pwa&logoColor=white)](#pwa--push-notifications)
[![PagarMe](https://img.shields.io/badge/PagarMe-API%20v5-00C853?style=for-the-badge&logo=stripe&logoColor=white)](#planos-e-pagamentos)

**Pare de alternar entre plataformas.** O Meus Prazos sincroniza suas tarefas do Google Calendar, Trello, Bitrix24, Canvas LMS, Notion e GitHub em um único dashboard dark com IA integrada e notificações push inteligentes.

🔗 **[meusprazos.rezumme.ai](https://meusprazos.rezumme.ai)**

> ⚠️ Este repositório contém apenas a documentação e apresentação do projeto. O código-fonte é mantido em repositório privado.

</div>

---

## O Problema

Quem trabalha, estuda e faz freela ao mesmo tempo tem prazos espalhados em **7 plataformas diferentes**:

| | Plataforma | Uso |
|----|------------|-----|
| <img src="docs/images/interations/calendar.png" width="20" /> | **Google Calendar** | Eventos pessoais e reuniões |
| <img src="docs/images/interations/trello.png" width="20" /> | **Trello** | Boards Scrum do trabalho |
| <img src="docs/images/interations/bitrix.png" width="20" /> | **Bitrix24** | Tarefas CRM e workflows |
| <img src="docs/images/interations/afya.png" width="20" /> | **Canvas LMS** | Trabalhos e avisos da faculdade |
| <img src="docs/images/interations/notion.png" width="20" /> | **Notion** | Bancos de dados pessoais |
| <img src="docs/images/interations/github.png" width="20" /> | **GitHub** | Issues e pull requests |
| <img src="docs/images/interations/discord.png" width="20" /> | **Discord** | Notificações de canal |

Ficar alternando entre abas pra saber "o que vence agora?" mata a produtividade. O **Meus Prazos** resolve isso trazendo tudo para um só lugar — automaticamente.

---

## Funcionalidades

### 🔄 Sincronização Multi-Plataforma
- **Sync automático** a cada 3 minutos via jobs em background (Laravel Horizon + Redis)
- **7 integrações**: Google Calendar, Trello, Bitrix24, Canvas LMS, Notion, GitHub, Discord
- Smart **upsert** — cria novas tarefas, atualiza existentes, marca removidas como concluídas
- **Logs de sincronização** completos com rastreamento de sucesso/falha por plataforma

### 📊 3 Visualizações no Dashboard
- **Feed** — Lista cronológica agrupada por tempo (hoje, amanhã, esta semana, depois)
- **Kanban Board** — Drag-and-drop por colunas: Pendente → Em Progresso → Concluído → Atrasado
- **Avisos** — Feed dedicado para anúncios do Canvas LMS

### 🤖 Chat com IA (Google Gemini)
- **Chat interativo** com Google Gemini integrado ao contexto das suas tarefas
- Anexe tarefas ao chat para análise e priorização inteligente
- Histórico de sessões persistente
- **Limite diário** no plano Free (5 msgs/dia), ilimitado no Pro
- Resumo semanal automático com insights de produtividade

### 🔔 Notificações Push Inteligentes (PWA)
- Instalável no celular — **sem app store**
- **Tarefas urgentes** (≤6h) recebem alertas individuais via Firebase Cloud Messaging
- **Tarefas próximas** (6–48h) são agrupadas em digest
- Smart dedup: cada tarefa é notificada apenas uma vez

### 💳 Planos e Pagamentos
- **Plano Free**: 5 mensagens IA/dia, sync manual
- **Plano Pro** (R$ 9,90/mês): IA ilimitada, sync automático, resumo semanal
- Integração com **PagarMe API v5** para assinaturas recorrentes
- Webhooks com filtragem por plataforma (metadata) — mesma conta PagarMe compartilhada entre múltiplos serviços
- Painel admin para conceder/revogar plano Pro manualmente

### 🛡️ Painel Administrativo
- Gerenciamento de usuários e planos
- Toggle Pro/Free com confirmação visual (SweetAlert2)
- Visão geral do sistema e logs

### 🎨 UI Dark com Glass-Morphism
- **Dark theme** moderno com efeitos glass-morphism
- Design responsivo — funciona em mobile e desktop
- Alertas bonitos com **SweetAlert2** (tema glassmorphism customizado)
- Saudação dinâmica baseada na hora do dia

---

## Arquitetura

```
┌──────────────────────────────────────────────────────────────┐
│                    VPS CloudPanel (Brasil)                   │
│                                                              │
│  ┌───────────────┐   ┌──────────┐   ┌───────────────┐       │
│  │  Laravel 13   │   │  Redis   │   │  PostgreSQL   │       │
│  │  PHP 8.5      │◄──│  Queue   │   │  16           │       │
│  │  Livewire 3   │   │  Cache   │   │  (data +      │       │
│  │  Horizon      │   └──────────┘   │   tokens)     │       │
│  └──────┬────────┘                  └───────────────┘       │
│         │                                                    │
│         ▼               Scheduler (cron) + PM2               │
│  ┌──────────────────────────────────────────────────┐       │
│  │           Background Jobs (Queue)                │       │
│  │  Google Calendar · Trello · Bitrix24 · Canvas    │       │
│  │  Notion · GitHub · Reminders · Weekly Summary    │       │
│  │  PagarMe Webhooks                               │       │
│  └──────────────────────┬───────────────────────────┘       │
└─────────────────────────┼────────────────────────────────────┘
                          │
        ┌─────────────────┼──────────────────┐
        ▼                 ▼                  ▼
  ┌───────────┐   ┌──────────────┐   ┌──────────────┐
  │ Composio  │   │  APIs REST   │   │   PagarMe    │
  │  OAuth    │   │  Bitrix24    │   │   API v5     │
  │ (Google,  │   │  Canvas LMS  │   │  (payments)  │
  │  Trello,  │   └──────────────┘   └──────────────┘
  │  Notion,  │
  │  GitHub)  │
  └───────────┘
```

### Fluxo de Dados

```
APIs Externas → Sync Jobs → TaskNormalizerService → Task Model (UUID)
                                                          │
                                    ┌─────────────────────┤
                                    ▼                     ▼
                              Livewire UI          FCM Push Alerts
                           (Feed/Board/Chat)      (via Firebase)

PagarMe Webhook → Filter by metadata.platform → ProcessPagarmeWebhookJob
                  (ignora webhooks de outros serviços)
```

---

## Tech Stack

| Camada | Tecnologia |
|--------|------------|
| **Backend** | Laravel 13 · PHP 8.5 |
| **Frontend** | Livewire 3 · Alpine.js · Tailwind CSS 4 |
| **Database** | PostgreSQL 16 |
| **Queue & Cache** | Redis · Laravel Horizon · PM2 |
| **Build** | Vite 8 |
| **Push Notifications** | Firebase Cloud Messaging (FCM) |
| **OAuth** | Composio SDK (Google Calendar, Trello, Notion, GitHub) |
| **APIs Diretas** | Bitrix24 (webhook), Canvas LMS (REST) |
| **IA** | Google Gemini (chat interativo + resumo semanal) |
| **Pagamentos** | PagarMe API v5 (assinaturas recorrentes + webhooks) |
| **Alertas UI** | SweetAlert2 (tema glassmorphism) |
| **Testes** | Pest 4 · PHPUnit 12 |
| **Infra** | VPS · CloudPanel · Brasil |

---

## Estrutura do Projeto

```
app/
├── Ai/
│   └── Agents/
│       └── WeeklySummaryAgent.php       # Agent Gemini para resumo semanal
├── Jobs/                                # 9 jobs de background
│   ├── SyncGoogleCalendarJob.php
│   ├── SyncTrelloJob.php
│   ├── SyncBitrixJob.php
│   ├── SyncCanvasJob.php
│   ├── SyncNotionJob.php
│   ├── SyncGithubJob.php
│   ├── SendDeadlineRemindersJob.php
│   ├── SendWeeklySummaryJob.php
│   └── ProcessPagarmeWebhookJob.php
├── Livewire/                            # 10 componentes reativos
│   ├── AdminDashboard.php
│   ├── AiChat.php
│   ├── TaskBoard.php
│   ├── TaskFeed.php
│   ├── TaskCrud.php
│   ├── AnnouncementFeed.php
│   ├── PlatformConnections.php
│   ├── NotificationSettings.php
│   ├── PinLogin.php
│   └── Register.php
├── Models/                              # 12 modelos
│   ├── User.php
│   ├── Task.php
│   ├── ChatSession.php
│   ├── ChatMessage.php
│   ├── AiSummary.php
│   ├── PlatformToken.php
│   ├── Subscription.php
│   ├── SubscriptionInvoice.php
│   ├── WebhookLog.php
│   ├── SyncLog.php
│   ├── SyncSetting.php
│   └── DeviceToken.php
├── Services/                            # 8 services
│   ├── GeminiService.php
│   ├── ComposioService.php
│   ├── PagarmeService.php
│   ├── TaskNormalizerService.php
│   ├── FcmService.php
│   ├── BitrixApiService.php
│   ├── CanvasApiService.php
│   └── DiscordNotificationService.php
├── Http/
│   ├── Controllers/
│   │   ├── Api/DeviceTokenController.php
│   │   ├── Webhooks/PagarmeWebhookController.php
│   │   └── OAuthCallbackController.php
│   └── Middleware/
│       ├── PinAuth.php
│       ├── Admin.php
│       └── VerifyPagarmeSignature.php
└── Enums/                               # 8 enums type-safe
    ├── Platform.php
    ├── TaskSource.php
    ├── TaskStatus.php
    ├── TaskPriority.php
    ├── SyncStatus.php
    ├── SubscriptionStatus.php
    ├── InvoiceStatus.php
    └── DeviceType.php
```

---

## PWA & Push Notifications

O app é um **Progressive Web App** totalmente instalável:

1. Abra o dashboard no navegador do celular
2. Toque em **"Instalar"** → ícone aparece na tela inicial
3. Autorize notificações → receba alertas de prazos em tempo real
4. Funciona offline com service worker

Sem App Store ou Play Store.

---

## Construído Com

Este projeto foi construído inteiramente com **GitHub Copilot Agent Mode** no VS Code, com os seguintes MCP servers auxiliando o desenvolvimento:

| MCP Server | Função |
|------------|--------|
| **GitHub** | Gerenciamento de repositório e commits |
| **Context7** | Documentação atualizada do Laravel 13 em tempo real |
| **Composio** | Integrações OAuth com plataformas externas |
| **Playwright** | Testes automatizados no navegador |
| **Filesystem** | Leitura/escrita de arquivos do projeto |

---

## Licença

Este projeto é proprietário. O código-fonte não está disponível publicamente.

---

<div align="center">

Desenvolvido por [Irving Samuel](https://github.com/IrvingSamuel)

</div>
