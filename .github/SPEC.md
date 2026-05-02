# Motor de Distribuição — Spec v5.0

## Visão Geral

Portal de parametrização de regras de distribuição de carteiras/leads para funcionais do Itaú. O usuário configura regras que determinam para quais funcionais e canais os leads e ofertas serão distribuídos, com suporte a condições SE/SENÃO por funcional específico.

**Stack:** HTML + CSS (custom properties) + JS vanilla. Arquivo único (`index.html`), sem framework, sem build step.

**Deploy:** GitHub Pages — `https://lucasverissimo222.github.io/motor-distribuicao/`

**Paleta:** laranja `#F97316`, sidebar branca `#FFFFFF`, fundo creme `#FDF8F3`.

---

## Estrutura de Navegação

```
Sidebar (fixa, 240px)
├── Principal
│   ├── Dashboard               (stub)
│   ├── Motor de Distribuição   → view-main
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

| Campo      | Tipo     | Obrigatório | Valores possíveis                    |
| ---------- | -------- | ----------- | ------------------------------------ |
| Ator/Canal | dropdown | Sim         | Iclientes, ION, GM, Central          |
| Modelo     | dropdown | Sim         | IAD, FISICA, IUD, SD, SDEMP, CONECTA |
| Plataforma | number   | Não         | Filtro de escopo (opcional)          |
| Destino    | checkbox | Sim (≥1)    | Lead, Oferta (pelo menos um marcado) |

**Regras de habilitação:**

- Modelo fica desabilitado até Ator ser selecionado.
- Botão `+` fica desabilitado até Ator **e** Modelo estarem selecionados.
- Destino exige ao menos uma opção marcada; se o usuário desmarcar ambas, a última desmarcada é reativada automaticamente.

---

### Card 2 — Regras de Distribuição

Filtros opcionais que definem o escopo da regra.

| Campo        | Tipo   | Autocomplete via datalist |
| ------------ | ------ | ------------------------- |
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

1. Usuário clica em **SE** para expandir o painel.
2. Seleciona o ID da regra existente no dropdown.
3. Preenche a condição inicial (campos: Funcionais sep. por vírgula, Carteira, Plataforma, Produto).
4. Clica **+ E** ou **+ OU** para encadear novas linhas.
5. Clica **Confirmar condições** para gravar.

Resultado na tabela: linha **SE** (fundo âmbar) e linha **SENÃO** (fundo verde).

---

### Card 3 — Regras Cadastradas

**Colunas:** `# | Ator | Modelo | Plataforma | Destino | Dineg | Região | Regional | Plataforma | Coordenadora | Coordenada | Carteira | Produto | Segmento | (ação)`

**Ações:** Lápis `✎` entra em modo edição populando Card 1 e Card 2.

**Pré-populado:** Regra #1 vem com dados de demonstração.

---

## View — Insights (v1)

### Q1 — Performance por Funcional (últimos 30 dias)

Top 10 funcionais com métricas de atuação.

| Coluna                           | Descrição                                       |
| -------------------------------- | ----------------------------------------------- |
| Funcional                        | Código do funcional (monospace)                 |
| Carteira                         | Carteira associada (badge índigo)               |
| Produto predominante na carteira | Produto mais frequente na carteira do funcional |
| Mais vendido (30d)               | Produto mais convertido nos últimos 30 dias     |
| Conversão (30d)                  | Taxa de conversão em % (badge verde)            |
| Leads ativos                     | Quantidade de leads ativos do funcional         |

### Barra de Ação em Lote (Q2 + Q3 compartilham)

| Controle                 | Função                                                   |
| ------------------------ | -------------------------------------------------------- |
| Nova carteira            | Texto livre `01_XXXX`                                    |
| Mudar fila               | Select: Fila 1 / Fila 2                                  |
| Nova posição             | Número                                                   |
| `Aplicar`                | Atualiza fila e posição nas linhas selecionadas in-place |
| Badge de contagem        | Exibe "N selecionados" quando há checkboxes marcados     |
| `📱 Enviar PCM WhatsApp` | Abre modal PCM com mensagem formatada                    |

**Modal PCM WhatsApp:** gera texto com data, ações e lista de CPFs. Botão "Copiar". Fecha com × ou Esc.

### Q2 — Priorização de Clientes

Top 15 clientes (`BEST_CLIENTS`) ordenados por PA decrescente.

| Coluna          | Descrição                                         |
| --------------- | ------------------------------------------------- |
| CPF             | Identificador do cliente (monospace)              |
| Nº Produtos     | Número de produtos contratados                    |
| Valor acumulado | Total investido no banco (R$)                     |
| Pré-aprovado    | Valor PA (R$, verde)                              |
| Fila atual      | Badge de fila (laranja = Fila 1, índigo = Fila 2) |
| Posição         | Posição na fila de atendimento                    |

Checkboxes por linha + select-all para ação em lote.

**Auto-refresh:** indicador "Atualizado às HH:MM · próx. às HH:MM" no header; `setInterval` de 1h relança `_renderClientPrio`.

**⬇ Extrair CSV:** exporta alterações aplicadas via "Aplicar" — CPF, Nova Carteira, Nova Fila, Nova Posição, Data/hora. CSV com BOM UTF-8.

### Q3 — Leads com Risco (5h–24h · Fila 2)

Visão agregada por hora de espera. Uma linha por hora = 20 linhas (5h a 24h).

| Coluna                | Descrição                                                  |
| --------------------- | ---------------------------------------------------------- |
| Horas aguardando      | Faixa horária (badge vermelho)                             |
| Nº de CPFs            | Total de leads nessa faixa na Fila 2                       |
| Carteira predominante | Carteira com maior volume de leads nessa faixa             |
| Leads ativos          | Total de leads ativos na carteira (todas as filas e horas) |

Checkboxes por linha + select-all para ação em lote.

---

## View — Insights v2

**Auto-refresh:** todas as três tabelas (Q1, Q2, Q3 v2) têm `setInterval` de 3600000ms (1h) iniciado em `renderInsightsV2`. Cada render function atualiza seu próprio indicador "Atualizado às HH:MM · próx. atualização às HH:MM" no header do card. Estados confirmados (`_ruleStates`, `_q2v2States`, `_q3v2States`) são preservados — o refresh só re-renderiza os dados.

---

### Q1 v2 — Performance por Funcional + Criar Regra

Mesma tabela do Q1 v1, com coluna extra **Ação** por linha e métricas no topo.

**Header do card:** indicador de refresh + badge âmbar "Conversão: base D-1 · atualiza diariamente" (tooltip explica que leads ativos atualizam a cada hora, conversão é D-1).

**Métricas:**

| Card                   | Descrição                          |
| ---------------------- | ---------------------------------- |
| Total analisado        | 10 (TOP10_FUNC.length)             |
| Conversão alta (≥55%)  | Count de funcionais com conv ≥ 55% |
| Conversão baixa (<40%) | Count de funcionais com conv < 40% |
| Regras criadas         | Count de `_ruleStates` confirmados |

**Botão "+ Criar regra"** por linha abre modal com **3 etapas (stepper visual no topo):**

#### Etapa 1 — Preview

Exibe dois campos de produto com separação visual clara:

- **Produto da carteira:** `produtoPredominante` — exibido em cor secundária como contexto ("onde o funcional está hoje")
- **Melhor produto (30d):** `maisVendido30d` — destaque em laranja `#F97316`
- **Conversão (30d):** taxa de conversão do funcional
- Badge de confirmação ao fundo: "✓ Regra será criada para: [MAIS_VENDIDO]" com fundo laranja

Botão: "Próximo →"

#### Etapa 2 — Tipo de regra

- Radio: **Prioritário** (funcional recebe primeiro, outros também) / **Exclusivo** (apenas esse funcional recebe) / **Teste** (expira automaticamente).
- Campo "Expirar em X dias" aparece somente quando Teste está selecionado.
- Botões: "← Voltar" / "Próximo →"

#### Etapa 3 — Origem dos leads

- Título dinâmico: "De onde vêm os leads de **[MAIS_VENDIDO]**?" — usa o `maisVendido30d`, não o predominante.
- Subtítulo: "Selecione as origens que esse funcional poderá visualizar"
- Mini-tabela: checkbox | Origem | Leads ativos | Região | Dineg
- Primeira linha: "Todas as carteiras" (soma total). Se marcada, desmarca as demais automaticamente.
- 4 carteiras geradas deterministicamente a partir de `maisVendido30d` via `_getOrigemData(produto)`.
- Se outra carteira for marcada, "Todas" é desmarcada.
- Botões: "← Voltar" / "Confirmar regra"

**Ao confirmar:** state grava `{ tipo, expiry, origem, produto: maisVendido30d }`. Badge verde na linha:
`✓ Regra criada — [Tipo] · [Produto] · [Origem]`
(ex: "Regra criada — Prioritário · LCI · 01_4094" ou "Regra criada — Exclusivo · CDC · Todas")

**"Aplicar todos":** cria regras Prioritárias com origem "Todas" e `produto = maisVendido30d` para todos os funcionais sem regra.

**Modal:** fecha com Esc, click no overlay ou × no header.

---

### Q2 v2 — Priorização de Clientes (VIP Score · Fila 2)

Mostra clientes estáticos da Fila 2 (`Q2_DATA`), filtrados pelo threshold de PA + piso de VIP Score, ordenados por **VIP Score decrescente**.

**Fórmula VIP Score:**
```
vipScore = (pa × 0.5) + (valor × 0.3) + (prods × 0.1) + (horasNaFila × 0.1)
```
Exibido como número inteiro. Para clientes não-Crédito, `pa = 0` — pontuam apenas por valor, produtos e urgência.

**Regra de exibição:**
```
mostrar se: vipScore >= 100000 E (cesta !== 'Crédito' OU pa > threshold_ativo)
```
Clientes não-Crédito sempre passam no filtro de cesta (pa = 0 por definição), mas ainda precisam de vipScore ≥ 100000.

**Dropdown de threshold:**

| Opção   | Corte de PA          |
| ------- | -------------------- |
| Top 5%  | PA > R$200k          |
| Top 10% | PA > R$100k (padrão) |
| Top 20% | PA > R$50k           |

**Colunas:** CPF | Cesta | Produto | Nº Produtos | Valor Acumulado | PA | VIP Score | Hrs na fila | Fila atual | Ação sugerida

**Badge de Cesta:**

| Cesta        | Cor    |
| ------------ | ------ |
| Crédito      | Verde  |
| Investimento | Azul   |
| Conveniência | Âmbar  |
| Recuperação  | Cinza  |

**Coluna PA:** exibe valor em verde para Crédito; exibe "—" para não-Crédito (pa = 0).

**Badge VIP Score:** verde se top 5% · âmbar se top 20% · sem badge abaixo disso (percentis calculados sobre todos os 7 registros de `Q2_DATA`).

**Badge Hrs na fila:** vermelho se > 20h · âmbar se > 8h · cinza se ≤ 8h.

**Lógica de ação (em ordem de prioridade):**

| Condição                            | Ação                      | Cor      |
| ----------------------------------- | ------------------------- | -------- |
| VIP top 5% **e** hrs > 20h          | "Mover fila 1 urgente"    | Vermelho |
| VIP top 5%                          | "Mover fila 1"            | Azul     |
| VIP top 20% **e** hrs > 20h         | "Disparar PCM"            | Âmbar    |
| VIP top 20%                         | "Subir posição"           | Verde    |
| Demais                              | "Monitorar" (texto, sem botão) | —   |

Linha não some ao confirmar — badge de ação confirmada permanece. Estado keyed por CPF; troca de threshold preserva ações já aplicadas.

**"Aplicar todos":** confirma todas as linhas visíveis com ação disponível.

**Métricas — todas contam apenas o conjunto visível no threshold atual:**

| Card                  | Descrição                                       |
| --------------------- | ----------------------------------------------- |
| Total no threshold    | visible.length                                  |
| VIP Top 5%            | visible com vipScore ≥ thresh5                  |
| Urgente (hrs > 20h)   | visible com horasNaFila > 20                    |
| Candidatos fila 1     | visible com vipScore ≥ thresh5 (igual VIP Top 5%) |

**Nota visual** abaixo da tabela: "Clientes sem PA (Investimento, Conveniência, Recuperação) pontuam apenas por valor acumulado, produtos e urgência."

**Dados fake (`Q2_DATA`):**

```js
[
  { cpf:'***.456.789-**', cesta:'Crédito',      produto:'Crédito Pessoal', prods:7, valor:1200000, pa:280000, posicao:1240, horasNaFila:22 },
  { cpf:'***.123.456-**', cesta:'Crédito',      produto:'Consórcio',       prods:6, valor:890000,  pa:195000, posicao:980,  horasNaFila:18 },
  { cpf:'***.789.012-**', cesta:'Investimento', produto:'LCI',             prods:5, valor:540000,  pa:0,      posicao:612,  horasNaFila:9  },
  { cpf:'***.321.654-**', cesta:'Crédito',      produto:'CDC',             prods:4, valor:320000,  pa:88000,  posicao:380,  horasNaFila:6  },
  { cpf:'***.654.321-**', cesta:'Conveniência', produto:'Seguro Vida',     prods:3, valor:210000,  pa:0,      posicao:290,  horasNaFila:14 },
  { cpf:'***.111.222-**', cesta:'Crédito',      produto:'Crédito Pessoal', prods:2, valor:180000,  pa:38000,  posicao:510,  horasNaFila:21 },
  { cpf:'***.333.444-**', cesta:'Investimento', produto:'CDB',             prods:2, valor:120000,  pa:0,      posicao:850,  horasNaFila:11 },
]
```

---

### Q3 v2 — Saúde das Carteiras

Análise de capacidade vs demanda por carteira na Fila 2, com urgência temporal. 6 linhas estáticas (`Q3_DATA`).

**Fórmulas:**
```
capDiaria     = Math.round(atuados2d / 2)
tempoRestante = 48 - horasMaisAntigo

Status (primeira regra que bater, ganha):
  Crítico → capDiaria × 3 < aguardando  OU  tempoRestante < 8
  Atenção → capDiaria × 3 >= aguardando  E  8 ≤ tempoRestante ≤ 24
  OK      → capDiaria × 3 >= aguardando  E  tempoRestante > 24
```

**Colunas:** Carteira | Aguardando | Atuados 2d | Cap. diária | Lead + antigo | Tempo restante | Status | Ação

**Badge "Lead + antigo"** (`horasMaisAntigo`):

| Condição             | Cor     | Texto       |
| -------------------- | ------- | ----------- |
| horasMaisAntigo > 40 | Vermelho | "Xh na fila" |
| horasMaisAntigo > 24 | Âmbar   | "Xh na fila" |
| ≤ 24                 | Cinza   | "Xh na fila" |

**Badge "Tempo restante"** (`48 - horasMaisAntigo`):

| Condição            | Cor    | Texto          |
| ------------------- | ------ | -------------- |
| tempoRestante < 8   | Vermelho | "Xh restantes" |
| tempoRestante < 24  | Âmbar  | "Xh restantes" |
| ≥ 24                | Verde  | "Xh restantes" |

**Ações por status:**

| Status  | Ação                                                                |
| ------- | ------------------------------------------------------------------- |
| OK      | "Nenhuma" (texto)                                                   |
| Atenção | Badge "Uber-like — em desenvolvimento" (sem botão)                  |
| Crítico | Botão "Disparar PCM" + badge "Uber-like — em desenvolvimento"       |

Ao confirmar Crítico: badge "✓ PCM disparado" + "Uber-like — em desenvolvimento".

**"Aplicar todos":** confirma apenas linhas com status Crítico; ignora Atenção e OK.

**Métricas:**

| Card              | Descrição                         |
| ----------------- | --------------------------------- |
| Total monitorado  | Q3_DATA.length (6)                |
| Crítico           | Count status === 'critico'        |
| Atenção           | Count status === 'atencao'        |
| Expirando em 8h   | Count tempoRestante < 8 (vermelho) |

**Dados fake (`Q3_DATA`):**

```js
[
  { cart:'01_5412', aguardando:1240, atuados2d:12,  horasMaisAntigo:44 },
  { cart:'01_2387', aguardando:960,  atuados2d:420, horasMaisAntigo:38 },
  { cart:'01_9001', aguardando:780,  atuados2d:680, horasMaisAntigo:18 },
  { cart:'01_1122', aguardando:620,  atuados2d:55,  horasMaisAntigo:31 },
  { cart:'01_3344', aguardando:440,  atuados2d:390, horasMaisAntigo:12 },
  { cart:'01_7788', aguardando:360,  atuados2d:310, horasMaisAntigo:6  },
]
```

---

## Bases de Dados (Modal — Sidebar Gestão)

| Base             | Registros | Campos principais                                           |
| ---------------- | --------- | ----------------------------------------------------------- |
| Tabela Dineg     | 100       | Dineg, Região, Regional, Plataforma, Coordenadora, Carteira |
| Base Coordenadora| variável  | Código, Coordenadora, Coordenada, Carteira, Modelo          |
| Base Funcional   | variável  | Funcional, Carteira                                         |
| Lead / Oferta    | 500       | Ver schema abaixo                                           |

**Schema `BASE_LEADS`:**

```js
{
  produto, cpf, carteira, plataforma, segmento,
  fila,       // 1 ou 2
  prioridade, // 1–50
  valor,      // valor acumulado (R$)
  nProdutos,  // 1–8
  pa,         // pré-aprovado fake; (50 + i*31%950)*1000
  ativo,      // 1 = ativo (~80%); 0 = já atuado
  horasNaFila,
  dataCriacao
}
```

Todos os dados gerados deterministicamente via progressões aritméticas — sem `Math.random()`.

---

## Constantes Derivadas

| Constante           | Tamanho | Origem / Uso                                      |
| ------------------- | ------- | ------------------------------------------------- |
| `BASE_DINEG`        | 100     | Gerada; autocomplete dos campos de Dineg          |
| `BASE_COORD`        | ~20     | Gerada; autocomplete de coordenadora/carteira     |
| `BASE_FUNC`         | ~200    | Gerada; autocomplete de funcional                 |
| `BASE_LEADS`        | 500     | Gerada; base principal de leads                   |
| `TOP10_FUNC`        | 10      | Derivado de BASE_LEADS × BASE_FUNC — Q1 v1 e v2  |
| `BEST_CLIENTS`      | 15      | Top 15 de BASE_LEADS por pa desc — Q2 v1          |
| `LEADS_HOURS_COUNT` | 20      | Contagem por hora (5h–24h) — Q3 v1               |
| `Q2_DATA`           | 7       | Estático — clientes Fila 2 com VIP Score — Q2 v2  |
| `Q3_DATA`           | 6       | Estático — carteiras com cap/demanda — Q3 v2      |
| `PRODUTOS_LIST`     | 10      | Nomes de produtos para geração determinística     |

---

## Estado Global (variáveis JS)

| Variável           | Tipo   | Descrição                                              |
| ------------------ | ------ | ------------------------------------------------------ |
| `_insightsReady`   | bool   | Flag lazy-init Insights v1                             |
| `_insightsV2Ready` | bool   | Flag lazy-init Insights v2 (setIntervals iniciados aqui) |
| `_q2Changes`       | array  | Log de alterações Q2 v1 para exportação CSV            |
| `_q2v2Threshold`   | number | Percentil ativo no Q2 v2 (5, 10 ou 20)                |
| `_ruleModalRow`    | number | Índice da linha em edição no modal de regra Q1 v2      |
| `_ruleModalStep`   | number | Etapa ativa no modal de regra (1, 2 ou 3)              |
| `_ruleStates`      | object | Regras confirmadas em Q1 v2 — `{ tipo, expiry, origem, produto }` keyed por índice |
| `_q2v2States`      | object | Ações aplicadas em Q2 v2 — `{ confirmed, done, bg, c }` keyed por CPF |
| `_q3v2States`      | object | Estado por linha Q3 v2 — `{ confirmed }` keyed por índice |

---

## Pendências / Próximos Passos

- [ ] Integração com back-end (endpoints de cadastro, listagem e exclusão de regras)
- [ ] Campo Pré-aprovado com dado real do **Raiox**
- [ ] Validação de carteira vs plataforma no back-end
- [ ] Paginação da tabela de regras cadastradas (Card 3)
- [ ] Edição/exclusão de regras via back-end
- [ ] Exportação CSV das regras do Motor
- [ ] Dados reais em Q1, Q2, Q3 v1 e v2 (atualmente 100% fake)
- [ ] Ação "Aplicar" em lote deve persistir via API
- [ ] Envio real do PCM WhatsApp
- [ ] Confirmação de regra no Q1 v2 deve gravar via API
- [ ] Autenticação / controle de acesso por perfil
