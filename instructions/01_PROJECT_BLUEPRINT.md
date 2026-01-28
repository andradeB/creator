# 01_PROJECT_BLUEPRINT.md

**Projeto:** Mind-to-Script (M2S) Engine
**Objetivo:** Pipeline de remasterização de vídeo aulas usando IA Generativa Local.
**Referência de Estrutura:** `00_PROJECT_STRUCTURE.md`

---

## 1. Visão Macro do Pipeline

O projeto segue um fluxo linear (Waterfall). Cada fase é bloqueante: a Fase 2 só começa se a Fase 1 gerar seus arquivos com sucesso na pasta correta.

### 🌊 Fluxo de Dados
`Vídeo Original` -> [Fase 1] -> `Áudio` -> [Fase 2] -> `Texto` -> [Fase 3] -> `Roteiro` -> [Fase 4] -> `Assets` -> [Fase 5] -> `Vídeo Final`

---

## 2. Detalhamento das Etapas

### 🟢 FASE 1: Ingestão e Extração
O objetivo é preparar a matéria-prima. O vídeo original é pesado demais para ser processado diretamente por modelos de áudio. Precisamos extrair o áudio em um formato leve e compatível com IA.

* **Input:** Pasta `data/00_raw_input/`
* **Output:** Pasta `data/01_audio_extracted/`
* **Engine:** FFmpeg (via Python Wrapper).
* **Artefato Crítico:** Arquivo `.wav` (16kHz, mono).
* **📄 Documento de Especificação Técnica:** `02_SPEC_PHASE_01.md`

---

### 🔵 FASE 2: Transcrição Neural (ASR)
Aqui transformamos ondas sonoras em dados de texto brutos. Usaremos a NPU do Apple Silicon para fazer isso em alta velocidade.

* **Input:** Pasta `data/01_audio_extracted/`
* **Output:** Pasta `data/02_transcriptions/`
* **Engine:** `mlx-whisper` (Modelo: Large-v3).
* **Artefato Crítico:** Arquivo `.json` contendo texto completo + *word-level timestamps*.
* **📄 Documento de Especificação Técnica:** `03_SPEC_PHASE_02.md`

---

### 🟣 FASE 3: A Mente (Estruturação LLM)
Esta é a etapa mais complexa. Uma LLM (Llama 3.1) vai ler o texto "sujo" da fase anterior, entender o conteúdo pedagógico e reescrever um roteiro estruturado para vídeo, eliminando erros do professor original.

* **Input:** Pasta `data/02_transcriptions/`
* **Output:** Pasta `data/03_structured_scripts/`
* **Engine:** Ollama (Local) + Python (Lógica de Validação e Retry).
* **Artefato Crítico:** `script.json` (Validado estritamente por Schema Pydantic).
* **📄 Documento de Especificação Técnica:** `04_SPEC_PHASE_03.md`

---

### 🟠 FASE 4: Fábrica de Assets (TTS & GenAI)
Com o roteiro em mãos, precisamos gerar os elementos multimídia. O sistema vai clonar a voz do professor para ler o novo roteiro limpo e calcular exatamente quanto tempo cada slide deve durar.

* **Input:** Pasta `data/03_structured_scripts/`
* **Output:** Pasta `data/04_assets_generated/`
* **Engine:** GPT-SoVITS (Áudio) + Librosa (Análise Temporal).
* **Artefato Crítico:**
    1.  Arquivos `.wav` para cada frase do roteiro.
    2.  `enriched_script.json` (O JSON original acrescido de `durationInFrames` para sincronia).
* **📄 Documento de Especificação Técnica:** `05_SPEC_PHASE_04.md`

---

### 🔴 FASE 5: Renderização e Montagem
O "Load" do ETL. O motor de vídeo lê o roteiro enriquecido e monta os componentes visuais (React) em sincronia com os áudios gerados.

* **Input:** Pasta `data/04_assets_generated/`
* **Output:** Pasta `data/05_final_output/`
* **Engine:** Remotion (Node.js/React) comandado via CLI.
* **Artefato Crítico:** Arquivo `.mp4` final (H.264 ou ProRes).
* **📄 Documento de Especificação Técnica:** `06_SPEC_PHASE_05.md`

---

## 3. Orquestração

O arquivo `src/main.py` não conterá lógica de negócio. Ele será apenas um gerenciador de estado que:
1.  Verifica se a Fase anterior gerou os arquivos necessários.
2.  Chama o script da Fase atual.
3.  Para a execução imediatamente em caso de erro (Fail Fast).

---

## 4. Dicionário de Conceitos e Decisões de Engenharia

Esta seção explica o "Porquê" das especificações técnicas, garantindo que a intenção do projeto seja mantida em futuras manutenções.

### 4.1. O Padrão de Áudio (WAV 16kHz Mono)
Na **Fase 1**, não estamos apenas "convertendo vídeo". Estamos criando o alimento ideal para a IA.
* **WAV (PCM s16le):** É o áudio "RAW" (cru). Diferente do MP3, que joga dados fora para comprimir (lossy), o WAV mantém a integridade matemática da onda sonora. Isso evita que a IA "alucine" palavras devido a artefatos de compressão.
* **16kHz (Sample Rate):** A voz humana raramente passa de 8kHz. Pelo Teorema de Nyquist, 16kHz é a frequência perfeita para capturar a voz com clareza total sem desperdiçar processamento. O modelo Whisper é nativo nessa frequência.
* **Mono (1 Canal):** A IA não precisa de "palco sonoro" (estéreo). Remover o segundo canal corta o tamanho do arquivo pela metade e dobra a velocidade de ingestão na NPU.

### 4.2. A Transcrição Temporal (Time-Aligned Data)
Na **Fase 2**, o objetivo não é apenas o texto. O valor real são os **Timestamps**.
* O JSON gerado atua como um mapa: *"A palavra X foi dita exatamente no milissegundo Y"*.
* Sem essa precisão temporal (garantida pela flag `word_timestamps=True` no MLX), seria impossível sincronizar o vídeo final na Fase 5.

### 4.3. O "Diretor" vs. O "Editor" (LLM vs. Code)
* **Fase 3 (LLM):** Atua como o **Roteirista/Editor**. Ele toma decisões criativas e subjetivas (limpar o texto, decidir o que é um slide de título e o que é código). É probabilístico.
* **Fase 5 (Remotion):** Atua como a **Equipe de Filmagem**. É determinístico. Ele não "pensa", ele apenas executa cegamente as ordens dadas pelo JSON. Se o JSON disser "dure 50 frames", ele durará exatos 50 frames.

### 4.4. A Ponte Crítica (Fase 4 - Assets)
Esta é a etapa mais frequentemente negligenciada em pipelines de vídeo.
* Como o vídeo é gerado via código (Fase 5), ele não sabe "ouvir" quando o áudio acaba.
* A **Fase 4** é obrigatória para calcular matematicamente a duração (`durationInFrames = segundos * 30fps`). Ela transforma a *duração do som* em *espaço na timeline de vídeo*.