# 06_SPEC_PHASE_05.md

**Projeto:** Mind-to-Script (M2S)
**Fase 5:** Renderização Programática (Video Assembly)
**Status:** Especificação Técnica
**Hardware Target:** Apple Silicon M4 (Renderização via CPU/GPU Metal)
**Dependência Crítica:** `Node.js v18+`, `Remotion CLI`

---

## 1. Contexto e Objetivo
Esta é a etapa de "Load" do pipeline ETL. O roteiro está pronto, os áudios estão gravados e cronometrados. Agora precisamos compilar tudo em um arquivo `.mp4` único.

Utilizaremos o **Remotion**, uma biblioteca que permite criar vídeos usando React. A grande vantagem é que o vídeo é definido por código (declarativo), permitindo que o JSON da Fase 4 controle 100% do que aparece na tela sem intervenção humana.

---

## 2. Estrutura de Diretórios e I/O

* **Input de Dados:** `data/04_assets_generated/{video_id}/enriched_script.json`
    * *Contém:* Lista de slides, textos, referências para os arquivos `.wav` e durações.
* **Input de Assets:** Arquivos `.wav` na mesma pasta do JSON.
* **Output:** `data/05_final_output/{video_id}_remastered.mp4`
* **Local do Código React:** Pasta `video_engine/` (na raiz do projeto).

---

## 3. Especificação da Ponte Python-Node

O script Python `step_05_render.py` não "gera" o vídeo diretamente. Ele atua como um gerente de processo que invoca o `npm`.

### Comando de Renderização
O script deve construir e executar um comando shell similar a este:

```bash
npx remotion render src/index.ts MainComposition \
  --props='{"jsonPath": "/abs/path/to/enriched_script.json"}' \
  --output="/abs/path/to/data/05_final_output/video.mp4" \
  --gl=angle  # Otimização para Mac M-series
```

*Nota:* Passar o caminho do JSON via props permite que o componente React leia o arquivo dinamicamente usando `require()` ou `fs` (já que Remotion roda em ambiente Node durante o render).

---

## 4. Especificação do Frontend (Remotion/React)

O projeto React em `video_engine/` deve ter um componente principal (ex: `MainComposition`) preparado para receber dados dinâmicos.

### 4.1. Arquitetura do Componente
* **`<Composition />`:** Deve ler o JSON fornecido nas props.
* **`<Series />`:** Usar este componente do Remotion para encadear os segmentos sequencialmente.
* **Dynamic Mapping:**
  ```tsx
  <Series>
    {data.segments.map((seg) => (
      <Series.Sequence key={seg.id} durationInFrames={seg.durationInFrames}>
        <Audio src={seg.audio_file} />
        <VisualModule type={seg.type} content={seg.visual_content} />
      </Series.Sequence>
    ))}
  </Series>
  ```

### 4.2. Módulos Visuais (`VisualModule`)
O componente deve ter um "Switch Case" para renderizar o layout correto baseado no `seg.type`:
* **`title_screen`:** Fundo sólido/gradiente, texto centralizado grande.
* **`code_block`:** Editor estilo VS Code (sintaxe highlight), código digitando sozinho (efeito typewriter).
* **`concept`:** Layout de dois terços (Texto à esquerda, diagrama/ícone à direita).

---

## 5. Requisitos Funcionais do Script (`src/pipeline/step_05_render.py`)

### 5.1. Verificação de Ambiente
* Verificar se `npm` e `node` estão instalados.
* Verificar se a pasta `video_engine/node_modules` existe. Se não, rodar `npm install` automaticamente antes de renderizar.

### 5.2. Execução do Render
* Construir caminhos absolutos para o Input (JSON) e Output (MP4).
* Executar o comando do Remotion usando `subprocess.run`.
* Capturar o output (stdout/stderr) para mostrar a barra de progresso do Remotion no terminal Python.

### 5.3. Cache de Render
* Renderizar vídeo é pesado. Se o arquivo final `.mp4` já existe, perguntar ao usuário ou verificar flag `--force` antes de sobrescrever.

---

## 6. Critérios de Aceite (Definition of Done)

1.  [ ] O script `step_05_render.py` instala dependências do Node se necessário.
2.  [ ] O Remotion inicia a renderização sem erros de "Component not found".
3.  [ ] O vídeo final `.mp4` é salvo na pasta `05_final_output`.
4.  [ ] Ao assistir o vídeo:
    * O áudio está sincronizado com o visual.
    * Quando o áudio de um slide acaba, o vídeo corta imediatamente para o próximo (sem telas pretas ou cortes abruptos).
    * O código aparece na tela formatado corretamente.

---

## 7. Prompt para Implementação (Copiar para IA)

> "Atue como um Engenheiro Full Stack (Python + React). Primeiro, descreva o componente React `MainComposition` que usa `Remotion` para renderizar uma timeline dinâmica baseada em um JSON de input. Em seguida, implemente o script Python `src/pipeline/step_05_render.py` que usa `subprocess` para disparar o comando CLI do Remotion (`npx remotion render`). Garanta que os caminhos de arquivos (JSON e Output) sejam passados corretamente como props ou argumentos. Implemente logs de progresso."
