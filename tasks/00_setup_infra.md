# Task: 00_setup_infra
**Fase:** Infraestrutura Base
**Prioridade:** Alta (Bloqueante)

## 1. Contexto e Referências
Esta tarefa estabelece a fundação do projeto. Os scripts subsequentes dependerão da estrutura de pastas e dos utilitários de configuração criados aqui.
* **Ler instrução:** `instructions/07_SPEC_ORCHESTRATOR_AND_UTILS.md`
* **Ler instrução:** `instructions/08_ENVIRONMENT_SETUP.md`

## 2. Princípios de Engenharia
* **Single Source of Truth:** Caminhos de diretório nunca devem ser strings soltas ("magic strings"). Tudo deve vir de um arquivo de configuração central.
* **OS Agnostic:** Use `pathlib` para garantir compatibilidade de caminhos (embora o alvo seja Mac, é boa prática).
* **Feedback Visual:** O terminal é a UI do desenvolvedor. Logs devem ser coloridos e semânticos.

## 3. Instruções de Implementação
Peça para o Copilot gerar os seguintes arquivos:

### A. Script de Setup (`setup_project.py`)
* Crie um script na raiz que, ao ser executado, gera a árvore de diretórios completa:
    * `src/pipeline`, `src/utils`
    * `data/00_raw_input`
    * `data/01_audio_extracted`
    * `data/02_transcriptions`
    * `data/03_structured_scripts`
    * `data/04_assets_generated`
    * `data/05_final_output`
    * `video_engine`
* O script deve criar arquivos `.gitkeep` em pastas vazias e `__init__.py` nas pastas de código fonte.

### B. Configuração Central (`src/config.py`)
* Defina `PROJECT_ROOT` dinamicamente usando `Path(__file__)`.
* Mapeie todas as pastas de dados listadas acima como constantes (ex: `INPUT_DIR`, `AUDIO_DIR`).
* Defina constantes de Modelo:
    * Whisper: `mlx-community/whisper-large-v3-mlx`
    * Ollama: `llama3.1:8b`
* Defina constantes de Áudio: `SAMPLE_RATE = 16000`, `CHANNELS = 1`.

### C. Logger Utilitário (`src/utils/logger.py`)
* Configure o `logging` do Python para usar a biblioteca `colorlog`.
* Defina um formato que inclua: Hora, Nível (Colorido), Módulo e Mensagem.
* Níveis: DEBUG (Cyan), INFO (Green), WARNING (Yellow), ERROR (Red).

### D. Dependências (`requirements.txt`)
* Liste as bibliotecas essenciais baseadas nas specs (pydantic, ffmpeg-python, mlx-whisper, ollama, librosa, colorlog).

## 4. Definition of Done (DoD)
- [ ] Ao rodar `python setup_project.py`, todas as pastas são criadas.
- [ ] Importar `src.config` em um shell python funciona e imprime os caminhos corretos.
- [ ] O logger imprime mensagens coloridas no terminal.
