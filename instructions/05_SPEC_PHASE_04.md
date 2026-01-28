# 05_SPEC_PHASE_04.md

**Projeto:** Mind-to-Script (M2S)
**Fase 4:** Fábrica de Assets (TTS & Timing Calculator)
**Status:** Especificação Técnica
**Hardware Target:** Apple Silicon M4 (PyTorch com aceleração MPS/Metal)
**Dependência Crítica:** `GPT-SoVITS` (Inferência TTS), `librosa` (Análise de Áudio)

---

## 1. Contexto e Objetivo
O roteiro da Fase 3 diz *o que* falar, mas não *como*. Nesta fase, usamos IA Generativa de Áudio para sintetizar a voz.
Diferente de TTS robóticos antigos, usaremos **Voice Cloning (Few-Shot)**. O sistema pegará uma pequena amostra (3 a 10 segundos) do áudio original da Fase 1 para "aprender" o timbre do professor e narrar o novo texto limpo.

**Ponto Crítico de Engenharia:** O motor de vídeo (Fase 5/Remotion) é "surdo". Ele não sabe quando acabar um slide. Portanto, esta fase tem a responsabilidade obrigatória de medir a duração de cada áudio gerado e escrever essa informação no JSON.

---

## 2. Estrutura de Diretórios e I/O

* **Inputs:**
    1.  `data/03_structured_scripts/{video}.json` (O roteiro limpo).
    2.  `data/01_audio_extracted/{video}.wav` (O áudio original para extrair a amostra de voz).
* **Output:**
    * Diretório: `data/04_assets_generated/{video_name}/`
    * Conteúdo:
        * `seg_001.wav`, `seg_002.wav`... (Áudios individuais).
        * `enriched_script.json` (O JSON final com metadados de tempo).

---

## 3. Especificação Técnica: GPT-SoVITS & Reference Audio

### 3.1 Seleção Automática de Amostra (The "Ref Audio")
Para clonar a voz, o modelo precisa de um clipe de referência do áudio original + a transcrição desse clipe.
* **Estratégia:** O script deve recortar automaticamente um trecho de 5 a 10 segundos do áudio original (`data/01_audio_extracted`).
* **Dica de Implementação:** Usar o arquivo de transcrição da Fase 2 (`02_transcriptions`) para encontrar um segmento que tenha entre 5s-10s e recortar exatamente naquele timestamp. Isso garante que temos o áudio e o texto correspondente para o prompt do GPT-SoVITS.

### 3.2 Configuração do TTS
* **Engine:** GPT-SoVITS (versão compatível com MPS é preferível, ou CPU rápida do M4).
* **Língua:** Português (pt).
* **Parâmetros:**
    * `top_k`: 5 (Reduz alucinações de voz).
    * `temperature`: 0.8 (Variação natural da fala).

---

## 4. Lógica de Enriquecimento (O "Calculator")

Para cada segmento gerado, o script deve calcular a duração em **Frames**.
* **FPS do Projeto:** 30 quadros por segundo (Padrão web/YouTube).
* **Fórmula:** `durationInFrames = ceil(duration_in_seconds * 30)`

### Schema do JSON Enriquecido (Output Final)
O JSON de saída deve ser idêntico ao da Fase 3, mas com dois campos novos em cada segmento:

```json
{
  "meta": { ... },
  "segments": [
    {
      "id": "seg_01",
      "type": "title_screen",
      "voiceover_text": "Olá e bem-vindos...",
      
      "//": "Campos adicionados nesta fase:",
      "audio_file": "data/04_assets_generated/video_01/seg_01.wav",
      "durationInFrames": 150  // (Ex: 5 segundos * 30 fps)
    }
  ]
}
```

---

## 5. Requisitos Funcionais do Script (`src/pipeline/step_04_synthesize.py`)

### 5.1. Classe `VoiceSynthesizer`
* Deve carregar o modelo TTS na memória apenas uma vez.
* Deve criar a pasta específica para o vídeo (`mkdir -p data/04.../{video_name}`).

### 5.2. Pipeline de Processamento
1.  Carregar `script.json`.
2.  Identificar e preparar o Áudio de Referência (Recortar do original).
3.  Loop `for segment in script['segments']`:
    * Verificar se `voiceover_text` existe.
    * Gerar áudio `.wav`.
    * **Padding de Áudio:** Adicionar 0.5s de silêncio (zeros) ao final do array numpy antes de salvar. Isso evita cortes abruptos no vídeo.
    * Salvar como `seg_{id}.wav`.
    * Ler arquivo com `librosa` ou `wave` para pegar `seconds`.
    * Calcular `frames = seconds * 30`.
    * Atualizar o objeto do segmento no JSON.
4.  Salvar `enriched_script.json`.

### 5.3. Cache de Áudio
* Gerar áudio é a operação mais cara computacionalmente. Se `seg_01.wav` já existe, **não gerar novamente**. Apenas ler a duração e atualizar o JSON.

---

## 6. Critérios de Aceite (Definition of Done)

1.  [ ] O script cria uma subpasta para o projeto em `data/04_assets_generated/`.
2.  [ ] Arquivos `.wav` individuais são criados, são audíveis e possuem um pequeno silêncio ao final.
3.  [ ] A voz gerada lembra o timbre do orador original (mesmo que não seja idêntica, deve ser consistente).
4.  [ ] O arquivo `enriched_script.json` é gerado e contém o campo `durationInFrames` (inteiro) em todos os segmentos.
5.  [ ] A soma das durações dos áudios corresponde aproximadamente ao tempo total estimado da aula.

---

## 7. Prompt para Implementação (Copiar para IA)

> "Atue como um Engenheiro de Audio e Python. Implemente o módulo `src/pipeline/step_04_synthesize.py`. Use uma API wrapper para o GPT-SoVITS (ou mock para teste inicial). O script deve ler o roteiro da Fase 3, gerar áudios para cada fala, adicionar 0.5s de silêncio ao final (padding) e salvar em uma pasta organizada. Crucialmente, meça a duração final de cada arquivo wav para calcular o `durationInFrames` (base 30fps). Salve o resultado como `enriched_script.json`. Implemente cache para evitar re-sintetizar áudios já existentes."