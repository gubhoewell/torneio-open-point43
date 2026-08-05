# Torneio OPEN Point 43 — Sistema de Vendas & Checklist

Sistema pra organizar o **Torneio OPEN Point 43 de Beach Tennis (14-16/08/2026)**: checklist de tarefas do evento compartilhado com o time, e lançamento de vendas (inscrições de dupla + patrocínio) com comissão de 10% pra quem vende.

## URLs e credenciais

| Item | Valor |
|---|---|
| App (produção) | `https://gubhoewell.github.io/torneio-open-point43/` |
| Repo GitHub | `github.com/gubhoewell/torneio-open-point43` |
| Projeto Firebase | `point-43-torneio-open` (Realtime Database) |
| `databaseURL` | `https://point-43-torneio-open-default-rtdb.firebaseio.com` |

## ⚠️ Alerta crítico — regras do Firebase revertem sozinhas

As regras do Realtime Database (`{"rules":{".read":true,".write":true}}`) **revertem automaticamente para o padrão fechado depois de ~30 dias**, e quando isso acontece **o app salva silenciosamente sem erro nenhum** — quem estiver lançando venda ou marcando tarefa não percebe que nada foi salvo. Esse mesmo problema já foi documentado no Game Arena Comercial.

**Antes de cada evento que usar este sistema, confirmar no Firebase Console → Realtime Database → Rules que ainda está `{"rules":{".read":true,".write":true}}`.** Pro Torneio OPEN de agosto/2026, checar isso na véspera (13/08).

## Como configurar o Firebase (primeira vez)

1. Criar projeto no [Firebase Console](https://console.firebase.google.com) chamado `point-43-torneio-open`.
2. Ativar **Realtime Database** (não Firestore).
3. Em Rules, colar `{"rules":{".read":true,".write":true}}` e publicar.
4. Em Configurações do Projeto → Geral → apps da Web, criar um app e copiar o objeto de config (`apiKey`, `databaseURL`, etc).
5. Colar esses valores no topo do `<script>` de `index.html`, na constante `FIREBASE_CONFIG`.
6. Cadastrar o primeiro usuário admin diretamente no console (aba Realtime Database → Dados), criando manualmente `usuarios/{algumId}` com `{nome, pin, isAdmin:true, ativo:true}` — depois disso dá pra cadastrar o resto do time pela própria tela de Configurações do app.

## Modelo de dados

```
usuarios/{uid}          → nome, pin (4 dígitos), cargo, isAdmin, isFinanceiro, ativo
categorias/{catId}      → nome, valor (inscrição por dupla), ativo, ordem
tiersPatrocinio/{id}    → nome, valor, ativo, ordem
config/                 → comissaoPercentual (default 10)

inscricoes/{id}
  categoriaId, categoriaNome (snapshot), jogador1, jogador2, contato,
  valor (travado no lançamento), vendedorId, vendedorNome, criadoEm,
  status: pendente | confirmado | cancelado,
  confirmadoPor, confirmadoPorNome, confirmadoEm,
  comissaoPercentual, comissaoValor (só gravado na confirmação — nunca recalculado depois)

patrocinios/{id}        → mesma estrutura de inscricoes, trocando categoria por tier + empresa/contato

tarefasEvento/{id}      → desc, grupo (texto livre), responsavel (uid opcional), ordem, status (a-fazer|em-andamento|concluida), criadoPor, criadoEm
```

**Regra de ouro:** `valor` e `comissaoValor` são fotografias travadas por registro. Mudar o preço de uma categoria ou o % de comissão global no admin **não** altera vendas já lançadas ou já confirmadas — só afeta vendas novas.

## Fluxo de venda → comissão

1. Vendedor(a) lança inscrição ou patrocínio → grava com `status: 'pendente'`.
2. Financeiro (Benedito, ou qualquer admin) confirma em **Confirmar Pagamentos** só depois que o pagamento realmente caiu → grava `comissaoValor = valor × comissaoPercentual/100`, trava esse número no registro.
3. Vendedor(a) acompanha em **Meu Resultado**: comissão pendente é só uma estimativa (não travada); comissão confirmada é o valor real gravado.
4. Cancelar uma venda pendente tira ela do ranking e das somas — não é possível cancelar uma venda já confirmada pelo app (se precisar estornar, editar direto no Firebase Console).

## Papéis e abas

- **Todo mundo** (professores + comercial): Lançar Venda, Checklist, Meu Resultado.
- **Financeiro** (`isFinanceiro:true`, ou qualquer admin): + Confirmar Pagamentos.
- **Admin** (`isAdmin:true` — Gustavo, e quem mais fizer sentido): + Visão Geral, Ranking, Todas as Vendas, Configurações (CRUD de categorias, tiers, usuários, % de comissão global).

## Checklist do evento — como funciona

Tarefas ficam agrupadas por um campo de texto livre `grupo` (não é uma entidade separada no banco — um grupo "existe" a partir do momento que alguma tarefa usa aquele nome). Grupos padrão sugeridos na primeira carga: Estrutura & Logística, Arbitragem, Comercial & Patrocínio, Marketing & Comunicação, Operação nos Dias do Evento — mas qualquer nome novo digitado no modal "+ Grupo" vira um grupo válido.

Reordenar: botões ▲▼ funcionam em qualquer dispositivo (não depende de drag-and-drop, que não é confiável em touch/mobile). Clicar na bolinha de status cicla `a-fazer → em-andamento → concluída → a-fazer`.

## Pendências de dados reais (a preencher pelo Gustavo)

- Tabela de categorias do torneio + valor de inscrição por dupla de cada uma.
- Lista de professores/comercial que vão vender (nome + PIN de 4 dígitos que cada um escolhe).
- Tiers de patrocínio e valores reais (o app já vem com sugestão Master/Ouro/Prata/Bronze, mas os valores são placeholder).
- Itens reais do checklist organizacional (o que precisa ser feito pra montar o torneio).

Tudo isso é cadastrado direto na aba **Configurações** — não precisa de redeploy de código pra atualizar.

## Fluxo de deploy (igual aos apps irmãos)

1. Editar `apps/torneio-open/index.html` (fonte de trabalho).
2. Copiar pra `apps/torneio-open-repo/index.html` (clone do repo).
3. `git add . && git commit -m "..." && git push` dentro de `apps/torneio-open-repo/`.
4. GitHub Pages já publica automaticamente em `https://gubhoewell.github.io/torneio-open-point43/`.
