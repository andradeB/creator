# Task: 00_setup_infra
**Fase:** Infraestrutura Base
**Prioridade:** Alta (Bloqueante)

## 1. Contexto e Referências
Esta tarefa estabelece a fundação do projeto. Os scripts subsequentes dependerão da estrutura de pastas e dos utilitários de configuração criados aqui.
* **Ler instrução:** `instructions/07_SPEC_ORCHESTRATOR_AND_UTILS.md`
* **Ler instrução:** `instructions/08_ENVIRONMENT_SETUP.md`

## 2. Princípios de Engenharia
* **Single Source of Truth:** Caminhos de diretório nunca devem ser strings soltas. Tudo deve vir de um arquivo de configuração central.
* **OS Agnostic:** Use `pathlib`.
* **Clean Repository:** Dados pesados (vídeo/áudio) nunca devem ser commitados no Git.

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
* Mapeie todas as pastas de dados listadas acima como constantes.
* Defina constantes de Modelo (Whisper Large-v3, Llama 3.1) e Áudio (16kHz).

### C. Logger Utilitário (`src/utils/logger.py`)
* Configure `logging` com `colorlog` (Info=Verde, Error=Vermelho).

### D. Dependências (`requirements.txt`)
* Liste: pydantic, ffmpeg-python, mlx-whisper, ollama, librosa, colorlog, soundfile, numpy.

### E. Controle de Versão (`.gitignore`)
* Crie um `.gitignore` robusto na raiz que ignore explicitamente:
    * `data/*` (Mas mantenha os `.gitkeep`: `!data/**/.gitkeep`).
    * `video_engine/node_modules`.
    * `video_engine/dist` ou `video_engine/out`.
    * `__pycache__`, `*.pyc`.
    * `.env` e pastas de venv (`.venv`, `env`).

## 4. Definition of Done (DoD)
- [ ] Rodar `python setup_project.py` cria todas as pastas.
- [ ] O arquivo `.gitignore` existe e impede o commit de arquivos na pasta `data`.
- [ ] Importar `src.config` funciona corretamente.