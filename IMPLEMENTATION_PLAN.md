# Plano de Implementacao - UI First

## Estrategia

Construir toda a UI com dados mockados (sem backend) para validar a experiencia
antes de conectar Supabase. A treinadora podera navegar por todas as telas,
preencher formularios, ver calculos em tempo real, e visualizar resultados.

Dados ficam em estado local (React state/context). Backend vem depois.

---

## Componentes shadcn/ui utilizados

| Componente | Onde |
|-----------|------|
| Button | Toda a app (CTAs, navegacao) |
| Input | Campos numericos (peso, medidas, dobras) |
| Label | Todos os formularios |
| Card | Cards de alunos, cards de metricas no resultado |
| Badge | Classificacoes (IMC, %G, risco) com cores |
| Avatar | Foto do aluno na lista e ficha |
| Sheet | Menu lateral mobile (hamburger) |
| Dialog | Confirmacoes, alertas PAR-Q |
| Select | Sexo, nivel de atividade, objetivo |
| RadioGroup | PAR-Q (sim/nao), severidade postural |
| Checkbox | Historico medico (condicoes) |
| Textarea | Observacoes, texto livre |
| Tabs | Vistas posturais (anterior/posterior/lateral) |
| Progress | Barra de progresso do wizard |
| Separator | Divisores entre secoes |
| Switch | Toggles (protocolo 3/7 dobras) |
| Table | Tabela comparativa de evolucao |
| Slider | Nivel de estresse (1-5) |
| Alert | Alerta do PAR-Q, mensagens de risco |
| Tooltip | Dicas sobre como medir |
| Collapsible | Secoes expandiveis na anamnese |
| ScrollArea | Listas longas mobile |
| Skeleton | Loading states |
| Sonner/Toast | Feedback (avaliacao salva, etc.) |

---

## Etapas de Implementacao

### Etapa 1 - Setup do Projeto
**Objetivo**: Projeto rodando com Next.js + shadcn/ui + estrutura base

1. `npx create-next-app@latest . --typescript --tailwind --app --src-dir --no-import-alias`
2. `npx shadcn@latest init` (tema: zinc, estilo: new-york)
3. Instalar componentes shadcn necessarios (lista acima)
4. Instalar dependencias: `react-hook-form`, `zod`, `@hookform/resolvers`, `recharts`, `date-fns`, `lucide-react`
5. Configurar tema de cores customizado (tons de verde/azul fitness)
6. Criar layout raiz mobile-first com max-w-md centralizado (simula app mobile)
7. Criar tipos TypeScript para todo o dominio (aluno, avaliacao, medidas, etc.)
8. Criar dados mockados (3-4 alunos com avaliacoes completas para demo)

**Arquivos criados:**
```
src/app/layout.tsx              # Layout raiz com fonte e providers
src/app/globals.css             # Variaveis CSS do tema
src/lib/types.ts                # Tipos do dominio
src/lib/mock-data.ts            # Dados mockados
src/lib/formulas/pollock.ts     # Formulas Pollock
src/lib/formulas/body.ts        # IMC, RCQ, Siri, TMB
src/lib/formulas/fitness.ts     # Cooper, Karvonen, 1RM
src/lib/formulas/classifications.ts  # Tabelas de classificacao
```

---

### Etapa 2 - Shell do App (Navegacao + Layout)
**Objetivo**: Estrutura de navegacao funcional, mobile-first

**Telas:**

**2a. Layout do App** (`src/app/(app)/layout.tsx`)
- Header fixo: titulo da pagina + botao de menu (Sheet)
- Menu lateral (Sheet): links para Dashboard, Alunos, Perfil
- Bottom navigation bar fixo com 3 itens: Inicio, Alunos, Perfil
- Conteudo com padding e scroll

**2b. Login** (`src/app/page.tsx`)
- Logo do app
- Campos email + senha (Input)
- Botao "Entrar" (Button)
- Link "Criar conta"
- Na V1-UI: clicar "Entrar" vai direto ao dashboard (sem auth real)

**2c. Cadastro** (`src/app/cadastro/page.tsx`)
- Campos: nome, email, senha, CREF
- Botao "Criar conta"
- Na V1-UI: redireciona ao dashboard

---

### Etapa 3 - Dashboard da Treinadora
**Objetivo**: Tela inicial apos login

**Tela: Dashboard** (`src/app/(app)/dashboard/page.tsx`)

Layout:
```
[Boas vindas, {nome}]

[Card: Resumo]
  - Total de alunos ativos: 12
  - Avaliacoes este mes: 5
  - Proximas avaliacoes: lista

[Card: Ultimas avaliacoes]
  - Lista das 5 avaliacoes mais recentes
  - Cada item: avatar + nome + data + badge(status)
  - Click -> vai para resultado da avaliacao

[FAB: + Nova Avaliacao]
```

Componentes: Card, Avatar, Badge, Button (FAB)

---

### Etapa 4 - Gestao de Alunos
**Objetivo**: CRUD visual completo de alunos

**4a. Lista de Alunos** (`src/app/(app)/alunos/page.tsx`)
```
[Input busca com icone lupa]

[Lista de alunos - ScrollArea]
  Cada item:
  [Avatar] [Nome          ] [Badge: ativo/inativo]
           [Ultima aval: 15/01/2026]
           [Proximo: 15/02/2026    ]

[FAB: + Novo Aluno]
```

- Busca filtra em tempo real
- Click no aluno -> ficha do aluno
- Componentes: Input, ScrollArea, Avatar, Badge, Button

**4b. Cadastro/Edicao de Aluno** (`src/app/(app)/alunos/novo/page.tsx`)
```
[Foto - circulo clicavel com icone camera]

[Form]
  Nome completo          [Input]
  Data de nascimento     [Input type=date]
  Sexo                   [Select: Feminino/Masculino]
  Telefone               [Input tel]
  Email                  [Input email]
  Observacoes            [Textarea]

[Button: Salvar Aluno]
```

- Validacao com zod (nome obrigatorio, data valida, sexo obrigatorio)
- Componentes: Input, Select, Textarea, Button, Avatar

**4c. Ficha do Aluno** (`src/app/(app)/alunos/[id]/page.tsx`)
```
[Header com foto grande + nome + idade + sexo]

[Tabs: Avaliacoes | Evolucao]

Tab Avaliacoes:
  [Button: + Nova Avaliacao]
  [Lista de avaliacoes ordenada por data]
    Cada item:
    [Data] [Badge: completa/em andamento]
    [Resumo: peso, %G, IMC]
    Click -> resultado

Tab Evolucao:
  [Graficos de evolucao - Recharts]
  - Peso ao longo do tempo (LineChart)
  - % Gordura ao longo do tempo (LineChart)
  - Circunferencias selecionaveis (LineChart)
```

- Componentes: Tabs, Card, Badge, Button, Avatar

---

### Etapa 5 - Wizard de Avaliacao (Container)
**Objetivo**: Fluxo guiado passo-a-passo que contem todas as etapas

**Tela: Nova Avaliacao** (`src/app/(app)/avaliacao/nova/[alunoId]/page.tsx`)

```
[Header fixo]
  [< Voltar]  [Nome do Aluno]  [Salvar rascunho]

[Progress bar - 6 etapas]
  1.Anamnese  2.Medidas  3.Perimetria  4.Dobras  5.Postural  6.Testes

[Conteudo da etapa atual - scroll]

[Footer fixo]
  [Button ghost: Anterior]  [Etapa 2/6]  [Button: Proximo ->]
```

- Estado global da avaliacao em React Context
- Cada etapa e um componente filho
- Navegacao entre etapas com animacao de slide
- Dados persistem ao navegar entre etapas
- Botao "Salvar rascunho" a qualquer momento
- Componentes: Progress, Button, contexto local

---

### Etapa 6 - Etapa 1 do Wizard: Anamnese
**Objetivo**: Questionario de saude guiado

**Sub-etapas dentro da anamnese (acordeoes expandiveis):**

**6a. PAR-Q**
```
[Alert info: "Responda as perguntas abaixo com o aluno"]

[7 perguntas - RadioGroup Sim/Nao cada]
1. "Algum medico ja disse que voce possui algum problema de coracao..."
2. "Voce sente dores no peito quando pratica atividade fisica?"
3. ...

[Se alguma = Sim -> Alert destructive]
  "Recomende ao aluno buscar liberacao medica antes de iniciar"
```

**6b. Historico Medico**
```
[Collapsible: Condicoes atuais]
  [Checkbox] Hipertensao
  [Checkbox] Diabetes
  [Checkbox] Cardiopatia
  [Checkbox] Asma/problemas respiratorios
  [Checkbox] Problemas articulares
  [Checkbox] Outros: [Input]

[Input: Cirurgias anteriores]
[Input: Medicamentos em uso]
[Input: Lesoes/problemas ortopedicos]

[Collapsible: Historico familiar]
  [Checkbox] Doenca cardiaca
  [Checkbox] Diabetes
  [Checkbox] AVC
  [Checkbox] Hipertensao
```

**6c. Estilo de Vida**
```
Nivel de atividade   [Select: Sedentario/Leve/Moderado/Ativo]
Experiencia treino   [Select: Iniciante/Intermediario/Avancado]
Horas de sono        [Input numerico: 1-12]
Nivel de estresse    [Slider: 1-5 com labels]
Tabagismo            [RadioGroup: Nao/Ex-fumante/Sim]
Alcool               [RadioGroup: Nao/Social/Frequente]
Agua (litros/dia)    [Input numerico]
```

**6d. Objetivos**
```
Objetivo principal   [Select: Emagrecimento/Hipertrofia/Saude/Performance/Reabilitacao]
Objetivos secundarios [Checkbox multiplos]
Dias disponiveis/semana [Select: 1-7]
```

**6e. Saude da Mulher** (condicional: aparece se sexo=F)
```
Gestante             [RadioGroup: Sim/Nao]
Ciclo menstrual      [Select: Regular/Irregular/Ausente]
Anticoncepcional     [RadioGroup: Sim/Nao] -> [Input: Qual?]
```

Componentes: RadioGroup, Checkbox, Select, Slider, Input, Alert, Collapsible, Textarea

---

### Etapa 7 - Etapa 2 do Wizard: Medidas Basicas
**Objetivo**: Peso, altura, IMC calculado em tempo real

```
[Card: Medidas Basicas]
  Peso (kg)    [Input numerico - teclado numerico mobile]
  Altura (cm)  [Input numerico]

  [Separator]

  [Card resultado - aparece quando peso E altura preenchidos]
    IMC: 24.3
    [Badge colorido]: "Normal"
    [Barra visual colorida com marcador da posicao]
      <18.5    18.5-24.9    25-29.9    30+
      Baixo    Normal       Sobrepeso  Obesidade

    Peso ideal estimado: 65-78 kg (baseado na altura)
```

- IMC calcula automaticamente ao digitar
- Badge muda de cor conforme classificacao
- Barra visual mostra onde o aluno esta
- Input com `inputMode="decimal"` para teclado numerico no celular
- Componentes: Card, Input, Badge, Separator, Progress (customizado como barra)

---

### Etapa 8 - Etapa 3 do Wizard: Perimetria (Circunferencias)
**Objetivo**: Registro de todas as circunferencias corporais

```
[Instrucao no topo]
"Use fita metrica flexivel. Mantenha a fita firme sem comprimir a pele."

[Secoes agrupadas - Collapsible]

[Tronco]
  Pescoco (cm)         [Input]
  Ombro (cm)           [Input]
  Torax relaxado (cm)  [Input]
  Torax inspirado (cm) [Input]
  Cintura (cm)         [Input]
  Abdomen (cm)         [Input]
  Quadril (cm)         [Input]

[Membros Superiores]
  [Tabs: Direito | Esquerdo]
  Braco relaxado (cm)  [Input]
  Braco contraido (cm) [Input]
  Antebraco (cm)       [Input]

[Membros Inferiores]
  [Tabs: Direito | Esquerdo]
  Coxa proximal (cm)   [Input]
  Coxa medial (cm)     [Input]
  Panturrilha (cm)     [Input]

[Separator]

[Card resultado - aparece quando cintura E quadril preenchidos]
  RCQ: 0.85
  [Badge]: "Risco Moderado"
  Cintura: 82cm -> [Badge]: "Risco Elevado" (se >80 mulher / >94 homem)
```

- Cada input com `inputMode="decimal"`, placeholder com valor de referencia
- Tooltip com icone (?) em cada campo explicando onde medir
- Auto-calculo RCQ e classificacao de risco
- Componentes: Collapsible, Tabs, Input, Card, Badge, Tooltip

---

### Etapa 9 - Etapa 4 do Wizard: Dobras Cutaneas
**Objetivo**: Protocolo de Pollock com guia visual

```
[Switch: Protocolo]
  [3 dobras (rapido)]  [7 dobras (completo)]

[Info do protocolo selecionado]
  Homem 3 dobras: Peitoral, Abdominal, Coxa
  Mulher 3 dobras: Tricipital, Suprailiaca, Coxa

[Para cada dobra - Card individual]
  [Titulo da dobra: "Peitoral"]
  [Area clicavel com ilustracao do ponto anatomico]
    -> Expande Dialog com imagem maior + instrucao detalhada:
       "Dobra diagonal, entre axila e mamilo..."

  Medida 1 (mm) [Input]  Medida 2 (mm) [Input]  Medida 3 (mm) [Input]
  [Texto pequeno]: Mediana: 12.0 mm

[Separator]

[Card Resultado - aparece quando todas as dobras preenchidas]
  Soma das dobras: 85.0 mm
  Densidade corporal: 1.0534
  % Gordura: 19.8%
  [Badge]: "Bom" (classificacao por faixa etaria)

  Massa gorda: 14.2 kg
  Massa magra: 57.5 kg
  Peso ideal (15% gordura): 67.6 kg

  [Card TMB]
    Harris-Benedict: 1,650 kcal/dia
    Katch-McArdle: 1,580 kcal/dia
    GET estimado (moderado): 2,560 kcal/dia
```

- Ilustracoes SVG simplificadas mostrando o ponto de cada dobra
- Mediana calculada automaticamente
- Todos os calculos de Pollock em tempo real
- Componentes: Switch, Card, Dialog, Input, Badge, Tooltip

---

### Etapa 10 - Etapa 5 do Wizard: Avaliacao Postural
**Objetivo**: Checklist visual de desvios posturais

```
[Tabs: Anterior | Posterior | Lateral D | Lateral E]

[Cada tab contem:]

  [Silhueta SVG do corpo na vista selecionada]
    - Segmentos clicaveis (cabeca, ombros, coluna, pelve, joelhos, pes)
    - Segmento selecionado fica destacado

  [Abaixo da silhueta - lista de segmentos]
    [Collapsible: Cabeca]
      [Checkbox] Inclinacao lateral -> [Select: Leve/Moderado/Acentuado]
      [Checkbox] Rotacao           -> [Select: Leve/Moderado/Acentuado]
      [Checkbox] Projecao anterior -> [Select: Leve/Moderado/Acentuado]
      Observacoes: [Textarea]

    [Collapsible: Ombros]
      [Checkbox] Elevacao  -> [Select]
      [Checkbox] Protrusao -> [Select]
      [Checkbox] Assimetria -> [Select]
      Observacoes: [Textarea]

    ... (demais segmentos)

  [Button outline: Anexar foto desta vista]

[Resumo no final]
  [Card: Desvios encontrados]
    Lista de todos os desvios marcados com severidade
    Ex: "Ombro D elevado (moderado), Hiperlordose lombar (leve)"
```

- Silhueta SVG com areas clicaveis por segmento
- Cada segmento e um Collapsible com checkbox para desvios
- Severidade aparece condicionalmente (so se checkbox marcado)
- Resumo atualiza em tempo real
- Componentes: Tabs, Collapsible, Checkbox, Select, Textarea, Button, Card

---

### Etapa 11 - Etapa 6 do Wizard: Testes Fisicos
**Objetivo**: Registro dos testes com classificacao automatica

```
[Collapsible aberto: Flexibilidade - Banco de Wells]
  [Info]: "Aluno sentado, pernas estendidas. Empurrar a regua o mais longe possivel."
  Tentativa 1 (cm)  [Input]
  Tentativa 2 (cm)  [Input]
  Tentativa 3 (cm)  [Input]
  [Texto]: Melhor resultado: 28.0 cm
  [Badge]: "Bom"

[Collapsible: Cardiorrespiratorio]
  FC de repouso (bpm)    [Input]
  [Badge]: "Excelente" (classificacao)

  [Card: Zonas de Treinamento (Karvonen)]
    FC maxima estimada: 190 bpm
    Zona 1 (50-60%): 130-142 bpm  [Badge: Aquecimento]
    Zona 2 (60-70%): 142-154 bpm  [Badge: Queima gordura]
    Zona 3 (70-80%): 154-166 bpm  [Badge: Aerobico]
    Zona 4 (80-90%): 166-178 bpm  [Badge: Anaerobico]
    Zona 5 (90-100%): 178-190 bpm [Badge: VO2max]

  [Collapsible: Teste de Cooper (opcional)]
    Distancia em 12 min (m) [Input]
    VO2max estimado: 42.3 ml/kg/min
    [Badge]: "Bom"

[Collapsible: Forca e Resistencia]
  Flexoes em 1 min   [Input]  [Badge: classificacao]
  Abdominais em 1 min [Input]  [Badge: classificacao]

  [Collapsible: Estimativa de 1RM]
    [Button: + Adicionar exercicio]

    [Card exercicio]
      Exercicio  [Input: "Supino reto"]
      Carga (kg) [Input]
      Repeticoes [Input]
      [Texto]: 1RM estimado: 80.5 kg (Brzycki)
    [Button: + Adicionar outro]
```

- Classificacoes aparecem automaticamente ao preencher
- Zonas de Karvonen calculam em tempo real com base na idade do aluno
- 1RM: pode adicionar quantos exercicios quiser (lista dinamica)
- Componentes: Collapsible, Input, Badge, Card, Button

---

### Etapa 12 - Tela de Resultado da Avaliacao
**Objetivo**: Dashboard visual com todos os dados calculados

**Tela: Resultado** (`src/app/(app)/avaliacao/[id]/page.tsx`)

```
[Header com dados do aluno]
  [Avatar] [Nome, Idade, Sexo]
  [Data da avaliacao]
  [Button: Exportar PDF]

[ScrollArea vertical - cards empilhados]

[Card: Composicao Corporal]
  [PieChart - Recharts]
    Massa gorda: 14.2 kg (19.8%)
    Massa magra: 57.5 kg (80.2%)

  [Grid 2x2 de mini-cards]
    Peso: 71.7 kg
    Altura: 172 cm
    IMC: 24.3 [Badge: Normal]
    % Gordura: 19.8% [Badge: Bom]

  [Barra visual IMC com marcador]
  [Barra visual %G com marcador por faixa etaria]

[Card: Circunferencias]
  [Grid responsivo com todas as medidas]
  RCQ: 0.85 [Badge: Moderado]
  Cintura: 82 cm [Badge: Alerta]

[Card: Avaliacao Postural]
  [Lista de desvios encontrados por vista]
    Anterior: Ombro D elevado (moderado)
    Lateral D: Hiperlordose lombar (leve)
    ...
  [Se nenhum desvio]: "Nenhum desvio significativo identificado"

[Card: Testes Fisicos]
  [RadarChart - Recharts]
    Eixos: Flexibilidade, Cardio, Forca superior, Forca abdominal
    Valores normalizados de 0-100 baseado na classificacao

  [Grid de resultados]
    Wells: 28 cm [Badge: Bom]
    FC repouso: 62 bpm [Badge: Excelente]
    Flexoes: 30 [Badge: Bom]
    Abdominais: 35 [Badge: Media]

[Card: Zonas de Treinamento]
  [5 barras coloridas com faixas de FC]

[Card: Metabolismo]
  TMB: 1,650 kcal
  GET (moderado): 2,560 kcal

[Card: Recomendacoes]
  - Texto automatico baseado nos resultados
  - Ex: "% de gordura na faixa 'Bom'. Manter com treino de forca 3x/semana."
  - Ex: "Flexibilidade abdominal abaixo da media. Incluir alongamento diario."

[Footer fixo]
  [Button full-width: Finalizar Avaliacao]
  [Button outline: Editar]
```

Componentes: Card, Badge, Avatar, PieChart, RadarChart, ScrollArea, Button

---

### Etapa 13 - Tela de Evolucao
**Objetivo**: Graficos de progresso ao longo do tempo

**Tela: Evolucao** (`src/app/(app)/alunos/[id]/evolucao/page.tsx`)

```
[Header: Nome do aluno - Evolucao]

[Select periodo: Ultimos 3 meses / 6 meses / 1 ano / Tudo]

[Card: Composicao Corporal]
  [LineChart - 2 linhas]
    - Peso (kg) ao longo do tempo
    - % Gordura ao longo do tempo
  [Legenda interativa: clicar para mostrar/esconder]

[Card: Circunferencias]
  [Select: escolher quais circunferencias mostrar]
  [LineChart com circunferencias selecionadas]

[Card: Testes Fisicos]
  [RadarChart comparando primeira vs ultima avaliacao]

[Card: Comparacao Detalhada]
  [Table com colunas: Metrica | Aval 1 (data) | Aval 2 (data) | Diferenca]
    Peso        71.7 kg    68.5 kg    -3.2 kg  [Badge verde: ↓]
    % Gordura   19.8%      17.2%      -2.6%    [Badge verde: ↓]
    Massa magra 57.5 kg    56.8 kg    -0.7 kg  [Badge amarelo: ↓]
    Cintura     82 cm      79 cm      -3 cm    [Badge verde: ↓]
    ...

[Card: Fotos Antes/Depois]
  [Grid lado a lado]
  [Select: escolher datas para comparar]
```

Componentes: LineChart, RadarChart, Table, Select, Card, Badge

---

### Etapa 14 - Perfil da Treinadora
**Objetivo**: Configuracoes basicas

```
[Avatar grande editavel]
[Form]
  Nome       [Input]
  Email      [Input disabled]
  CREF       [Input]
  Telefone   [Input]
[Button: Salvar]
[Separator]
[Button destructive: Sair]
```

---

## Ordem de Execucao

| # | Etapa | Depende de | Descricao |
|---|-------|-----------|-----------|
| 1 | Setup | - | Next.js + shadcn + tipos + mocks + formulas |
| 2 | Shell | 1 | Layout, navegacao, bottom bar |
| 3 | Login | 2 | Tela de login (mock, vai direto) |
| 4 | Dashboard | 2 | Tela inicial da treinadora |
| 5 | Alunos | 2 | Lista + cadastro + ficha do aluno |
| 6 | Wizard | 2,5 | Container do fluxo de avaliacao (stepper) |
| 7 | Anamnese | 6 | Etapa 1: questionario de saude |
| 8 | Medidas | 6,1* | Etapa 2: peso, altura, IMC |
| 9 | Perimetria | 6,1* | Etapa 3: circunferencias |
| 10 | Dobras | 6,1* | Etapa 4: Pollock + calculos |
| 11 | Postural | 6 | Etapa 5: avaliacao postural |
| 12 | Testes | 6,1* | Etapa 6: testes fisicos |
| 13 | Resultado | 7-12 | Dashboard de resultado |
| 14 | Evolucao | 5,13 | Graficos de progresso |
| 15 | Perfil | 2 | Tela de perfil da treinadora |

*1 = depende das formulas criadas na etapa 1

---

## Entregavel

Ao final, a treinadora podera:
1. Abrir o app no celular
2. Ver dashboard com alunos mock
3. Navegar pela lista de alunos
4. Iniciar uma avaliacao completa passo-a-passo
5. Preencher todos os formularios com calculos em tempo real
6. Ver resultado completo com graficos
7. Comparar evolucao entre avaliacoes (dados mock)
8. Tudo mobile-first, tocavel, rapido

**Sem backend** - tudo funciona com dados em memoria/mock.
Backend (Supabase) sera conectado apos validacao da UX.
