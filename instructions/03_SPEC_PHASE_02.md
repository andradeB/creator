# 03_SPEC_PHASE_02.md

**Projeto:** Mind-to-Script (M2S)
**Fase 2:** Transcrição Neural (Speech-to-Text)
**Status:** Especificação Técnica
**Hardware Target:** Apple Silicon M4 (Uso obrigatório de MLX/NPU)
**Dependência Crítica:** `mlx-whisper` (Python Lib)

---

## 1. Contexto e Objetivo
Nesta fase, transformamos o áudio bruto (WAV) em dados estruturados (JSON). O objetivo não é apenas ter o texto, mas ter os **timestamps precisos** (tempo de início e fim) de cada frase e palavra.

Como estamos operando em um MacBook M4, **é proibido** usar a implementação padrão do Whisper (`openai-whisper`) que roda lenta na CPU. Devemos usar a biblioteca `mlx-whisper` da Apple, que é otimizada para a arquitetura de memória unificada, permitindo transcrições do modelo `Large-v3` em velocidade superior ao tempo real.

---

## 2. Estrutura de Diretórios e I/O

O script deve operar estritamente nestes caminhos:

* **Diretório de Entrada (Input):** `data/01_audio_extracted/`
    * *Arquivo esperado:* `.wav` (Gerado na Fase 1)
* **Diretório de Saída (Output):** `data/02_transcriptions/`
    * *Formato gerado:* `.json` (UTF-8)
    * *Nomenclatura:* O arquivo de saída deve ter o mesmo nome base do input.
        * *Exemplo:* `Aula_01_Intro.wav` -> `Aula_01_Intro.json`

---

## 3. Especificação Técnica do MLX-Whisper

### 3.1 Modelo Escolhido
* **Modelo:** `mlx-community/whisper-large-v3-mlx` (ou simplesmente `large-v3` se configurado via MLX).
* **Precisão:** 4-bit quantization (Recomendado para economizar memória sem perda perceptível de qualidade) ou FP16.

### 3.2 Configuração da Transcrição
O código Python deve invocar o `mlx_whisper` com os seguintes parâmetros obrigatórios:
* `word_timestamps=True`: Fundamental para sincronizar a animação do vídeo posteriormente.
* `language="pt"`: Forçar português (ou detectar automaticamente, mas preferencialmente fixar para evitar trocas no meio).

### 3.3 Schema do JSON de Saída
O arquivo JSON salvo deve conter o dump completo do resultado do Whisper. A estrutura mínima esperada é:

```json
{
  "text": "Texto completo da transcrição...",
  "segments": [
    {
      "id": 0,
      "start": 0.0,
      "end": 4.5,
      "text": "Olá pessoal, bem-vindos a mais uma aula.",
      "words": [
        {"word": "Olá", "start": 0.0, "end": 0.5},
        {"word": "pessoal", "start": 0.6, "end": 1.0}
      ]
    }
  ]
}
```

---

## 4. Requisitos Funcionais do Script (`src/pipeline/step_02_transcribe.py`)

### 4.1. Gerenciamento de Dependências
* O script deve importar `mlx_whisper`. Se falhar, deve alertar o usuário para instalar:
  `pip install mlx-whisper`

### 4.2. Classe `Transcriber`
Implementar uma classe que gerencie o estado do modelo. O modelo deve ser carregado na memória apenas uma vez e reutilizado se houver múltiplos arquivos na fila.

### 4.3. Lógica de Caching
* **Regra:** Se `arquivo.json` já existe na pasta `02_transcriptions` e não está vazio (size > 0), pular o processamento.
* Isso é crucial pois a transcrição é a etapa mais demorada (mesmo no M4).

### 4.4. Tratamento de Arquivos Longos
* O MLX Whisper lida bem com arquivos longos, mas o script deve garantir que não haja timeout.
* Logs de progresso ("Transcribing... this may take a while") devem ser exibidos no terminal.

---

## 5. Critérios de Aceite (Definition of Done)

1.  [ ] O script `src/pipeline/step_02_transcribe.py` executa sem erros de importação.
2.  [ ] O modelo `large-v3` é baixado/carregado corretamente na primeira execução.
3.  [ ] Um arquivo `.json` válido é gerado na pasta `02_transcriptions` para cada `.wav` encontrado na pasta `01`.
4.  [ ] O JSON contém a chave `segments` com timestamps.
5.  [ ] O Activity Monitor (Monitor de Atividade) do Mac mostra uso de GPU/Neural Engine durante a execução (indica que o MLX está funcionando).

---

## 6. Prompt para Implementação (Copiar para IA)

> "Atue como um Engenheiro de IA Especialista em Apple Silicon. Implemente o módulo `src/pipeline/step_02_transcribe.py` utilizando a biblioteca `mlx-whisper`. O script deve ler arquivos `.wav` da pasta `data/01_audio_extracted`, transcrever usando o modelo `large-v3` com `word_timestamps=True`, e salvar o resultado em JSON na pasta `data/02_transcriptions`. Implemente verificação de cache para não reprocessar arquivos existentes. Inclua tratamento de exceções e logs claros."
