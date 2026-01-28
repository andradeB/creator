# Task: 03_phase_3_structure
**Fase:** 3 - Estruturação (The Brain)
**Input:** JSON Bruto | **Output:** Roteiro Estruturado (Schema Validado)
**Dependência:** Ollama (Llama 3.1)

## 1. Contexto e Referências
Transformar texto falado (bagunçado) em texto visual (estruturado). Aqui usamos LLM para atuar como "Editor".
* **Ler instrução:** `instructions/04_SPEC_PHASE_03.md`

## 2. Princípios de Engenharia
* **Strict Typing:** Não confie na saída do LLM. Use `Pydantic` para garantir que o JSON tem exatamente os campos necessários. Se não tiver, é considerado erro.
* **Self-Healing (Retry):** LLMs falham. O código deve ter um loop que tenta corrigir erros de parse automaticamente (ex: reenviar o erro para o LLM pedindo correção).
* **Prompt Engineering:** O prompt do sistema deve ser explícito sobre o formato de saída ("Responda APENAS JSON").

## 3. Instruções de Implementação
Peça para o Copilot gerar o arquivo `src/pipeline/step_03_refine.py` com a classe `ScriptArchitect`:

### A. Definição do Schema (Pydantic)
* Defina modelos para:
    * `VisualContent`: (headline, code_snippet, etc.)
    * `Segment`: (id, type [title_screen, code, concept], voiceover_text, visual_content)
    * `Script`: (meta, segments)

### B. Integração com Ollama
* Use a lib `ollama`.
* Crie um método que lê o texto completo da transcrição da Fase 2.

### C. Lógica de Refinamento
* Envie o texto para o `llama3.1` com um System Prompt focado em Design Instrucional.
* Instrua o modelo a limpar o texto falado (remover "hmmm", "tá", "né").
* O modelo deve dividir o conteúdo em slides visuais.

### D. Validação e Loop de Retry
* Receba a string do LLM.
* Tente: `Script.model_validate_json(response)`.
* `Except ValidationError`: Capture o erro, incremente um contador de tentativas, e chame o LLM novamente: "Seu JSON estava inválido aqui: {erro}. Corrija."
* Limite a 3 tentativas. Se falhar, levante erro crítico.

## 4. Definition of Done (DoD)
- [ ] O script gera um arquivo JSON limpo em `data/03`.
- [ ] O JSON passa na validação do Pydantic.
- [ ] O texto `voiceover_text` está fluido e sem vícios de linguagem.
