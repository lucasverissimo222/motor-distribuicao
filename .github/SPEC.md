# Motor de Distribuição — Spec v3.0

## Visão Geral

Portal de parametrização de regras de distribuição de carteiras/leads para funcionais do Itaú. O usuário configura regras que determinam para quais funcionais e canais os leads e ofertas serão distribuídos, com suporte a condições SE/SENÃO por funcional específico.

**Stack:** HTML + CSS (custom properties) + JS vanilla. Arquivo único (`index.html`), sem framework, sem build step.

**Deploy:** GitHub Pages — `https://lucasverissimo222.github.io/motor-distribuicao/`

**Paleta:** laranja claro `#F97316`, sidebar branca `#FFFFFF`, fundo creme `#FDF8F3`.

---

## Estrutura de Navegação

```
Sidebar (fixa, 240px)
├── Principal
│   ├── Dashboard               (stub)
│   ├── Motor de Distribuição   → view-main (ativo)
│   ├── Regras Ativas           (stub)
│   ├── Insights                → view-insights
│   └── Insights v2             → view-insights-v2
├── Gestão (bases em modal)
│   ├── Tabela Dineg
│   ├── Base Coordenadora
│   ├── Base Funcional
│   └── Lead / Oferta
└── Sistema
    ├── Configurações           (stub)
    └── Suporte                 (stub)
```

Três views reais, trocadas via `showView('main' | 'insights' | 'insights-v2')`. O título do topbar muda de acordo.

---

## View — Motor de Distribuição

### Card 1 — Ator e Modelo

| Campo      | Tipo     | Obrigatório | Valores possíveis                        |
|------------|----------|-------------|------------------------------------------|
| Ator/Canal | dropdown | Sim         | Iclientes, ION, GM, Central              |
| Modelo     | dropdown | Sim         | IAD, FISICA, IUD, SD, SDEMP, CONECTA    |
| Plataforma | number   | Não         | Filtro de escopo (opcional)              |
| Destino    | checkbox | Sim (≥1)    | Lead, Oferta (pelo menos um marcado)     |

**Regras de habilitação:**
- Modelo fica desabilitado até Ator ser selecionado.
- Botão `+` fica desabilitado até Ator **e** Modelo estarem selecionados.
- Destino exige ao menos uma opção marcada; se o usuário desmarcar ambas, a última desmarcada é reativada automaticamente.

---

### Card 2 — Regras de Distribuição

Filtros opcionais que definem o escopo da regra.

| Campo        | Tipo   | Autocomplete via datalist |
|--------------|--------|---------------------------|
| Dineg        | number | Sim (BASE_DINEG)          |
| Região       | number | Sim (BASE_DINEG)          |
| Regional     | number | Sim (BASE_DINEG)          |
| Plataforma   | number | Sim (BASE_DINEG)          |
| Coordenadora | number | Sim (BASE_COORD)          |
| Coordenada   | number | Sim (BASE_COORD)          |
| Carteira     | text   | Sim (BASE_COORD/FUNC)     |
| Produto      | text   | Não                       |
| Segmento     | number | Não                       |

**Botões do card:**
- `Limpar campos` — zera todos os inputs do Card 2.
- `Cancelar edição` — visível apenas em modo edição; descarta a edição.
- `+` — adiciona a regra na tabela de Regras Cadastradas (Card 3).

---

### Bloco SE / SENÃO (dentro do Card 2)

**Fluxo:**
1. Usuário clica em **SE** para expandir o painel.
2. Seleciona o ID da regra existente no dropdown.
3. Preenche a condição inicial (campos: Funcionais sep. por vírgula, Carteira, Plataforma, Produto).
4. Clica **+ E** ou **+ OU** para encadear novas linhas.
5. Clica **Confirmar condições** para gravar.

**Resultado na tabela:**
- Linha **SE** (fundo âmbar) — aplica quando a condição é verdadeira.
- Linha **SENÃO** (fundo verde) — aplica quando a condição é falsa.

---

### Card 3 — Regras Cadastradas

**Colunas:** `# | Ator | Modelo | Plataforma | Destino | Dineg | Região | Regional | Plataforma | Coordenadora | Coordenada | Carteira | Produto | Segmento | (ação)`

**Ações:** Lápis `✎` entra em modo edição populando Card 1 e Card 2.

**Pré-populado:** Regra #1 vem com dados de demonstração.

---

## View — Insights (v1)

Página analítica com dados da base D-1.

### Q1 — Performance por Funcional (últimos 30 dias)

Top 10 funcionais com métricas de atuação.

| Coluna                           | Descrição                                       |
|----------------------------------|-------------------------------------------------|
| Funcional                        | Código do funcional (monospace)                 |
| Carteira                         | Carteira associada (badge índigo)               |
| Produto predominante na carteira | Produto mais frequente na carteira do funcional |
| Mais vendido (30d)               | Produto mais convertido nos últimos 30 dias     |
| Conversão (30d)                  | Taxa de conversão em % (badge verde)            |
| Leads ativos                     | Quantidade de leads ativos do funcional         |

---

### Barra de Ação em Lote (compartilhada Q2 + Q3)

Posicionada entre Q1 e Q2.

| Controle                 | Função                                                   |
|--------------------------|----------------------------------------------------------|
| Nova carteira            | Texto livre `01_XXXX`                                    |
| Mudar fila               | Select: Fila 1 / Fila 2                                  |
| Nova posição             | Número                                                   |
| `Aplicar`                | Atualiza fila e posição nas linhas selecionadas in-place |
| Badge de contagem        | Exibe "N selecionados" quando há checkboxes marcados     |
| `📱 Enviar PCM WhatsApp` | Abre modal PCM com mensagem formatada                    |

**Modal PCM WhatsApp:** gera texto com data, ações e lista de CPFs. Botão "Copiar". Fecha com × ou Esc.

---

### Q2 — Priorização de Clientes

Top 15 clientes ordenados por **valor PA decrescente**.

| Coluna          | Descrição                                          |
|-----------------|----------------------------------------------------|
| CPF             | Identificador do cliente (monospace)               |
| Nº Produtos     | Número de produtos contratados                     |
| Valor acumulado | Total investido no banco (R$)                      |
| Pré-aprovado    | Valor PA (R$, verde, ordenação principal)          |
| Fila atual      | Badge de fila (laranja = Fila 1, índigo = Fila 2) |
| Posição         | Posição na fila de atendimento                     |

Checkboxes por linha + select-all para ação em lote.

**Auto-refresh:** indicador "Atualizado às HH:MM · próx. às HH:MM" no header; `setInterval` de 1h relança `_renderClientPrio`.

**⬇ Extrair CSV:** exporta todas as alterações aplicadas via "Aplicar" com colunas: CPF, Nova Carteira, Nova Fila, Nova Posição, Data/hora. CSV com BOM UTF-8.

> **Pré-aprovado** vem do sistema **Raiox** (integração pendente — dado fake na versão atual).

---

### Q3 — Leads com Risco (5h–24h · Fila 2)

Visão agregada por hora de espera. Uma linha por hora = 20 linhas (5h a 24h).

| Coluna                | Descrição                                                          |
|-----------------------|--------------------------------------------------------------------|
| Horas aguardando      | Faixa horária (badge vermelho)                                     |
| Nº de CPFs            | Total de leads nessa faixa na Fila 2                               |
| Carteira predominante | Carteira com maior volume de leads nessa faixa                     |
| Leads ativos          | Total de leads ativos na carteira (todas as filas e horas)         |

> **Lógica de decisão:** `Leads ativos ≈ Nº de CPFs` → balde quase esgotado, menos urgência. `Leads ativos >> Nº de CPFs` → há volume, ação recomendada.

Checkboxes por linha + select-all para ação em lote.

---

## View — Insights v2

Análise avançada de risco com scoring por pressão de carteira.

---

### Q1 v2 — Performance por Funcional + Criar Regra

Mesma tabela do Q1 v1, com coluna extra **Ação** por linha.

**Botão "+ Criar regra"** por linha abre modal com:
- Preview pré-preenchido: funcional, carteira, produto, conversão.
- Radio buttons — tipo de regra:
  - **Prioritário** — funcional recebe primeiro, outros também podem.
  - **Exclusivo** — apenas esse funcional recebe esse produto.
  - **Teste** — expira automaticamente (campo "Expirar em X dias" aparece).
- Botões: Cancelar / Confirmar regra.
- Ao confirmar: botão vira badge verde "✓ [Tipo]" (ou "✓ Teste · Xd").

**"Aplicar todos"** cria regras Prioritárias para todos os funcionais sem regra.

**Modal:** fecha com Esc, click no overlay ou botão Cancelar.

**Métricas no topo:**

| Card                   | Descrição                              |
|------------------------|----------------------------------------|
| Total analisado        | 10 (TOP10_FUNC.length)                 |
| Conversão alta (≥55%)  | Count de funcionais com conv ≥ 55%     |
| Conversão baixa (<40%) | Count de funcionais com conv < 40%     |
| Regras criadas         | Count de `_ruleStates` confirmados     |

---

### Q2 v2 — Priorização de Clientes (só Fila 2, threshold dinâmico)

Mostra **apenas clientes da Fila 2** (`Q2_DATA` — 7 linhas estáticas), filtrados pelo corte de PA do dropdown. Clientes abaixo do threshold somem da tabela ao trocar o percentil.

**Colunas:** CPF | Nº Produtos | Valor acumulado | PA | Posição fila 2 | Ação sugerida

**Dropdown no header:**

| Opção   | Corte de PA  |
|---------|--------------|
| Top 5%  | PA > R$200k  |
| Top 10% | PA > R$100k (padrão) |
| Top 20% | PA > R$50k   |

**Zonas de decisão por linha (avaliadas em ordem):**

| Zona    | Condição                         | Botão                 | Badge ao confirmar        |
|---------|----------------------------------|-----------------------|---------------------------|
| fila1   | PA > R$200k                      | "Mover para fila 1"  | azul "Movido para fila 1" |
| subir   | PA > R$100k **e** posição > 500  | "Subir posição"       | verde "Posição atualizada"|
| pcm     | PA > R$50k **e** posição > 1000  | "Disparar PCM"        | verde "PCM disparado"     |
| monitor | demais casos                     | — (texto "Monitorar") | —                         |

Linha **não some** ao confirmar — mantém registro da ação tomada. Estado keyed por CPF — troca de percentil preserva ações já aplicadas.

**"Aplicar todos"** confirma todas as linhas com botão ativo no threshold atual (ignora "Monitorar").

**Métricas no topo:**

| Card                       | Descrição                                             |
|----------------------------|-------------------------------------------------------|
| Total no threshold         | Clientes acima do corte de PA                         |
| Candidatos a fila 1        | Count de Q2_DATA com PA > R$200k (absoluto)           |
| Em risco de expirar (pos>1k)| Count de Q2_DATA com posição > 1000 (absoluto)       |
| PA total em risco          | Soma de PA dos registros com posição > 1000           |

**Dados fake (`Q2_DATA`) — todos Fila 2:**
```js
[
  {cpf:'***.456.789-**', prods:7, valor:1200000, pa:280000, posicao:1240},
  {cpf:'***.123.456-**', prods:6, valor:890000,  pa:195000, posicao:980},
  {cpf:'***.789.012-**', prods:5, valor:540000,  pa:120000, posicao:612},
  {cpf:'***.321.654-**', prods:4, valor:320000,  pa:88000,  posicao:380},
  {cpf:'***.654.321-**', prods:3, valor:210000,  pa:45000,  posicao:290},
  {cpf:'***.111.222-**', prods:2, valor:180000,  pa:38000,  posicao:510},
  {cpf:'***.333.444-**', prods:2, valor:120000,  pa:25000,  posicao:850}
]
```

---

### Q3 v2 — Leads em Risco (Pressão / Score / Zonas)

6 linhas com dados estáticos deterministicos (`Q3_DATA`).

**Fórmulas (exatas no código):**
```js
pressao       = aguardando / Math.max(atuados2d, 1)
score_raw     = Math.min(1 / pressao, 1) * 0.6 + conv * 0.4
score_display = Math.round(score_raw * 100)  // 0–100
```

**Zonas (avaliadas em ordem — primeira que bater, ganha):**

| Zona     | Condição                        | Badge                    | Ação disponível          |
|----------|---------------------------------|--------------------------|--------------------------|
| Roxa     | pressao > 10 **e** conv > 0.35  | "Segurar — buffer"       | Nenhuma (buffer auto)    |
| Vermelha | pressao > 10                    | "PCM urgente"            | Botão "Disparar PCM"     |
| Verde    | pressao < 3 **e** conv > 0.30   | "Ativar uber-like"       | Botão "Ativar uber-like" |
| Amarela  | demais casos                    | "Monitorar"              | Botão "Marcar monitorado"|

**Buffer (zona roxa):**
- Linha marcada com 🔒 Em buffer automaticamente.
- Quando pressão cair abaixo de 3 (verificado a cada `_renderQ3v2`): zona muda para verde, botão "Ativar uber-like" aparece.

**Baixa confiança na conversão:**
- Se `recebidos < 100`: ícone ⚠ ao lado da conversão com tooltip "Volume baixo — dado pode não ser representativo".
- Não altera o cálculo do score.

**Score visual:** ≥70 → verde · 45–69 → âmbar · <45 → vermelho.

**Limite uber-like:**
- Contador global `uberlikeUsados`, cap 200 por ciclo (reset ao recarregar).
- Se `uberlikeUsados >= 200`: botões verdes exibem badge âmbar "Limite do ciclo atingido".

**"Aplicar todos":** dispara ações para todas as linhas com botão ativo (ignora roxa e limite atingido).

**Métricas no topo:**

| Card                         | Descrição                                |
|------------------------------|------------------------------------------|
| Total em risco               | 6 (Q3_DATA.length)                       |
| Zona roxa — segurar          | Count zona === 'roxa'                    |
| Zona verde — uber-like disp. | Count zona === 'verde'                   |
| Uber-likes restantes         | `200 − uberlikeUsados` (exibe X/200)     |

**Colunas da tabela:**
`Horas | Aguardando | Carteira | Funcional | Atuados 2d | Pressão / tend. | Conversão | Score | Zona | Ação`

**Dados fake (`Q3_DATA`):**
```js
[
  {hrs:24, cart:'01_5412', func:'102938475', aguardando:1240, atuados2d:12,  conv:0.48, recebidos:320, trend:'up'},
  {hrs:20, cart:'01_2387', func:'987654321', aguardando:960,  atuados2d:420, conv:0.41, recebidos:890, trend:'flat'},
  {hrs:16, cart:'01_9001', func:'112233445', aguardando:780,  atuados2d:680, conv:0.37, recebidos:45,  trend:'down'},
  {hrs:12, cart:'01_1122', func:'556677889', aguardando:620,  atuados2d:55,  conv:0.29, recebidos:210, trend:'up'},
  {hrs:8,  cart:'01_3344', func:'667788990', aguardando:440,  atuados2d:390, conv:0.24, recebidos:180, trend:'flat'},
  {hrs:5,  cart:'01_7788', func:'334455667', aguardando:360,  atuados2d:310, conv:0.20, recebidos:95,  trend:'down'}
]
// recebidos < 100 → rows idx 2 (recebidos:45) e idx 5 (recebidos:95) → flag ⚠
```

---

## Bases de Dados (Modal — Sidebar Gestão)

### Tabela Dineg
100 registros fake: Dineg, Região, Regional, Plataforma, Coordenadora, Coordenada, Carteira, Segmento.

### Base Coordenadora
Campos: Código, Coordenadora, Coordenada, Carteira, Modelo.

### Base Funcional
Campos: Funcional, Carteira.

### Lead / Oferta
500 registros.

| Campo      | Descrição                                   |
|------------|---------------------------------------------|
| Produto    | Produto do lead                             |
| CPF        | Identificador do cliente                    |
| Carteira   | Carteira atribuída (`01_XXXX`)              |
| Plataforma | Código da plataforma (6001–6399)            |
| Segmento   | Segmento do cliente (1–5)                   |
| Fila       | Fila atual (1 ou 2)                         |
| Posição    | Posição na fila de atendimento              |
| Hrs. fila  | Horas aguardando atendimento                |
| Ativo      | 1 = lead ativo (não atuado), 0 = já atuado |

---

## Dados Fake (Desenvolvimento)

Todos gerados deterministicamente via progressões aritméticas — sem `Math.random()`, reproduzíveis.

| Constante           | Tamanho  | Descrição                                             |
|---------------------|----------|-------------------------------------------------------|
| `BASE_DINEG`        | 100      | Hierarquia organizacional                             |
| `BASE_COORD`        | variável | Coordenadoras com modelo atribuído                    |
| `BASE_FUNC`         | variável | Funcionais com carteira                               |
| `BASE_LEADS`        | 500      | Leads ativos — todos os campos + `pa`, `ativo`        |
| `TOP10_FUNC`        | 10       | Derivado de BASE_LEADS × BASE_FUNC para Q1            |
| `BEST_CLIENTS`      | 15       | Top 15 de BASE_LEADS por `pa` desc — Q2 v1            |
| `LEADS_HOURS_COUNT` | 20       | Contagem agregada de leads por hora (5h–24h) — Q3 v1  |
| `Q2_DATA`           | 7        | Clientes estáticos Fila 2 com PA e posição — Q2 v2    |
| `Q3_DATA`           | 6        | Dados estáticos de risco por carteira — Q3 v2         |

**Campos de `BASE_LEADS`:**
```js
{
  produto,       // string
  cpf,           // string 11 dígitos
  carteira,      // string '01_XXXX'
  plataforma,    // number 6001–6399
  segmento,      // number 1–5
  fila,          // number 1 ou 2
  prioridade,    // number 1–50
  valor,         // number — valor acumulado (R$)
  nProdutos,     // number 1–8
  pa,            // number — pré-aprovado Raiox (fake); (50 + i*31%950)*1000
  ativo,         // number 0 ou 1; (i*7%5)!==0 ? 1 : 0  (~80% ativos)
  horasNaFila,   // number 1–72
  dataCriacao    // string pt-BR
}
```

---

## Estado Global (variáveis JS)

| Variável           | Tipo   | Descrição                                              |
|--------------------|--------|--------------------------------------------------------|
| `_insightsReady`   | bool   | Flag lazy-init Insights v1                             |
| `_insightsV2Ready` | bool   | Flag lazy-init Insights v2                             |
| `_q2Changes`       | array  | Log de alterações do Q2 v1 para exportação CSV         |
| `_q2v2Threshold`   | number | Percentil ativo no Q2 v2 (5, 10 ou 20)                |
| `_ruleModalRow`    | number | Índice da linha em edição no modal de regra            |
| `_ruleStates`      | object | Regras confirmadas em Q1 v2 (keyed por índice)         |
| `_q2v2States`      | object | Ações aplicadas em Q2 v2 (keyed por CPF)               |
| `_q3v2States`      | object | Estado por linha Q3 v2 (buffer, confirmed, action)     |
| `uberlikeUsados`   | number | Contador global de uber-likes neste ciclo (cap 200)    |

---

## Pendências / Próximos Passos

- [ ] Integração com back-end (endpoints de cadastro, listagem e exclusão de regras)
- [ ] Campo `Pré-aprovado` com dado real do **Raiox**
- [ ] Campo `Ativo` com dado real do **Raiox**
- [ ] Validação de carteira vs plataforma no back-end
- [ ] Paginação da tabela de regras cadastradas (Card 3)
- [ ] Edição/exclusão de regras via back-end
- [ ] Exportação CSV das regras do Motor
- [ ] Dados reais em Q1, Q2, Q3 v1 e v2 (atualmente 100% fake)
- [ ] Ação "Aplicar" em lote deve persistir via API
- [ ] Envio real do PCM WhatsApp
- [ ] Confirmação de regra no Q1 v2 deve gravar via API
- [ ] Persistência do `uberlikeUsados` entre sessões
- [ ] Reset controlado do ciclo uber-like
- [ ] Autenticação / controle de acesso por perfil
