# 00_PROJECT_STRUCTURE.md
**Projeto:** Mind-to-Script (M2S) Engine
**Arquitetura:** Linear Waterfall Pipeline (Baseada em Arquivos)
**Hardware Alvo:** Apple Silicon (M4)

## 1. Filosofia de Fluxo de Dados
O sistema opera em um fluxo unidirecional estrito. Cada etapa é um "transformador" que lê de uma pasta numerada e escreve na próxima. O sucesso da etapa anterior é pré-requisito para a próxima.

```mermaid
graph LR
    Input[0_input] -->|FFmpeg| Audio[1_audio]
    Audio -->|Whisper| Text[2_transcript]
    Text -->|LLM| Json[3_structure]
    Json -->|TTS| Synth[4_assets]
    Synth -->|Remotion| Video[5_output]
