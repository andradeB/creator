# Task: 01_phase_1_extraction
**Fase:** 1 - Ingestão e Extração
**Input:** Vídeo Bruto | **Output:** Áudio WAV 16kHz
**Dependência:** FFmpeg

## 1. Contexto e Referências
O objetivo é normalizar a entrada. Modelos de IA são sensíveis a formatos de áudio. Precisamos padronizar tudo para o formato nativo do Whisper.
* **Ler instrução:** `instructions/02_SPEC_PHASE_01.md`

## 2. Princípios de Engenharia
* **Idempotência (Cache):** Se o script rodar 10 vezes, ele só deve processar o arquivo na primeira vez. Nas outras 9, deve detectar que o output já existe.
* **Fail Fast:** Se o FFmpeg não estiver instalado, o script deve crashar imediatamente com uma mensagem clara, antes de tentar processar qualquer coisa.
* **Wrapper Pattern:** O código Python deve apenas orquestrar; o trabalho pesado é do binário FFmpeg.

## 3. Instruções de Implementação
Peça para o Copilot gerar o arquivo `src/pipeline/step_01_extract.py` com a classe `AudioExtractor`:

### A. Verificação de Ambiente
* No `__init__`, verifique se o comando `ffmpeg` responde no terminal. Se não, levante `EnvironmentError`.

### B. Lógica de Processamento (`process_video`)
* **Input:** Caminho do vídeo (`pathlib.Path`).
* **Check de Cache:** Verifique se o arquivo destino (.wav) existe e tem tamanho > 0 bytes. Se sim, logue "Cache Hit" e retorne o caminho.
* **Conversão FFmpeg:**
    * Use `ffmpeg-python` para montar o comando.
    * Flags obrigatórias: `-ar 16000` (Sample Rate), `-ac 1` (Mono), `-c:a pcm_s16le` (Codec WAV).
    * Configure `-loglevel error` para não sujar o terminal.
* **Retorno:** O caminho absoluto do arquivo `.wav` gerado.

### C. Tratamento de Erro
* Use `try/except` ao chamar o `ffmpeg.run()`. Se falhar, logue o erro específico e levante uma exceção para parar o pipeline.

## 4. Definition of Done (DoD)
- [ ] O script converte um `.mp4` em `.wav`.
- [ ] `ffprobe` confirma que o output é 16kHz e Mono.
- [ ] Rodar o script duas vezes seguidas gera um "Cache Hit" na segunda vez.
