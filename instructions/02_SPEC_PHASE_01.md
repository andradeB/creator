# 02_SPEC_PHASE_01.md

**Projeto:** Mind-to-Script (M2S)
**Fase 1:** Ingestão de Vídeo e Extração de Áudio
**Status:** Especificação Técnica
**Hardware Target:** CPU (Processamento leve)
**Dependência do Sistema:** FFmpeg (Deve estar instalado e no PATH)

---

## 1. Contexto e Objetivo
O objetivo desta fase é normalizar a entrada de dados. Vídeos de aulas originais podem vir em diversos formatos (.mp4, .mov, .mkv), codecs de áudio (AAC, MP3) e taxas de amostragem (44.1kHz, 48kHz).

Para que a **Fase 2 (Transcrição com Whisper)** e a **Fase 4 (Clonagem de Voz)** funcionem com precisão máxima e velocidade, precisamos converter tudo para o "padrão ouro" de áudio para IA: **WAV PCM, 16kHz, Mono**.

---

## 2. Estrutura de Diretórios e I/O

O script deve operar estritamente nestes caminhos relativos à raiz do projeto:

* **Diretório de Entrada (Input):** `data/00_raw_input/`
    * *Formatos aceitos:* `.mp4`, `.mov`, `.avi`, `.mkv`
    * *Comportamento:* O script deve iterar sobre todos os arquivos desta pasta.
* **Diretório de Saída (Output):** `data/01_audio_extracted/`
    * *Formato gerado:* `.wav`
    * *Nomenclatura:* O arquivo de saída deve manter o nome exato do arquivo de entrada, trocando apenas a extensão.
        * *Exemplo:* `Aula_01_Intro.mp4` -> `Aula_01_Intro.wav`

---

## 3. Especificação Técnica do FFmpeg

O wrapper Python (`ffmpeg-python`) deve montar e executar o seguinte comando lógico:

```bash
ffmpeg -i "{INPUT_FILE}" -ar 16000 -ac 1 -c:a pcm_s16le "{OUTPUT_FILE}" -y -hide_banner -loglevel error
```

### Detalhamento Obrigatório dos Parâmetros:

1.  **`-ar 16000` (Audio Rate):**
    * Reamostra o áudio para 16.000 Hz.
    * *Motivo:* O modelo Whisper (OpenAI) é treinado nativamente em 16kHz. Se enviarmos 48kHz, a ferramenta terá que fazer downsampling interno, desperdiçando CPU na etapa de IA.
2.  **`-ac 1` (Audio Channels):**
    * Converte (downmix) estéreo para Mono.
    * *Motivo:* Reduz o tamanho do arquivo pela metade. A informação espacial (esquerda/direita) é irrelevante para transcrição e clonagem de voz.
3.  **`-c:a pcm_s16le` (Codec):**
    * Usa PCM Signed 16-bit Little Endian.
    * *Motivo:* É o formato RAW não comprimido. Evita artefatos de compressão (como em MP3) que podem causar alucinações na transcrição.
4.  **`-y` (Overwrite):**
    * Força a sobrescrita se o arquivo já existir (caso a flag de força seja usada).
5.  **`-loglevel error`:**
    * Suprime o output verboso do FFmpeg, mantendo o console limpo, mostrando apenas erros reais.

---

## 4. Requisitos Funcionais do Script (`src/pipeline/step_01_extract.py`)

O desenvolvedor deve implementar uma classe `AudioExtractor` que atenda aos seguintes requisitos:

### 4.1. Verificação de Dependência
* Ao iniciar, o script deve verificar se o comando `ffmpeg` está disponível no terminal.
* Se não estiver, deve levantar um erro crítico: `SystemError: FFmpeg not found. Please install it (brew install ffmpeg).`

### 4.2. Lógica de Caching (Idempotência)
* O sistema não deve reprocessar arquivos desnecessariamente.
* **Regra de Cache:** Antes de processar `video.mp4`, verifique se `video.wav` existe na pasta de saída.
* Se existir E tiver tamanho > 0 bytes:
    * Pular extração.
    * Logar: `[CACHE HIT] Skipping extraction for {filename}`.
* Se não existir (ou flag `--force` estiver ativa):
    * Executar extração.

### 4.3. Tratamento de Erros
* Se um arquivo de vídeo estiver corrompido, o script deve logar o erro (`[ERROR] Failed to process...`) e continuar para o próximo vídeo da lista, sem interromper todo o pipeline.

### 4.4. Logging
* O script deve fornecer feedback visual claro no terminal:
    * `[INFO] Found 3 videos in input folder.`
    * `[STEP 1] Extracting audio from 'aula_01.mp4'...`
    * `[SUCCESS] Saved 'aula_01.wav' (15.4MB).`

---

## 5. Critérios de Aceite (Definition of Done)

A implementação desta fase só será considerada pronta se:
1.  [ ] O script rodar sem erros via linha de comando (`python src/pipeline/step_01_extract.py`).
2.  [ ] Arquivos `.wav` forem gerados corretamente na pasta `data/01_audio_extracted`.
3.  [ ] O comando `ffprobe` confirmar que os arquivos de saída são **16000 Hz** e **Mono**.
4.  [ ] Executar o script uma segunda vez resulta em "Cache Hits" (nenhum processamento repetido).

---

## 6. Prompt para Implementação (Copiar para IA)

> "Atue como um Engenheiro de Software Python. Implemente o módulo `src/pipeline/step_01_extract.py` usando a biblioteca `ffmpeg-python`. Siga estritamente as especificações de I/O, parâmetros do FFmpeg e lógica de cache descritas no arquivo `02_SPEC_PHASE_01.md`. Use `pathlib` para manipulação de caminhos e `logging` para output no terminal."
