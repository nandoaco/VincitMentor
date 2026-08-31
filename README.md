# VincitMentor — Plataforma de Aprendizagem Adaptativa com IA

O VincitMentor é uma plataforma de aprendizagem adaptativa orientada por evidências. O sistema identifica o nível e as necessidades do aluno, entrega experiências de aprendizagem, avalia sua capacidade de aplicar o conhecimento e utiliza os resultados para orientar a próxima atividade.

O primeiro domínio de validação é **Cybersecurity** — especialmente Blue Team e SOC — e o primeiro usuário é o próprio criador do projeto. A arquitetura, no entanto, está sendo projetada para que o motor pedagógico possa posteriormente ser aplicado a outras áreas de conhecimento.

---

## Premissa do projeto

Plataformas tradicionais assumem que consumo de conteúdo equivale a aprendizado. Em operações de segurança, o critério real é outro: interpretar log bruto, formular hipótese, validar evidência e decidir a contenção.

O VincitMentor inverte a lógica de EAD tradicional (modelo *Educação Push*): o sistema define a próxima atividade, exige demonstração prática e usa o erro como dado de diagnóstico — não como reprovação.

---

## O que o sistema faz

1. **Gera a aula** — o Gemini recebe uma Diretriz Mestre pedagógica e o tema, e devolve JSON estruturado (conceito, contexto MITRE ATT&CK, prática guiada, desafio com log bruto, raciocínio de analista, exercício, gabarito e critérios de avaliação).
2. **Normaliza o conteúdo** — um nó Code converte o JSON aninhado em Markdown padronizado, com fallbacks que impedem campos vazios ou `undefined` de chegarem ao banco.
3. **Persiste no Supabase** — a tabela `lessons` é a fonte da verdade do sistema.
4. **Espelha para consumo do frontend** — uma linha é replicada no Google Sheets, que alimenta o Softr durante o MVP.
5. **Entrega ao aluno** — Home com cards, filtro por nível e página de detalhe da aula com conteúdo integral e evidências.
6. **Coleta a resposta do exercício** — formulário embutido na página da aula envia `recordId` + resposta ao n8n via webhook (POST/JSON), armazenando em `student_answers`.

---

## Arquitetura

```text
[Aluno]
   ↓
Softr (frontend)
   ↓
Supabase (fonte da verdade)
   ↓
n8n (orquestração)
   ↓
Google Gemini (geração e avaliação)
   ↓
Google Sheets (espelho de leitura — camada temporária do MVP)
```

> **Decisão arquitetural:** Supabase é a fonte da verdade; Google Sheets é apenas espelho de leitura, previsto para remoção quando o frontend consumir o banco diretamente. Nenhuma regra de negócio depende da planilha.

### Workflow 1 — Geração de aula (ativo)

```text
Trigger > Code (tema) > Gemini (Diretriz Mestre) > Code (normalização JSON→Markdown)
        > Supabase (lessons) > Google Sheets (espelho)
```

### Workflow 2 — Avaliação da resposta (em construção)

```text
Softr (form) > Webhook n8n > Supabase (busca gabarito e critérios)
             > Gemini (avaliação com rubrica) > Supabase (student_answers) > feedback
```

Cada etapa consome o output real da etapa anterior, sem valores hardcoded, usando expressões do n8n com fallback explícito para campos ausentes.

---

## Modelo pedagógico

Toda aula gerada segue o ciclo obrigatório definido na Diretriz Mestre:

```text
CONCEITO → PRÁTICA GUIADA → DESAFIO → RACIOCÍNIO DE ANALISTA → EVIDÊNCIA → EXERCÍCIO
```

Regras aplicadas na geração:

- O desafio inclui log bruto real (IIS W3C, Sysmon, auth.log, headers SMTP) para análise.
- O gabarito não é exposto no corpo da aula — permanece em campo separado, de uso do avaliador.
- O exercício exige triagem de IOCs, veredito de impacto fundamentado em campo do log e plano de contenção.
- O conteúdo é vinculado ao MITRE ATT&CK (tática, técnica, ID) quando aplicável.

---

## Modelo de dados (Supabase)

| Tabela | Função |
|---|---|
| `lessons` | Aula gerada: título, tópico, dificuldade, conteúdo, exercício, gabarito, critérios |
| `student_answers` | Resposta do aluno, nota, feedback, pontos fortes e lacunas identificadas |
| `profiles` | Perfil e nível atual do aluno |
| `learning_paths` | Trilha e posição atual |
| `knowledge_states` | Grau estimado de domínio por competência |

Contrato de dados padronizado entre Gemini → Code → Supabase → Sheets → Softr, para evitar divergência de schema entre camadas.

---

## Ferramentas utilizadas

| Camada | Tecnologia | Função |
|---|---|---|
| Orquestração | n8n (self-hosted, Pikapods) | Execução dos workflows |
| IA | Google Gemini (Flash / Flash-Lite) | Geração de aula e avaliação de resposta |
| Banco | Supabase (PostgreSQL) | Fonte da verdade |
| Espelho | Google Sheets API (Service Account) | Camada de leitura do MVP |
| Frontend | Softr | Interface do aluno |
| Versionamento | GitHub | Workflows e documentação |

---

## Gerenciamento de credenciais

Todas as credenciais (Google Gemini API Key, Supabase Service Key, Service Account do Google Sheets) permanecem armazenadas de forma criptografada no n8n — nunca hardcoded no workflow, no frontend ou neste repositório. Os exports de workflow publicados aqui contêm apenas referências de credencial, sem segredos.

Controles aplicados:

- Autenticação multifator habilitada no painel do n8n.
- Service Account do Google com escopo restrito às APIs Sheets e Drive, e permissão concedida apenas na planilha específica do projeto (princípio do menor privilégio).
- Gabarito e critérios de avaliação não são renderizados no frontend do aluno.
- Webhook de avaliação recebe apenas identificador de registro e texto da resposta.

**Pendências de segurança já mapeadas para a fase de produção:** Row Level Security no Supabase, autenticação de usuário no frontend, sanitização de saída do modelo (XSS) e rotação de chaves.

---

## Estado atual

| Capacidade | Status |
|---|---|
| Geração de aula estruturada via IA | Concluído |
| Normalização e contrato de dados | Concluído |
| Persistência em Supabase | Concluído |
| Entrega ao aluno (Home + detalhe da aula) | Concluído |
| Log bruto e exercício prático na aula | Concluído |
| Envio da resposta do aluno para o n8n (webhook) | Concluído |
| Avaliação automatizada com rubrica | Em desenvolvimento |
| Motor de personalização por lacunas | Planejado |
| RAG sobre base documental própria | Planejado |
| Autenticação, dashboard e multiusuário | Planejado |

---

## Como reproduzir

1. Importe os arquivos de `workflows/` no seu n8n (Cloud ou self-hosted).
2. Configure as credenciais de Google Gemini, Supabase e Google Sheets (Service Account com acesso de editor à planilha).
3. Crie as tabelas do Supabase conforme `schema/` e ajuste os IDs de tabela nos nós.
4. Ajuste o endpoint do webhook no formulário do frontend para a URL do seu n8n.
5. Execute o workflow de geração e valide o registro no Supabase e no frontend.

---

## Roadmap

- [x] Pipeline de geração de aula ponta a ponta (IA → banco → frontend)
- [x] Aulas com evidência técnica real (log bruto) e exercício aplicado
- [x] Captura da resposta do aluno via webhook
- [x] Avaliação automatizada da resposta com rubrica e feedback estruturado
- [ ] Mapa de conhecimento e definição da próxima atividade por lacuna
- [ ] RAG com pgvector sobre documentação técnica própria
- [ ] Autenticação, RLS e suporte a múltiplos usuários
- [ ] Portfólio de competências com evidências verificáveis

> **Princípio de escopo adotado:** nenhuma tecnologia nova é incorporada antes de o ciclo *gerar → entregar → estudar → responder → avaliar → identificar lacuna → próxima aula* estar operacional.

---

## Autor

**Luis Fernando Gomes (Nando)** — Cyber Security
Concepção do produto, arquitetura da solução, orquestração dos workflows, engenharia de prompt pedagógica e validação ponta a ponta.

[GitHub](https://github.com/nandoaco) · [LinkedIn](https://www.linkedin.com/in/luis-fernando-gomes-9b1531171/)
