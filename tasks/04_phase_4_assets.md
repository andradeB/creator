# Task: 04_phase_4_assets
**Fase:** 4 - Fábrica de Assets
**Input:** Roteiro + Áudio Ref | **Output:** Áudios Sintéticos + JSON Enriquecido
**Dependência:** GPT-SoVITS, Librosa

## 1. Contexto e Referências
O roteiro diz "o que" falar. Aqui definimos "como" (áudio) e "quanto tempo" (frames) dura cada slide.
* **Ler instrução:** `instructions/05_SPEC_PHASE_04.md`

## 2. Princípios de Engenharia
* **Asset Integrity:** Cada segmento do roteiro deve ter um arquivo de áudio correspondente. Nenhuma chave órfã.
* **Math Precision:** O cálculo de frames deve ser arredondado para cima (`ceil`) para evitar cortes prematuros no vídeo.
* **File Organization:** Os assets gerados devem ficar em uma subpasta específica para cada vídeo (`data/04_assets/{nome_video}/`) para não virar bagunça.

## 3. Instruções de Implementação
Peça para o Copilot gerar o arquivo `src/pipeline/step_04_synthesize.py` com a classe `AssetFactory`:

### A. Preparação do Reference Audio
* Implemente lógica para ler a transcrição da Fase 2, encontrar um segmento entre o segundo 5 e 15, e recortar esse trecho do áudio original da Fase 1 usando FFmpeg ou Soundfile. Isso será a "voz clone".

### B. Loop de Síntese
* Para cada segmento no `script.json`:
    * Verifique se já existe áudio gerado.
    * Se não, chame o TTS (Mock ou API do GPT-SoVITS) passando o texto e o áudio de referência.
    * Salve como `seg_{id}.wav`.

### C. Calculadora de Tempo (The Calculator)
* Use `librosa.get_duration(filename)` para ler o wav gerado.
* Calcule: `frames = ceil(duration_sec * 30_FPS)`.
* Injete este valor no objeto do segmento.

### D. Output Final
* Salve um novo arquivo `enriched_script.json` contendo os caminhos absolutos dos áudios e a duração em frames.

## 4. Definition of Done (DoD)
- [ ] Arquivos `.wav` são criados na subpasta correta.
- [ ] O JSON Enriquecido contém o campo `durationInFrames` (inteiro) em todos os itens.
- [ ] O áudio gerado é audível e coerente.
