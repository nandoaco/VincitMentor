# VINCITMENTOR — CURRENT STATE

Última atualização: Setembro/2026

Fase atual: TRANSIÇÃO FASE 2 (LOOP DE AVALIAÇÃO CONCLUÍDO) → FASE 3 (GAPS & LEARNING ENGINE)

FUNCIONANDO:
- Pipeline de Geração (Fase 1): Gemini flash-lite → Code JS1 → Supabase (lessons) → Google Sheets (espelho) → Softr.
- Exibição de cenários SOC L1, logs brutos formatados (Windows Event Logs 4624/4625), desafios e filtros na interface Softr.
- Formulário customizado em HTML/JS no Lesson Detail extraindo recordId via query string de URL.
- Disparo de telemetria via HTTP POST para o Webhook (/avaliar-resposta) no n8n.
- Orquestração de ponta a ponta no n8n:
  - Supabase (Get a row): Recupera a aula correspondente usando recordId.
  - Google Gemini (Message a model): Avaliador analítico utilizando a Rubrica Dimensional V1.0.
  - Code (Normalize Evaluation): Parser robusto do JSON gerado pela IA com fallbacks de segurança.
  - Supabase (Create a row): Gravação completa do histórico em student_answers, incluindo o payload estruturado em evaluation_json (JSONB).
  - Supabase (Update a row): Atualização de status da aula para completed e prefixo ✅ [Concluído] no título.
  - Google Sheets (Update row in sheet): Atualização do espelho no Sheets sem duplicidade de linhas.
  - Respond to Webhook: Retorno formatado via HTTP para a interface.
- Frontend Softr renderizando em tempo real a pontuação (0-100), o veredito dimensional (ex: APROVADO COM RESSALVAS) e o parecer técnico do mentor.

ÚLTIMO MARCO HOMOLOGADO:
Loop completo de avaliação e feedback (Fase 2) validado em produção com nota 82/100, persistência relacional e atualização automática de status no banco.

PRÓXIMO PASSO EXATO (FASE 3 — GAPS & LEARNING ENGINE):
Criar a tabela knowledge_states no Supabase para mapear e persistir o nível de domínio das competências avaliadas e os gaps detectados, alimentando o motor de seleção da próxima atividade adaptativa.
