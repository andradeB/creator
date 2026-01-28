# Task: 05_phase_5_render
**Fase:** 5 - Renderização e Montagem
**Input:** JSON Enriquecido | **Output:** MP4 Final
**Dependência:** Node.js, Remotion

## 1. Contexto e Referências
Esta etapa é híbrida. Python comanda, Node.js executa. O Python atua como um gerenciador de processo.
* **Ler instrução:** `instructions/06_SPEC_PHASE_05.md`

## 2. Princípios de Engenharia
* **Cross-Language Communication:** A passagem de dados entre Python e React deve ser feita estritamente via injeção de arquivo JSON (Props).
* **Dependency Check:** O script nunca deve assumir que `npm install` foi rodado. Ele deve verificar a pasta `node_modules` e instalar se necessário.
* **Observability:** O Python deve capturar o `stdout` do processo Node para mostrar a barra de progresso do render no terminal Python.

## 3. Instruções de Implementação
Esta task tem duas partes: React (Setup) e Python (Execution).

### Parte A: Setup React (`video_engine/`)
* Instrua a criação/modificação do `src/Root.tsx` e `src/Composition.tsx` no projeto Remotion.
* O componente deve:
    * Ler `jsonPath` das props.
    * Usar `require()` dinâmico ou `fetch` local para ler o JSON.
    * Mapear a lista de segmentos para um componente `<Series>`.
    * Renderizar componentes condicionais (`Title`, `Code`, `Concept`) baseados no tipo do segmento.

### Parte B: Script Python (`src/pipeline/step_05_render.py`)
* Crie a classe `VideoRenderer`.
* Método `check_dependencies`: Roda `npm install` na pasta `video_engine` se necessário.
* Método `render`:
    * Constrói o comando: `npx remotion render ... --props='{"jsonPath": "..."}'`.
    * Usa `subprocess.Popen` para executar em tempo real.
    * Salva o vídeo final em `data/05_final_output`.

## 4. Definition of Done (DoD)
- [ ] O comando Python dispara o renderizador Node.js.
- [ ] O vídeo final `.mp4` aparece na pasta de saída.
- [ ] O vídeo respeita os tempos definidos no JSON (sincronia áudio/vídeo).
