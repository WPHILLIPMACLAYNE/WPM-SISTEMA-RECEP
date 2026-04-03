# Fechamento GO Final — Kanban de Pendências

Data: 2026-04-03
Arquivo auditado: `index.html`
Método de validação: inspeção direta do código e rastreio do fluxo de chamadas no SPA single-file.

## O que foi encontrado

- O atalho `Alt + ArrowLeft / Alt + ArrowRight` já estava implementado no fluxo de acessibilidade do kanban.
- A captura do atalho ocorre no listener global de teclado em `bindAcessibilidade()`, quando o foco está em um card de pendência.
- A função `moverPendenciaPorTeclado(id, direcao)` já existia e já fazia a transição de status na ordem `aberto -> respondido -> concluido`.
- A pendência identificada pela auditoria anterior não era ausência de funcionalidade; era descompasso de rastreabilidade e precisão da ajuda textual.

## O que foi validado

### 1. Atalho existente

- `bindAcessibilidade()` intercepta `Alt + ArrowLeft` e `Alt + ArrowRight` no card focado do kanban.
- O atalho chama `moverPendenciaPorTeclado(...)`.

### 2. Foco

- Antes da mudança de status, `moverPendenciaPorTeclado(...)` chama `agendarRetornoFocoPendencia(id)`.
- O kanban usa roving tabindex com `atualizarRovingPendencias()`.
- Após o rerender, `renderPending()` chama `restaurarFocoPendenteSeNecessario()`, que recoloca o foco no mesmo card, já na nova coluna, via `requestAnimationFrame`.

### 3. Render

- `updatePendingStatus(id, status)` altera o status no estado atual.
- Em seguida, a função agenda `requestRender(['hero', 'dashboard', 'pending'])`.
- O lote de render chama `renderPending()`, que recompõe o kanban a partir do estado atualizado.

### 4. Persistência

- `updatePendingStatus(id, status)` chama `saveData()`.
- `saveData()` atualiza `storage.periods[currentPeriodKey]` e delega para `saveStore(...)`.
- `saveStore(...)` persiste o store saneado via `persistStoredJson(...)`, mantendo o contrato atual de persistência híbrida do app.

### 5. Guarda de escrita

- O fluxo continua protegido por `assertWritableCurrentPeriod(...)`, preservando o bloqueio de meses fechados.

## O que foi corrigido

- O texto de ajuda `pendingKeyboardHelp` foi ajustado para refletir com precisão o comportamento real:
  - navegação entre cards com `ArrowUp / ArrowDown`
  - salto para extremidades com `Home / End`
  - movimentação entre colunas com `Alt + ArrowLeft / Alt + ArrowRight`
- Foi adicionado `aria-keyshortcuts` aos cards do kanban para reforçar rastreabilidade e acessibilidade sem alterar o comportamento.

## Impacto técnico

- Nenhuma mudança de arquitetura.
- Nenhuma mudança de comportamento central.
- Nenhuma mudança em modelo de dados, persistência, render pipeline ou estrutura SPA/browser-only/single-file.
- Correção estritamente mínima, voltada a precisão documental e acessibilidade do fluxo já existente.

## Status final

GO final.
