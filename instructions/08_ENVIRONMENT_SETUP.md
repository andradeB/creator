# 08_ENVIRONMENT_SETUP.md

**Projeto:** Mind-to-Script (M2S)
**Fase:** Preparação de Ambiente (Zero-to-Hero)
**Hardware Target:** Apple Silicon (M4) - macOS Sequoia+

---

## 1. Objetivo
Este documento define os pré-requisitos de sistema. Antes de pedir para a IA gerar qualquer código Python ou React, você deve garantir que estas ferramentas estejam instaladas no seu Mac.

---

## 2. Ferramentas de Sistema (Homebrew)
Abra o Terminal e instale os binários essenciais.

1.  **Homebrew** (Se ainda não tiver):
    ```bash
    /bin/bash -c "$(curl -fsSL [https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh](https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh))"
    ```

2.  **FFmpeg** (Obrigatório para Fase 1):
    ```bash
    brew install ffmpeg
    ```

3.  **Ollama** (Obrigatório para Fase 3):
    * Baixe em: [ollama.com](https://ollama.com)
    * Após instalar, abra o terminal e puxe o modelo Llama 3.1:
    ```bash
    ollama pull llama3.1:8b
    ```

4.  **Node.js** (Obrigatório para Fase 5):
    * Recomendado usar `fnm` ou `nvm`, mas via brew funciona:
    ```bash
    brew install node
    # Verifique a versão (deve ser v18 ou superior)
    node -v
    ```

---

## 3. Ambiente Python (Conda/Venv)

Para usar o **MLX** (Apple Silicon) corretamente, recomenda-se um ambiente limpo com Python 3.10 ou 3.11.

```bash
# Opção 1: Usando Conda (Miniforge - Recomendado para Apple Silicon)
conda create -n m2s python=3.10
conda activate m2s

# Opção 2: Usando Venv padrão
python3 -m venv venv
source venv/bin/activate
```

---

## 4. Dependências Python Críticas (`requirements.txt`)

Quando a IA gerar o arquivo `requirements.txt` baseado nas specs anteriores, ele deve conter versões compatíveis com M-Series.

**Lista de Referência para a IA:**
```text
# Core & Utils
pydantic>=2.0
tqdm
colorlog

# Fase 1 (Audio Extraction)
ffmpeg-python

# Fase 2 (Transcription - Apple MLX)
mlx
mlx-whisper

# Fase 3 (LLM Interface)
ollama

# Fase 4 (TTS & Audio Analysis)
librosa
soundfile
numpy
# Nota: GPT-SoVITS pode exigir instalação via git ou wheel específico
# torch deve ser compatível com MPS (Metal Performance Shaders)
torch
torchaudio
```

---

## 5. Inicialização do Projeto Remotion

Antes de rodar o pipeline, você precisará inicializar a pasta `video_engine` manualmente uma vez. Este passo cria o esqueleto do projeto React.

```bash
# Na raiz do projeto 'mind-to-script':
npx create-remotion@latest video_engine

# -- Perguntas do instalador --
# 1. Template? Escolha "Blank" (Vazio)
# 2. TypeScript? Sim (Recomendado)
# 3. Install dependencies? Sim

# Após instalar:
cd video_engine
npm install
```

---

## 6. Prompt Final de Verificação (Checklist)

> "Atue como um Engenheiro DevOps. Verifique se o meu ambiente local cumpre todos os requisitos descritos em `08_ENVIRONMENT_SETUP.md`. Crie um script de bash simples `setup_check.sh` que roda comandos de versão (`ffmpeg -version`, `ollama --version`, `node -v`, `python --version`) e imprime um checklist verde/vermelho no terminal para eu saber se posso começar a codar."
