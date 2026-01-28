# Task: 02_phase_2_transcription
**Fase:** 2 - Transcrição Neural
**Input:** Áudio WAV | **Output:** JSON Bruto (Time-aligned)
**Hardware Target:** Apple Silicon (M4)

## 1. Contexto e Referências
Usaremos a NPU do Mac para transcrever. A precisão temporal (timestamps) é mais importante que a precisão textual, pois definirá a sincronia do vídeo.
* **Ler instrução:** `instructions/03_SPEC_PHASE_02.md`

## 2. Princípios de Engenharia
* **Hardware Awareness:** O código deve usar estritamente `mlx-whisper` e não a implementação padrão, para garantir performance no chip M4.
* **Data Persistence:** Salvar o resultado bruto (dump) é crucial para debug. Não tente "limpar" o texto agora; apenas salve o que a IA ouviu.
* **Resource Management:** O modelo é pesado. Carregue-o apenas uma vez na memória (Lazy Loading ou Singleton) se for processar múltiplos arquivos.

## 3. Instruções de Implementação
Peça para o Copilot gerar o arquivo `src/pipeline/step_02_transcribe.py` com a classe `Transcriber`:

### A. Setup do MLX
* Importe `mlx_whisper`.
* Defina o modelo padrão como `mlx-community/whisper-large-v3-mlx`.

### B. Lógica de Transcrição (`transcribe`)
* **Input:** Caminho do áudio `.wav`.
* **Check de Cache:** Se o `.json` correspondente já existir, pule.
* **Execução:**
    * Chame `mlx_whisper.transcribe()`.
    * Parâmetro CRÍTICO: `word_timestamps=True` (sem isso, o projeto falha na Fase 5).
    * Parâmetro Opcional: `language="pt"`.
* **Output:** Salve o dicionário resultante inteiro como um arquivo JSON na pasta `data/02_transcriptions`.

### C. Feedback de Progresso
* Transcrição demora. Adicione logs antes e depois do processo (`[INFO] Starting transcription...` / `[SUCCESS] Done in 45s`).

## 4. Definition of Done (DoD)
- [ ] O script gera um arquivo `.json`.
- [ ] Ao abrir o JSON, existe a chave `segments` e dentro dela, a chave `words` com `start` e `end` para cada palavra.
- [ ] O Activity Monitor mostra uso da GPU/Neural Engine durante a execução.
