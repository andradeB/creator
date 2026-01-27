# 04_SPEC_PHASE_03.md

**Projeto:** Mind-to-Script (M2S)
**Fase 3:** Refinamento e Estruturação (The Brain)
**Status:** Especificação Técnica
**Hardware Target:** Apple Silicon M4 (Ollama com Metal Backend)
**Dependência Crítica:** `ollama` (Service), `pydantic` (Validation)

---

## 1. Contexto e Objetivo
A transcrição da Fase 2, embora precisa, contém vícios de linguagem, pausas e falta de estrutura visual. O objetivo desta fase é usar um LLM (Llama 3.1) para atuar como um "Editor de Vídeo Inteligente".

Ele deve ler o texto bruto e gerar um **Roteiro de Produção JSON**. Este roteiro define não apenas o que será dito (texto limpo), mas **o que será mostrado na tela** (slides, códigos, títulos) em cada momento.

---

## 2. Estrutura de Diretórios e I/O

O script deve operar estritamente nestes caminhos:

* **Diretório de Entrada (Input):** `data/02_transcriptions/`
    * *Arquivo esperado:* `.json` (Output do Whisper com `text` completo).
* **Diretório de Saída (Output):** `data/03_structured_scripts/`
    * *Formato gerado:* `.json` (Validado pelo schema abaixo).
    * *Nomenclatura:* Mesmo nome base do input.
        * *Exemplo:* `Aula_01_Intro.json` -> `Aula_01_Intro_Script.json`

---

## 3. Especificação do Prompt e Modelo (Ollama)

### 3.1 Configuração do Modelo
* **Modelo:** `llama3.1:8b` (Excelente balanço entre raciocínio e velocidade no M4).
* **Parâmetros:** `temperature=0.2` (Baixa criatividade, alta precisão estrutural).
* **Context Window:** O script deve verificar o tamanho do texto. Se exceder o contexto (ex: 8k tokens), deve-se usar uma estratégia de *chunking* ou garantir que o modelo suporte janelas maiores (Llama 3.1 suporta até 128k, então geralmente é seguro passar o texto inteiro).

### 3.2 System Prompt (A "Persona")
O prompt do sistema deve ser rigoroso quanto ao formato JSON. Exemplo de diretriz:
*"Você é um Especialista em Design Instrucional. Sua tarefa é converter transcrições brutas em roteiros de vídeo-aula altamente estruturados. Você deve limpar a fala, remover redundâncias e dividir o conteúdo em segmentos visuais claros (Intro, Conceito, Código, Resumo). Responda APENAS com o JSON válido."*

---

## 4. Schema JSON Obrigatório (O "Contrato")

O LLM deve gerar (e o Python deve validar) a seguinte estrutura. Recomenda-se usar `Pydantic` para garantir a tipagem.

```json
{
  "meta": {
    "title": "Título inferido da aula",
    "topic": "Tópico principal (ex: Python Basics)"
  },
  "segments": [
    {
      "id": "seg_01",
      "type": "title_screen",
      "visual_content": {
        "headline": "Introdução ao Python",
        "subheadline": "Conceitos Básicos"
      },
      "voiceover_text": "Olá e bem-vindos. Hoje vamos aprender os fundamentos da linguagem Python."
    },
    {
      "id": "seg_02",
      "type": "code_block",
      "visual_content": {
        "language": "python",
        "code_snippet": "print('Hello World')"
      },
      "voiceover_text": "A função print é a mais básica. Ela escreve uma mensagem no terminal."
    }
  ]
}
```

**Tipos de Segmentos Permitidos:**
1.  `title_screen` (Apenas texto grande)
2.  `bullet_points` (Lista de tópicos)
3.  `code_block` (Snippet de código)
4.  `concept_explanation` (Texto + Ícone/Imagem simples)

---

## 5. Requisitos Funcionais do Script (`src/pipeline/step_03_refine.py`)

### 5.1. Integração com Ollama
* Usar a biblioteca `ollama` para Python (`pip install ollama`) ou chamadas HTTP locais (`localhost:11434`).
* Verificar se o serviço Ollama está rodando antes de começar (`ollama list`).

### 5.2. Validação e Retry (Self-Correction)
* **Problema Comum:** LLMs às vezes colocam texto antes ou depois do JSON (ex: "Aqui está seu JSON: ...").
* **Solução:** O script deve tentar fazer o parse do JSON.
    * *Sucesso:* Salva o arquivo.
    * *Erro:* Tenta limpar a string (encontrar o primeiro `{` e o último `}`) e validar novamente.
    * *Erro Persistente:* Faz uma nova chamada ao LLM passando o erro e pedindo correção ("Você gerou um JSON inválido, corrija: [erro]").

### 5.3. Cache Inteligente
* Se o arquivo de saída existe, não chame o LLM novamente (economiza tempo e energia).

---

## 6. Critérios de Aceite (Definition of Done)

1.  [ ] O script `step_03_refine.py` lê o JSON bruto da Fase 2.
2.  [ ] O Ollama processa o texto e retorna uma estrutura.
3.  [ ] O JSON de saída passa na validação do Schema (campos obrigatórios presentes).
4.  [ ] O campo `voiceover_text` está limpo (sem "hmmm", "ééé", "tipo assim").
5.  [ ] Arquivos gerados são salvos em `data/03_structured_scripts/`.

---

## 7. Prompt para Implementação (Copiar para IA)

> "Atue como um Engenheiro de Backend Python. Implemente o módulo `src/pipeline/step_03_refine.py`. Use a biblioteca `ollama` para processar transcrições. Defina modelos `Pydantic` para validar a estrutura do JSON de saída (Meta e Segmentos). Implemente uma lógica robusta que tenta extrair JSON válido da resposta do LLM e, em caso de falha, realiza até 2 tentativas de retry. O prompt deve instruir o modelo a limpar o texto para um formato didático."
