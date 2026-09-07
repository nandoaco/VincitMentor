# RUBRICA DE AVALIAÇÃO — VINCITMENTOR

**Versão:** 1.0  
**Aplicação:** Avaliador Gemini (Fase 2 — Loop de Avaliação e Feedback)  
**Domínio Inicial:** Cybersecurity — Blue Team / SOC L1  

---

## 1. PROPÓSITO DO INSTRUMENTO
Este documento define o modelo dimensional pelo qual o VincitMentor avalia as submissões técnicas a incidentes e desafios táticos. O objetivo é eliminar avaliações genéricas baseadas apenas em acerto binário, transformando a IA em um avaliador com critério técnico profissional.

A rubrica opera dentro do nó Gemini no workflow de avaliação do n8n após submissão de relatório via interface.

---

## 2. PRINCÍPIO PEDAGÓGICO
> "Não fazer o aluno parecer competente. Torná-lo competente."

- Erro é instrumento de diagnóstico para direcionar a próxima atividade.
- Raciocínio articulado tem peso superior à memorização de gabaritos.
- A clareza sobre o que a evidência **não prova** é critério mandatório de maturidade operacional.

---

## 3. DIMENSÕES DE AVALIAÇÃO

| Dimensão | Peso | Foco de Avaliação |
| :--- | :---: | :--- |
| **1. Acurácia Técnica** | 30% | Identificação correta de campos (IP, porta, Event ID, usuário, status) e ausência de dados inventados. |
| **2. Raciocínio Investigativo** | 30% | Encadeamento lógico, correlação temporal de eventos e formulação de hipóteses técnicas. |
| **3. Decisão e Contenção** | 25% | Proporcionalidade das ações de contenção, respeito à alçada de SOC L1 e registro formal em ticket. |
| **4. Consciência de Limite** | 15% | Reconhecimento explícito das limitações do log apresentado, evitando suposições não comprovadas. |
| **TOTAL** | **100%** | Pontuação final dimensional. |

---

## 4. CRITÉRIOS DE PONTUAÇÃO E FAIXAS DE VEREDITO

- **85 – 100 (APROVADO):** Competência tática demonstrada com solidez e aderência total aos logs.
- **70 – 84 (APROVADO COM RESSALVAS):** Interpretação correta da ocorrência, demandando ajustes em terminologia, limites de evidência ou documentação de tickets.
- **50 – 69 (EM DESENVOLVIMENTO):** Leitura superficial ou lacunas na correlação temporal; exige prática direcionada.
- **0 – 49 (REQUER REVISÃO):** Resposta em branco, fuga total ao escopo ou premissas técnicas insustentáveis.

---

## 5. ESTRUTURA DE DADOS GERADA (SCHEMA JSON)
Toda avaliação produzida pelo motor cognitivo segue este schema estrito, sendo gravada na coluna `evaluation_json` (JSONB) da tabela `student_answers`:

```json
{
  "score_total": 82,
  "veredito": "APROVADO COM RESSALVAS",
  "dimensoes": {
    "acuracia_tecnica": { "nota": 28, "peso": 30, "comentario": "..." },
    "raciocinio_investigativo": { "nota": 24, "peso": 30, "comentario": "..." },
    "decisao_contencao": { "nota": 20, "peso": 25, "comentario": "..." },
    "consciencia_limite": { "nota": 10, "peso": 15, "comentario": "..." }
  },
  "pontos_fortes": ["..."],
  "pontos_fracos": ["..."],
  "feedback_mentor": "...",
  "sugestao_estudo": "...",
  "competencias_avaliadas": ["windows_event_log", "soc_l1_triage"],
  "gap_detectado": "logon_type_distinction",
  "proxima_atividade_recomendada": "..."
}