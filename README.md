# Release-automatic
Automação de release notes no slack e no gitbook
# Release Automation — Amigo Flow

Automação que roda toda **quarta e sexta** para:
1. Buscar histórias concluídas (`Done`) no Jira
2. Gerar documentação com Claude (Anthropic)
3. Criar página no GitBook
4. Publicar comunicado de release notes no Slack

---

## Estrutura

```
release-automation/
├── .github/
│   └── workflows/
│       └── release.yml          # Agendamento e execução no GitHub Actions
├── scripts/
│   ├── release_automation.py    # Script principal
│   └── published_issues.json    # Controle de duplicatas (commitado automaticamente)
└── README.md
```

---

## Configuração — GitHub Secrets

Vá em **Settings → Secrets and variables → Actions → New repository secret** e adicione:

| Secret | Descrição | Exemplo |
|--------|-----------|---------|
| `JIRA_DOMAIN` | Domínio Atlassian | `amigotech.atlassian.net` |
| `JIRA_EMAIL` | E-mail da conta Jira | `geo@amigotech.com` |
| `JIRA_TOKEN` | API Token do Jira | Gerado em [id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens) |
| `JIRA_PROJECT` | Chave do projeto | `GREEN` |
| `JIRA_STATUS` | Status de conclusão (exato) | `Done` |
| `JIRA_DAYS` | Janela de busca em dias | `4` (cobre qua→sex e sex→qua) |
| `ANTHROPIC_API_KEY` | Chave da API Anthropic | `sk-ant-...` |
| `GITBOOK_TOKEN` | API Token do GitBook | Gerado em GitBook → Settings → Developer |
| `GITBOOK_SPACE_ID` | ID do Space de destino | `space_xxxx` |
| `GITBOOK_PARENT_PAGE_ID` | *(opcional)* ID da página-pai | `page_xxxx` |
| `SLACK_BOT_TOKEN` | Bot Token do Slack | `xoxb-...` |
| `SLACK_CHANNEL_ID` | ID do canal de destino | `C0XXXXXXXXX` |

---

## Como obter cada credencial

### Jira API Token
1. Acesse [id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens)
2. Clique em **Create API token**
3. Copie o token gerado

### GitBook Token
1. No GitBook, vá em **Settings → Developer → API Tokens**
2. Crie um token com permissão de escrita no Space desejado
3. O Space ID aparece na URL: `app.gitbook.com/o/ORG/s/SPACE_ID/`

### Slack Bot Token
1. Acesse [api.slack.com/apps](https://api.slack.com/apps) → **Create New App**
2. Em **OAuth & Permissions**, adicione os escopos: `chat:write`
3. Instale o app no workspace e copie o **Bot User OAuth Token** (`xoxb-...`)
4. Adicione o bot ao canal desejado: `/invite @nome-do-bot`
5. O Channel ID aparece na URL do canal no Slack Web

---

## Ajustando os horários

O arquivo `.github/workflows/release.yml` usa cron no fuso UTC.  
Para rodar às **10h de Brasília (UTC-3)**, o cron usa `13` no campo de hora:

```yaml
- cron: "0 13 * * 3"   # quarta-feira
- cron: "0 13 * * 5"   # sexta-feira
```

Para mudar o horário: `"MINUTO HORA_UTC * * DIA_SEMANA"` (0=dom, 1=seg ... 5=sex, 6=sáb)

---

## Controle de duplicatas

O arquivo `scripts/published_issues.json` guarda as chaves Jira já processadas.  
O GitHub Actions faz commit automático desse arquivo após cada execução, evitando reprocessamento.

---

## Rodando manualmente

Na aba **Actions** do repositório → **Release Automation — Amigo Flow** → **Run workflow**.

---

## Personalizando os prompts

Os prompts de geração de conteúdo ficam no próprio `scripts/release_automation.py`, nas variáveis `SYSTEM_GITBOOK` e `SYSTEM_SLACK`. Edite diretamente no repositório para ajustar tom, estrutura ou contexto.
