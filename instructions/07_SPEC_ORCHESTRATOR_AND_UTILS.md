# 07_SPEC_ORCHESTRATOR_AND_UTILS.md

**Projeto:** Mind-to-Script (M2S)
**Fase:** Orquestração Final & Infraestrutura
**Arquivos Alvo:** `src/main.py`, `src/config.py`, `src/utils/logger.py`
**Objetivo:** Unificar os módulos isolados em um pipeline executável via CLI.

---

## 1. Contexto
Até o momento, temos 5 scripts isolados (`step_01` a `step_05`). O `main.py` será o ponto de entrada único. Ele deve gerenciar o fluxo de dados, passar os caminhos corretos de entrada/saída para cada etapa e tratar erros globalmente (Fail Fast).

Além disso, precisamos definir as constantes do projeto (caminhos de pastas) em um único lugar para evitar "magic strings" espalhadas pelo código.

---

## 2. Especificação: `src/config.py` (Central de Configuração)
Este arquivo deve conter todas as constantes de diretórios e configurações de modelos.

**Requisitos:**
* Usar `pathlib.Path` para definir caminhos absolutos baseados na raiz do projeto.
* Definir estrutura:
    ```python
    PROJECT_ROOT = Path(__file__).parent.parent
    DATA_DIR = PROJECT_ROOT / "data"
    
    # Paths das Fases
    INPUT_DIR = DATA_DIR / "00_raw_input"
    AUDIO_DIR = DATA_DIR / "01_audio_extracted"
    TRANSCRIPT_DIR = DATA_DIR / "02_transcriptions"
    SCRIPT_DIR = DATA_DIR / "03_structured_scripts"
    ASSETS_DIR = DATA_DIR / "04_assets_generated"
    OUTPUT_DIR = DATA_DIR / "05_final_output"
    
    # Configs de Modelos
    WHISPER_MODEL = "mlx-community/whisper-large-v3-mlx"
    OLLAMA_MODEL = "llama3.1:8b"
    ```

---

## 3. Especificação: `src/utils/logger.py`
Um logger customizado para dar feedback visual bonito no terminal.

**Requisitos:**
* Configurar o módulo `logging` padrão do Python.
* Formato sugerido: `[HORA] [LEVEL] [MÓDULO] Mensagem`
* Cores (opcional, mas recomendado): Verde para SUCCESS, Amarelo para WARNING, Vermelho para ERROR.

---

## 4. Especificação: `src/main.py` (O Maestro)

Este script importa as classes definidas nas Fases 1-5 e as executa sequencialmente.

### 4.1. Interface de Linha de Comando (CLI)
Deve usar `argparse` para aceitar argumentos:
* `--video`: (Obrigatório) Nome do arquivo de vídeo na pasta `00_raw_input` (ex: `aula01.mp4`).
* `--force`: (Opcional) Flag para forçar reprocessamento ignorando caches.
* `--start-at`: (Opcional) Número da fase para começar (1 a 5). Útil para debug.

### 4.2. Lógica de Execução (Pipeline)
O fluxo deve ser envolto em um bloco `try/catch` global.

1.  **Setup:** Inicializar Logger e verificar se o arquivo de vídeo existe em `INPUT_DIR`.
2.  **Step 1 (Extract):**
    * Instanciar `AudioExtractor`.
    * Executar `process(video_path)`.
    * Receber caminho do áudio gerado.
3.  **Step 2 (Transcribe):**
    * Instanciar `Transcriber`.
    * Executar `transcribe(audio_path)`.
    * Receber caminho do JSON bruto.
4.  **Step 3 (Structure):**
    * Instanciar `ScriptArchitect` (Ollama Wrapper).
    * Executar `structure(json_path)`.
    * Receber caminho do script limpo.
5.  **Step 4 (Synthesize):**
    * Instanciar `AssetFactory`.
    * Executar `generate_assets(script_path, reference_audio_path)`.
    * Receber caminho do script enriquecido.
6.  **Step 5 (Render):**
    * Instanciar `VideoRenderer`.
    * Executar `render(enriched_json_path)`.

### 4.3. Tratamento de Erro
Se qualquer passo falhar (lançar exceção), o `main.py` deve:
1.  Logar o erro com traceback (`logger.error`).
2.  Abortar imediatamente a execução.
3.  Não deixar o processo "pendurado".

---

## 5. Prompt para Implementação (Copiar para IA)

> "Atue como um Arquiteto de Software Python. Crie os arquivos finais de infraestrutura para o projeto Mind-to-Script.
>
> 1.  `src/config.py`: Defina todos os caminhos de diretórios usando `pathlib`.
> 2.  `src/utils/logger.py`: Configure um logger colorido.
> 3.  `src/main.py`: Implemente o orquestrador CLI usando `argparse`. Ele deve importar as classes dos módulos `src.pipeline.step_XX` (assuma que elas existem conforme specs anteriores) e encadear a execução. Implemente a lógica de `--start-at` para pular etapas e tratamento global de erros."
