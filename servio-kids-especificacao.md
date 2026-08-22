# SERVIO KIDS — Especificação de Produto, Fluxo de Telas e Design System

*Documento de referência atualizado a partir do protótipo interativo. Reflete o estado final construído nesta conversa — serve de guia para desenvolvimento e para uso no Claude Code junto com o `CLAUDE.md`.*

---

## 1. Visão Geral do Produto

**Nome:** Servio (a logomarca atual não usa mais o sufixo "Kids", removido do app e do texto de identidade visual).
**Proposta:** aplicativo mobile para prestadores de serviços recreativos infantis (pula-pula, motinha elétrica, piscina de bolinhas, tobogã inflável etc.), permitindo controlar atendimentos por tempo ou por pacote de tempo fixo, cobrança, locações para eventos e fechamento de caixa — tudo em poucos toques.
**Plataforma:** mobile, prioridade Android (conforme documento de requisitos original).
**Marca responsável:** Marati Tech.

---

## 2. Perfis de Usuário e Regras de Acesso

| Recurso / Tela | Gestor | Monitor |
|---|---|---|
| Início (resumo do dia) | ✅ com faturamento em destaque | ✅ sem faturamento, só "meus atendimentos" |
| Novo atendimento | ✅ | ✅ |
| Atendimentos ativos | ✅ todos | ✅ só os seus |
| Histórico de atendimentos | ✅ todos, com filtros | ✅ só os seus, sem filtro de monitor |
| Pausar / Estender / Trocar brinquedo / Antecipar / Finalizar | ✅ | ✅ |
| Locações (CRUD) | ✅ | ❌ aba oculta |
| Fechamento de caixa | ✅ | ❌ aba oculta |
| Cadastro de Brinquedos / Pacotes / Usuários | ✅ via menu ⋮ | ❌ |
| Sair / trocar perfil | ✅ via menu ⋮ | ✅ via menu ⋮ |

Cada usuário tem status **Ativo/Inativo**. Usuários inativos **não aparecem mais** na tela de seleção de perfil (antes apareciam esmaecidos; a regra foi simplificada para listar só ativos).

---

## 3. Autenticação (Login)

**3.1 Tela de Perfil**
- Logomarca oficial da Servio no topo (arquivo de imagem único, sem texto "Kids").
- "Selecione seu perfil para continuar".
- **Abas "Gestores" / "Monitores"** logo abaixo da mensagem — trocar de aba troca a lista exibida (não aparecem mais as duas listas juntas).
- Cada aba lista **somente usuários ativos** daquele tipo (avatar, nome, "Gestor(a)"/"Monitor(a)", etiqueta "Ativo"). Se não houver nenhum, mostra aviso de lista vazia.
- Tocar em um usuário leva à tela de PIN.

**3.2 Tela de PIN**
- Avatar grande (92px), nome em preto, papel em cinza, link "‹ Trocar perfil".
- Teclado numérico em formato de pílula (não mais círculos), números grandes (26px), tecla de apagar em destaque (navy, ícone branco), ocupando a largura útil da tela.
- 4 dígitos → autenticação automática → Início.

**3.3 Logout:** menu ⋮ → "Sair (trocar perfil)" → volta para a tela de Perfil.

---

## 4. Estrutura de Navegação

- **Barra de status:** horário + etiqueta do perfil atual (Gestor/Monitor).
- **Appbar (barra azul sólida):** presente em todas as telas internas exceto Perfil/PIN. Ícone mini da marca + "Servio" (sem "Kids") em branco à esquerda; botão ⋮ à direita.
- **Menu ⋮:** Gestor vê "Cadastrar brinquedos", "Cadastrar pacotes", "Cadastrar usuários", divisor, "Sair"; Monitor vê só "Sair".
- **Tab bar inferior:** Início / Atendim. / Locações / Caixa (Gestor); Início / Atendim. (Monitor — os outros dois ficam ocultos).
- Telas de fluxo/detalhe (PIN, detalhe do atendimento, pagamento, telas de cadastro) escondem a tab bar e usam link "‹ Voltar" no topo.

---

## 5. Regras de Negócio Detalhadas

### 5.1 Atendimento — criação

Tela **Novo Atendimento** abre com um filtro no topo: **"Brinquedos"** (padrão) ou **"Pacote"**.

- **Modo Brinquedos:** grade de brinquedos ativos (com foto real cadastrada) + seção "Tempo" com chips **10 / 15 / 20 / 30 min** e um quinto chip **"Outro"** que revela um campo numérico para o operador digitar qualquer duração. Valor estimado = preço/min do brinquedo × minutos escolhidos, recalculado ao vivo.
- **Modo Pacote:** lista os pacotes que estão **Ativos** e com a flag **"Tempo fixo no atendimento"** ligada — cada card mostra nome do pacote, valor, tempo (com unidade) e os brinquedos inclusos. Tempo é convertido internamente para minutos (minutos ×1, horas ×60, dias ×1440, meses ×43200).
- Nome da criança é opcional em ambos os modos.
- Um único toque em "Iniciar atendimento" cria o registro (`id`, `toy`, `cliente`, `op` = usuário logado, `horaInicio`, `data`, `minutos`, `valor`, `status:'ativo'`) e o cronômetro real começa a contar a partir de `Date.now()`.

### 5.2 Atendimento — ciclo de vida e ações

- **Cronômetro real:** baseado em `elapsedMs` + `lastResumeAt`, atualizado a cada 1 segundo (não é um contador ingênuo — resiste a pausar/retomar sem perder precisão).
- **Ordenação nas listas (Início e Atendimentos ativos):** os atendimentos mais perto de acabar (ou já com tempo esgotado) ficam no **topo**; os com mais tempo restante ficam mais **embaixo**. Atendimentos de pacote (Tempo Livre) sempre ficam por último, já que não competem contra o relógio.
- **Etiquetas informativas** (linha própria, abaixo de "Cliente · horário · tempo"): "Tempo finalizado" (vermelho, tempo=0), "Quase Acabando" (amarelo escuro, 1 min restante), "Pausado" (cinza, sobrepõe as anteriores). Não aparecem para atendimentos de pacote.
- **Anel de progresso:** cor dinâmica conforme tempo restante (verde → amarelo escuro → vermelho). Para atendimentos de **pacote**, o anel normal é substituído por um **círculo verde sólido com ícone de infinito e o texto "Tempo Livre"** — tanto nas listas quanto no anel grande da tela de detalhe.
- **Nome do operador** aparece em **negrito e azul** logo após o tempo, tanto nos cards das listas (Início/Atendimentos) quanto na tela de detalhe.
- **Pausar/Retomar:** disponível na tela de detalhe **e** como botão direto em cada card da lista de Atendimentos ativos (atalho, sem precisar abrir o detalhe). Ao pausar, o cronômetro congela e a etiqueta muda para "Pausado".
- **Estender tempo:** chips +5/+10/+15 min na tela de detalhe, com abertura/fechamento suave (evita "pulos" de layout) e trava contra duplo-toque acidental.
- **Trocar brinquedo:** botão na tela de detalhe (oculto para atendimentos de pacote) — abre uma grade dos demais brinquedos ativos e um interruptor **"Recalcular valor"** (ligado por padrão: usa o preço do novo brinquedo × tempo total; desligado: mantém o valor original).
- **Antecipar pagamento:** leva à cobrança sem encerrar o atendimento; ao confirmar, volta ao detalhe com a etiqueta "✓ Pago antecipado" e o botão de antecipar some.
- **Finalizar:** leva à cobrança normalmente; se já pago antecipado, encerra direto sem pedir pagamento de novo. Ao finalizar, o registro muda para `status:'finalizado'`, some da lista de ativos e passa a aparecer no Histórico.
- **Alerta sonoro:** toca um bipe duplo (gerado via Web Audio API, sem arquivo externo) na primeira vez que o tempo de um atendimento chega a zero. Não repete a cada segundo, não toca para atendimentos de pacote nem pausados, e é reativado se o tempo for estendido depois de já ter tocado.

### 5.3 Pagamento (Pix / Dinheiro / Cartão)

- **Pix:** QR code ilustrativo + chave/CNPJ configurado, valor em destaque.
- **Dinheiro:** campo de valor recebido + chips de valores rápidos, cálculo automático de troco, botão de confirmar bloqueado se o valor for insuficiente.
- **Cartão:** mensagem orientando a usar a maquininha do estabelecimento; botão vira "Confirmar recebimento na maquininha".
- Estado de sucesso com ícone de check e retorno automático (para Início, se finalização normal; para o detalhe, se pagamento antecipado).

### 5.4 Cadastro de Brinquedos

Campos: foto (upload com pré-visualização), nome, faixa etária, capacidade máxima, valor por minuto, valor para locação, ativo/inativo. Edição ao tocar no item da lista (toggle ativo/inativo tem toque isolado, não abre edição). Imagem cadastrada aparece em todas as telas que referenciam aquele brinquedo (Novo Atendimento, cards de atendimento, detalhe).

### 5.5 Cadastro de Pacotes

Campos: **imagem do pacote** (upload, aparece como miniatura nos atendimentos criados a partir dele), nome, brinquedos inclusos (seleção múltipla, com opção especial "Todos os brinquedos"), valor, **tempo numérico + unidade** (Minutos/Horas/Dias/Meses — substituiu o campo de texto livre original), interruptor **"Tempo fixo no atendimento"** (só pacotes com essa flag ligada aparecem no modo "Pacote" de Novo Atendimento) e ativo/inativo. Edição ao tocar no item.

Pacote de exemplo: **"Pulseira Vale Tudo"** — R$ 30,00, acesso a "Todos os brinquedos", 8 horas, com imagem própria cadastrada, flag de tempo fixo ativa.

### 5.6 Locações (CRUD completo)

Tela redesenhada como "Gestão de Locações":
- Abas de filtro: **Todas / Confirmadas / Em análise**.
- Cada card tem faixa colorida à esquerda + selo de status preenchido (verde/laranja/vermelho), nome do evento (campo próprio, separado do nome do cliente), cliente, data/horário, caixa cinza com os brinquedos/combo, e "Valor total" em destaque.
- Botão de cancelar (ícone) em cada card ativo, com **modal de confirmação próprio do app** (não usa `window.confirm`, que é bloqueado no ambiente de preview).
- Botão flutuante azul **"+"** fixo no canto inferior direito, **acima da tab bar**, que não se move com o scroll da lista (corrigido — antes ele fazia parte da área rolável).
- Tocar no card (quando não cancelado) abre edição.

### 5.7 Histórico de Atendimentos

Dentro da aba "Atendimentos", alternância **Ativos / Histórico**. Na aba Histórico:
- **Filtro de período:** Hoje / Últimos 7 dias / Últimos 13 dias / Personalizado (revela dois campos de data).
- **Filtro de monitor** (só para Gestor): "Todos" + um chip dinâmico para cada monitor/gestor que tem pelo menos um atendimento finalizado.
- **Agrupamento por dia:** quando o período cobre mais de 1 dia, a lista aparece separada por cabeçalhos de data ("Hoje", "Ontem", ou dia da semana + data), cada grupo mostrando o total faturado daquele dia.
- Cada linha do histórico mostra: brinquedo, cliente, tempo utilizado, horário de início, valor e forma de pagamento — sem imagem nem anel (removidos para priorizar leitura rápida em texto).
- Botão **"Gerar PDF do histórico"** (simulado — gera nome de arquivo baseado no período filtrado).

### 5.8 Fechamento de Caixa (só Gestor)

Filtros de período e monitor (mesmo padrão do Histórico), indicadores (faturado, atendimentos, ticket médio), gráfico de barras por dia, tabela "Separado por monitor" e botão "Gerar PDF do relatório".

---

## 6. Fluxo de Telas (ordem de navegação)

1. **Perfil** → abas Gestores/Monitores → **PIN** → **Início**
2. **Início** → prévia de até 3 atendimentos ativos (ordenados por urgência) → "Novo atendimento" ou "ver todos"
3. **Novo Atendimento** → filtro Brinquedos/Pacote → (grade + tempo) ou (lista de pacotes) → iniciar → **Atendimentos ativos**
4. **Atendimentos** → alterna Ativos/Histórico → toque no card (ativos) → **Detalhe do Atendimento**
5. **Detalhe do Atendimento** → pausar/estender/trocar brinquedo/antecipar/finalizar → **Pagamento** (quando aplicável)
6. **Pagamento** → Pix/Dinheiro/Cartão → confirmação → retorno automático
7. **Locações** (Gestor) → filtro de status → cadastrar/editar/cancelar
8. **Fechamento de Caixa** (Gestor) → filtros → indicadores/gráfico/tabela → PDF
9. **Cadastro de Brinquedos / Pacotes / Usuários** (Gestor, via menu ⋮) → lista com edição ao toque + botão "+ Novo"

---

## 7. Padronização Visual (Design System)

### 7.1 Paleta de Cores

| Papel | Hex |
|---|---|
| Fundo geral | `#F3F5F8` |
| Superfície (cards) | `#FFFFFF` |
| Texto principal | `#171B22` |
| Texto secundário | `#68707C` |
| Bordas | `#E4E7EC` |
| Navy (marca, avatares, destaque) | `#16294D` |
| Azul de ação (botões, appbar, links) | `#3B6FE0` |
| Sucesso (pago, ativo, tempo livre) | `#1D8A5C` |
| Alerta (quase acabando) | `#C9861D` |
| Erro/cancelamento (tempo finalizado, cancelado) | `#D6484A` |

### 7.2 Tipografia
- **Sora** (500–800): títulos, valores monetários, números de indicadores, cronômetros.
- **Inter** (400–700): corpo de texto, labels, campos de formulário.

### 7.3 Iconografia
**Font Awesome**, autohospedado dentro do próprio HTML (fonte woff2 em base64 embutida no CSS — sem depender de CDN externo, que se mostrou bloqueado no ambiente de preview). Nenhum emoji é usado na interface funcional.

### 7.4 Componentes padrão
Botão primário (gradiente azul), botão fantasma, botão suave (pares de ação), chips de seleção única/múltipla, cards de listagem com borda e cantos arredondados, anel de progresso com cor dinâmica, círculo "Tempo Livre", toggle switch, etiquetas de status coloridas, toast de notificação (rodapé, ~2,4s), **modal de confirmação próprio** (usado no lugar de `window.confirm`/`alert`, que são bloqueados em iframes/sandboxes de preview).

### 7.5 Princípios de UX
Poucos toques para ações centrais, botões grandes para uso em pé/ao ar livre, cor sempre acompanhada de texto/ícone (nunca só cor), uma ação primária por tela, estados vazios explicativos, feedback imediato via toast, transições suaves em painéis que expandem/recolhem (evita "saltos" de layout).

---

## 8. Observações sobre o Protótipo

Este documento descreve o comportamento **tal como implementado no protótipo interativo**, que roda inteiramente no navegador sem backend. Para uma implementação de produção, ainda seriam necessários:

- Persistência real de dados (hoje vive em memória, reseta a cada reload).
- Geração real de PDF (hoje é simulada com nome de arquivo + notificação).
- QR Code Pix funcional, gerado por integração bancária real.
- Autenticação segura de PIN (hoje é só validação de 4 dígitos preenchidos).
- Sincronização multi-dispositivo e fila de sincronização offline.
- Cálculo de "tempo livre" em pacotes de longa duração (dias/meses) considerando fechamento de caixa por turno/dia, não coberto em detalhe neste protótipo.
