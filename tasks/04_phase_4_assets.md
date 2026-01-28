# Task: 04_phase_4_assets
**Fase:** 4 - Fábrica de Assets
**Input:** Roteiro + Áudio Ref | **Output:** Áudios Sintéticos + JSON Enriquecido
**Dependência:** GPT-SoVITS (Opcional via Mock), Librosa

## 1. Contexto e Referências
O roteiro diz "o que" falar. Aqui definimos "como" (áudio) e "quanto tempo" (frames) dura cada slide.
* **Ler instrução:** `instructions/05_SPEC_PHASE_04.md`

## 2. Princípios de Engenharia
* **Abstraction Layer:** O pipeline não deve depender da instalação complexa do GPT-SoVITS para funcionar em modo de desenvolvimento. Devemos ter um "Mock" (simulador).
* **Sync Integrity:** O cálculo de frames deve ser preciso e arredondado para cima (`ceil`).

## 3. Instruções de Implementação
Peça para o Copilot gerar o arquivo `src/pipeline/step_04_synthesize.py` com a classe `AssetFactory`:

### A. Configuração de Mock
* Adicione uma flag no `__init__` ou `config.py`: `USE_MOCK_TTS = True`.
* Se `True`: Não carregue modelos pesados. Apenas simule a geração.

### B. Lógica de Síntese (Com suporte a Mock)
* Para cada segmento do script:
    * Se `USE_MOCK_TTS` for True:
        * Estime a duração baseada no texto (ex: 0.15s por caractere).
        * Gere um arquivo `.wav` contendo silêncio ou ruído branco com essa duração exata usando `numpy` e `soundfile`.
    * Se `USE_MOCK_TTS` for False (Produção):
        * Chame a API/Lib do GPT-SoVITS usando o áudio de referência recortado da Fase 1.

### C. Processamento de Áudio (Padding)
* Seja Mock ou Real, aplique um **Padding de 0.5s de silêncio** ao final do áudio gerado. Isso evita cortes secos no vídeo.

### D. Calculadora e Output
* Use `librosa.get_duration` para ler o arquivo gerado (seja ruído ou voz).
* Calcule `frames = ceil(duration * 30)`.
* Salve o `enriched_script.json`.

## 4. Definition of Done (DoD)
- [ ] O script roda instantaneamente com `USE_MOCK_TTS = True`, gerando arquivos `.wav` de ruído/silêncio na pasta correta.
- [ ] Os arquivos gerados têm duração coerente com o tamanho do texto.
- [ ] O JSON final contém `durationInFrames` calculado corretamente a partir desses arquivos.