# AI Mentor — Sistema de Automação de Estudos

Sistema de automação construído em n8n que agenda sessões de estudo, registra o progresso em uma base de conhecimento e envia lembretes automáticos — projetado como base de portfólio para vagas de Cyber Security, Cloud e AI Engineering.

## O que o sistema faz

1. **Agenda a aula** — cria um evento no Google Calendar com título e horário.
2. **Registra o estudo** — salva automaticamente no Notion (título, data/hora, área e status), a partir dos dados reais gerados na etapa anterior.
3. **Envia o lembrete** — dispara uma mensagem no Telegram confirmando o agendamento, com os dados dinâmicos do evento.

## Arquitetura

```
Trigger > Google Calendar (cria evento) > Notion (registra estudo) >migrar para Infisical Telegram (envia alerta)
```

Cada etapa consome o output real da etapa anterior (sem valores fixos/hardcoded), usando expressões do n8n para passar os dados adiante.

## Ferramentas utilizadas

- **n8n** — orquestração do workflow
- **Google Calendar API** — agendamento (OAuth 2.0)
- **Notion API** — base de conhecimento / log de estudos
- **Telegram Bot API** — canal de notificação

## Gerenciamento de credenciais

Todas as credenciais (Google OAuth Client Secret, Notion Integration Token, Telegram Bot Token) ficam armazenadas de forma criptografada dentro do n8n, nunca hardcoded no workflow ou no repositório. O acesso ao Google Calendar segue o princípio de menor privilégio (escopo restrito a criação/edição de eventos, sem acesso de listagem/exclusão de agendas).

> Próximo passo planejado: migração das credenciais para o Google Secret Manager, com policies de IAM dedicadas.

## Como reproduzir

1. Importe o `workflow.json` no seu n8n (Cloud ou self-hosted).
2. Configure as credenciais de Google Calendar, Notion e Telegram (veja os requisitos de escopo/permissão de cada uma na documentação oficial de cada serviço).
3. Ajuste o ID da base do Notion e o Chat ID do Telegram para os seus próprios.
4. Execute o workflow.

## Roadmap

- [x] Migrar credenciais para Infisical
- [ ] Adicionar geração de conteúdo de aula via LLM (Claude) como "Professor IA"
- [ ] Trigger automático (agenda recorrente, em vez de manual)
- [ ] Expandir para o projeto RENOVIA

## Autor

Luis Fernando Gomes — [GitHub](https://github.com/nandoaco) · [LinkedIn](https://www.linkedin.com/in/luis-fernando-gomes-9b1531171/)