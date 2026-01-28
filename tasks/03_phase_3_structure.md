# Task: 03_phase_3_structure
**Fase:** 3 - Estruturação (The Brain)
**Input:** JSON Bruto | **Output:** Roteiro Estruturado (Schema Validado)
**Dependência:** Ollama (Llama 3.1)

## 1. Contexto e Referências
Transformar texto falado (bagunçado) em texto visual (estruturado). Aqui usamos LLM para atuar como "Editor".
* **Ler instrução:** `instructions/04_SPEC_PHASE_03.md`

## 2. Princípios de Engenharia
* **Strict Typing:** Use `Pydantic` para garantir o contrato de dados.
* **Defensive Parsing:** Nunca confie que a string do LLM virá limpa. Ela virá suja. Prepare o código para limpá-la.
* **Self-Healing:** Se falhar, tente corrigir automaticamente.

## 3. Instruções de Implementação
Peça para o Copilot gerar o arquivo `src/pipeline/step_03_refine.py`:

### A. Definição do Schema (Pydantic)
* Defina modelos para `VisualContent`, `Segment` e `Script` conforme a spec.

### B. Utilitário de Limpeza (JSON Sanitizer)
* Crie uma função privada `_clean_json_response(text: str) -> str`.
* Ela deve usar Regex para remover delimitadores de código Markdown (ex: ```json ... ```).
* Ela deve buscar o primeiro `{` e o último `}` da string para ignorar textos introdutórios do chat.

### C. Integração e Lógica
* Conecte ao Ollama (`llama3.1:8b`).
* System Prompt: "Atue como Designer Instrucional. Responda APENAS JSON. Responda em Português do Brasil".
* Fluxo:
    1. Enviar prompt.
    2. Receber texto sujo.
    3. Passar pelo `_clean_json_response`.
    4. Tentar `Script.model_validate_json()`.

### D. Loop de Retry
* Se a validação falhar, capture o erro e reenvie ao LLM: "Erro no JSON: {e}. Corrija mantendo o mesmo conteúdo." (Máx 3 tentativas).

## 4. Definition of Done (DoD)
- [ ] O script consegue ler uma resposta do Ollama contendo "Aqui está seu JSON: ```json {...}```" e extrair apenas o objeto válido sem quebrar.
- [ ] O JSON final é salvo e validado pelo Pydantic.