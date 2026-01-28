# Task: 06_orchestrator
**Fase:** Orquestração Final
**Input:** CLI Args | **Output:** Pipeline Completo
**Arquivos:** `src/main.py`

## 1. Contexto e Referências
Unificar os 5 scripts isolados em uma ferramenta de CLI coesa e robusta.
* **Ler instrução:** `instructions/07_SPEC_ORCHESTRATOR_AND_UTILS.md`

## 2. Princípios de Engenharia
* **Fail Fast & Clean:** Se o script parar no meio, o usuário deve saber exatamente onde e porquê. Stack traces devem ser logados, mas a mensagem final deve ser amigável.
* **Modularidade:** O `main.py` não deve conter lógica de negócio. Ele apenas importa as classes (Extractor, Transcriber, etc.) e chama seus métodos.
* **Developer Experience:** Argumentos de CLI (`--start-at`, `--force`) são essenciais para evitar reprocessar tudo durante o desenvolvimento.

## 3. Instruções de Implementação
Peça para o Copilot gerar o `src/main.py`:

### A. Argument Parsing (`argparse`)
* `--video`: Obrigatório. Nome do arquivo (ex: `aula.mp4`).
* `--start-at`: Opcional (int 1-5). Define em qual etapa começar (padrão: 1).
* `--force`: Opcional. Sobrescreve caches.

### B. Pipeline Execution Flow
* Instanciar o Logger.
* **Step 1:** Chamar `AudioExtractor`. Input: Video Path. Output: Audio Path.
* **Step 2:** Chamar `Transcriber`. Input: Audio Path. Output: Transcript Path.
* **Step 3:** Chamar `ScriptArchitect`. Input: Transcript Path. Output: Script Path.
* **Step 4:** Chamar `AssetFactory`. Input: Script Path + Audio Path. Output: Enriched Script Path.
* **Step 5:** Chamar `VideoRenderer`. Input: Enriched Script Path. Output: Final Video.

### C. Tratamento Global
* Envolva tudo em um bloco `try/except Exception as e`.
* Em caso de erro: `logger.critical(f"Pipeline failed at step X: {e}")`.

## 4. Definition of Done (DoD)
- [ ] Rodar `python src/main.py --video teste.mp4` executa todas as etapas sequencialmente.
- [ ] Rodar com `--start-at 3` pula as etapas 1 e 2 e começa direto na estruturação (assumindo que os arquivos anteriores existem).
- [ ] Logs coloridos mostram o progresso passo a passo.
