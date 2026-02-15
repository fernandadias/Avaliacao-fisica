# Avaliacao Fisica - Plano V1

## Visao Geral

App web (PWA) para treinadoras realizarem avaliacoes fisicas de atletas/alunos.
O app guia a treinadora durante todo o processo de avaliacao e gera um perfil
completo do aluno, permitindo acompanhar a evolucao ao longo do tempo.

**Publico**: Treinadoras/Personal trainers que atendem alunos presencialmente.

**Uso principal**: Durante a sessao de avaliacao no ginasio/academia, no celular ou tablet.

---

## Stack Tecnico

| Camada | Tecnologia | Justificativa |
|--------|-----------|---------------|
| Framework | Next.js 15 (App Router) + TypeScript | SSR, PWA nativo, React Server Components |
| UI | Tailwind CSS + shadcn/ui | Mobile-first, componentes acessiveis |
| Formularios | react-hook-form + zod | Validacao robusta dos dados de avaliacao |
| Graficos | Recharts | SVG responsivo, nativo React |
| Backend/DB | Supabase (PostgreSQL) | Auth, Realtime, Storage, Row Level Security |
| Offline | Serwist + IndexedDB (idb) | Funcionar no ginasio sem internet |
| PDF | @react-pdf/renderer | Exportar relatorio da avaliacao |
| Datas | date-fns | Manipulacao de datas leve |

---

## Modulos da V1

### 1. Autenticacao e Perfil da Treinadora

- Login/cadastro por email (Supabase Auth)
- Perfil basico: nome, CREF, foto, contato
- Nenhum sistema de pagamento na V1

### 2. Gestao de Alunos

- Cadastro de aluno: nome, data nascimento, sexo, telefone, email, foto
- Lista de alunos com busca
- Ficha do aluno com historico de avaliacoes

### 3. Anamnese (Questionario de Saude)

Formulario guiado passo-a-passo com as seguintes secoes:

**a) PAR-Q (7 perguntas padrao)**
- Sim/Nao para cada pergunta
- Alerta automatico se alguma resposta for "Sim" (encaminhar ao medico)

**b) Historico Medico**
- Condicoes atuais (checkbox: hipertensao, diabetes, cardiopatia, etc.)
- Cirurgias anteriores (texto livre)
- Medicamentos em uso
- Lesoes/problemas ortopedicos
- Historico familiar (doenca cardiaca, diabetes, AVC)

**c) Estilo de Vida**
- Nivel de atividade fisica atual (sedentario/leve/moderado/ativo)
- Experiencia com treino (iniciante/intermediario/avancado)
- Horas de sono por noite
- Nivel de estresse (1-5)
- Tabagismo/alcool
- Consumo diario de agua

**d) Objetivos**
- Objetivo principal (emagrecimento, hipertrofia, saude, performance, reabilitacao)
- Objetivos secundarios
- Disponibilidade semanal para treino

**e) Para mulheres**
- Gravidez atual/historico
- Regularidade menstrual
- Uso de anticoncepcionais

### 4. Antropometria e Composicao Corporal

**a) Medidas Basicas**
- Peso (kg)
- Estatura (cm)
- Calculo automatico: IMC + classificacao OMS

**b) Circunferencias (Perimetria)**
Campos para medir (todos em cm):
- Pescoco
- Ombro
- Torax (relaxado e inspirado)
- Cintura
- Abdomen
- Quadril
- Braco D/E (relaxado e contraido)
- Antebraco D/E
- Coxa proximal D/E
- Coxa medial D/E
- Panturrilha D/E

Calculos automaticos:
- Relacao Cintura-Quadril (RCQ) + classificacao de risco
- Circunferencia da cintura + classificacao de risco

**c) Dobras Cutaneas - Protocolo de Pollock**

O app oferece duas opcoes:
- **Protocolo 3 dobras** (rapido)
- **Protocolo 7 dobras** (completo)

**Homens 3 dobras**: Peitoral, Abdominal, Coxa
**Mulheres 3 dobras**: Tricipital, Suprailíaca, Coxa

**7 dobras (ambos)**: Peitoral, Axilar Media, Tricipital, Subescapular, Abdominal, Suprailíaca, Coxa

Para cada dobra:
- Campo para 3 medicoes (usa a mediana)
- Ilustracao/foto mostrando o ponto anatomico exato
- Instrucao de texto de como medir

Calculos automaticos (Jackson & Pollock):
- Densidade corporal
- % de gordura (equacao de Siri)
- Massa gorda (kg)
- Massa magra (kg)
- Peso ideal (baseado em % de gordura desejado)
- Classificacao por faixa etaria (tabela Pollock & Wilmore)

**Formulas implementadas:**

```
// Homens 3 dobras
DC = 1.10938 - 0.0008267*S + 0.0000016*S² - 0.0002574*idade

// Mulheres 3 dobras
DC = 1.0994921 - 0.0009929*S + 0.0000023*S² - 0.0001392*idade

// Homens 7 dobras
DC = 1.112 - 0.00043499*S + 0.00000055*S² - 0.00028826*idade

// Mulheres 7 dobras
DC = 1.0970 - 0.00046971*S + 0.00000056*S² - 0.00012828*idade

// Gordura corporal (Siri)
%G = (4.95 / DC - 4.50) * 100
```

**d) Taxa Metabolica Basal**
- Harris-Benedict (revisada)
- Katch-McArdle (quando massa magra disponivel)

### 5. Avaliacao Postural

Interface visual com silhueta do corpo humano em 4 vistas:

- **Anterior** (frente)
- **Posterior** (costas)
- **Lateral Direita**
- **Lateral Esquerda**

Para cada vista, a treinadora avalia segmentos corporais:

| Segmento | Desvios possiveis |
|----------|-------------------|
| Cabeca | Inclinacao lateral, rotacao, projecao anterior |
| Ombros | Elevacao, protrusao, assimetria |
| Escapulas | Alada, elevada, abduzida |
| Coluna cervical | Retificacao, hiperlordose |
| Coluna toracica | Hipercifose, retificacao |
| Coluna lombar | Hiperlordose, retificacao |
| Coluna (geral) | Escoliose (C ou S) |
| Pelve | Anteversao, retroversao, inclinacao lateral |
| Joelhos | Valgo, varo, recurvatum, flexo |
| Tornozelos/Pes | Pronacao, supinacao, pe plano, pe cavo, halux valgo |

Para cada desvio:
- Severidade: Normal / Leve / Moderado / Acentuado
- Campo para observacoes

Opcao de anexar fotos do aluno (com consentimento).

### 6. Testes Fisicos

**a) Teste de Flexibilidade - Banco de Wells**
- 3 tentativas, registra a melhor (cm)
- Classificacao automatica por sexo e faixa etaria

**b) Teste Cardiorrespiratorio**
- Frequencia cardiaca de repouso (bpm) + classificacao
- FC maxima estimada (220-idade ou Tanaka)
- Zonas de treinamento (Karvonen): 5 zonas
- Opcao: Teste de Cooper (distancia em 12 min) -> VO2max estimado

**c) Testes de Forca/Resistencia**
- Flexoes de braco (repeticoes em 1 min) + classificacao
- Abdominais (repeticoes em 1 min) + classificacao
- Estimativa de 1RM: exercicio + carga + repeticoes -> calculo (Brzycki/Epley)

### 7. Resultado e Relatorio

**Tela de Resumo da Avaliacao:**
Dashboard com todas as metricas calculadas:

- Dados pessoais + foto
- Composicao corporal (grafico de pizza: massa gorda vs magra)
- IMC com classificacao visual (barra colorida)
- RCQ com classificacao de risco
- % gordura com classificacao por faixa etaria
- Resultados dos testes fisicos com classificacao
- Mapa postural com desvios identificados
- Recomendacoes automaticas baseadas nos resultados

**Exportar PDF:**
- Relatorio completo formatado profissionalmente
- Com logo da treinadora
- Todos os dados, graficos e classificacoes
- Espaco para assinatura

### 8. Acompanhamento e Evolucao

**Comparacao entre avaliacoes:**
- Grafico de linha: evolucao do peso, % gordura, massa magra ao longo do tempo
- Grafico de barras: circunferencias (antes vs depois)
- Tabela comparativa: todas as metricas lado a lado
- Fotos lado a lado (antes/depois)
- Indicadores de melhora/piora com setas e cores

**Dashboard do aluno:**
- Resumo da ultima avaliacao
- Principais metricas com tendencia
- Proxima avaliacao sugerida

---

## Fluxo de Uso Principal

```
1. Treinadora abre o app no celular
2. Seleciona aluno (ou cadastra novo)
3. Inicia "Nova Avaliacao"
4. App guia passo-a-passo:

   [Anamnese] -> [Medidas Basicas] -> [Circunferencias] -> [Dobras Cutaneas]
        -> [Avaliacao Postural] -> [Testes Fisicos] -> [Resultado]

5. Em cada etapa:
   - Instrucoes claras de como realizar a medicao
   - Campos otimizados para input rapido no celular
   - Calculos automaticos em tempo real
   - Botao "Proximo" / "Anterior" para navegar

6. No final: Resumo completo + opcao de gerar PDF
7. Dados salvos automaticamente (offline-first)
```

---

## Estrutura do Projeto

```
avaliacao-fisica/
  app/
    manifest.ts                 # Configuracao PWA
    layout.tsx                  # Layout raiz
    page.tsx                    # Landing / Login
    (auth)/
      login/page.tsx
      register/page.tsx
    (app)/
      layout.tsx                # Layout com navegacao
      dashboard/page.tsx        # Dashboard principal
      alunos/
        page.tsx                # Lista de alunos
        [id]/page.tsx           # Ficha do aluno
        novo/page.tsx           # Cadastro de aluno
      avaliacao/
        nova/[alunoId]/
          page.tsx              # Fluxo guiado de avaliacao
          anamnese/page.tsx
          antropometria/page.tsx
          postural/page.tsx
          testes/page.tsx
          resultado/page.tsx
        [id]/page.tsx           # Visualizar avaliacao
      evolucao/
        [alunoId]/page.tsx      # Graficos de evolucao
  components/
    ui/                         # shadcn/ui components
    forms/
      anamnese-form.tsx
      medidas-form.tsx
      dobras-form.tsx
      postural-form.tsx
      testes-form.tsx
    charts/
      body-composition-chart.tsx
      circumference-chart.tsx
      progress-line-chart.tsx
      fitness-radar-chart.tsx
    assessment/
      step-wizard.tsx           # Wizard de navegacao entre etapas
      body-silhouette.tsx       # Silhueta para avaliacao postural
      skinfold-guide.tsx        # Guia visual dos pontos de dobra
    pdf/
      assessment-report.tsx     # Template do relatorio PDF
  lib/
    formulas/
      pollock.ts                # Formulas Jackson & Pollock
      body-composition.ts       # IMC, RCQ, massa gorda/magra
      metabolic.ts              # TMB Harris-Benedict, Katch-McArdle
      cardio.ts                 # FCmax, Karvonen, Cooper VO2max
      strength.ts               # 1RM Brzycki, Epley
      classification.ts         # Tabelas de classificacao
    supabase/
      client.ts                 # Cliente Supabase
      types.ts                  # Tipos gerados do DB
    offline/
      sync.ts                   # Sincronizacao offline
      store.ts                  # IndexedDB wrapper
    utils/
      date.ts
      format.ts
  public/
    images/
      skinfold-sites/           # Ilustracoes dos pontos de dobra
      posture-guides/           # Guias visuais posturais
```

---

## Modelo de Dados (Supabase/PostgreSQL)

```sql
-- Treinadora
create table trainers (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users(id),
  name text not null,
  cref text,
  phone text,
  photo_url text,
  created_at timestamptz default now()
);

-- Aluno
create table students (
  id uuid primary key default gen_random_uuid(),
  trainer_id uuid references trainers(id),
  name text not null,
  birth_date date not null,
  sex text check (sex in ('M', 'F')) not null,
  phone text,
  email text,
  photo_url text,
  active boolean default true,
  created_at timestamptz default now()
);

-- Avaliacao
create table assessments (
  id uuid primary key default gen_random_uuid(),
  student_id uuid references students(id),
  trainer_id uuid references trainers(id),
  date date not null default current_date,
  status text check (status in ('in_progress', 'completed')) default 'in_progress',
  notes text,
  created_at timestamptz default now()
);

-- Anamnese
create table anamnesis (
  id uuid primary key default gen_random_uuid(),
  assessment_id uuid references assessments(id) unique,
  parq_responses jsonb not null default '{}',
  medical_history jsonb default '{}',
  lifestyle jsonb default '{}',
  objectives jsonb default '{}',
  women_health jsonb default '{}',
  risk_level text check (risk_level in ('low', 'moderate', 'high'))
);

-- Medidas Corporais
create table body_measurements (
  id uuid primary key default gen_random_uuid(),
  assessment_id uuid references assessments(id) unique,
  weight_kg numeric(5,2),
  height_cm numeric(5,1),
  bmi numeric(4,1),
  bmi_class text,
  -- Circunferencias (cm)
  neck_cm numeric(5,1),
  shoulder_cm numeric(5,1),
  chest_relaxed_cm numeric(5,1),
  chest_expanded_cm numeric(5,1),
  waist_cm numeric(5,1),
  abdomen_cm numeric(5,1),
  hip_cm numeric(5,1),
  right_arm_relaxed_cm numeric(5,1),
  right_arm_contracted_cm numeric(5,1),
  left_arm_relaxed_cm numeric(5,1),
  left_arm_contracted_cm numeric(5,1),
  right_forearm_cm numeric(5,1),
  left_forearm_cm numeric(5,1),
  right_thigh_proximal_cm numeric(5,1),
  right_thigh_medial_cm numeric(5,1),
  left_thigh_proximal_cm numeric(5,1),
  left_thigh_medial_cm numeric(5,1),
  right_calf_cm numeric(5,1),
  left_calf_cm numeric(5,1),
  waist_hip_ratio numeric(4,3),
  waist_hip_risk text
);

-- Dobras Cutaneas
create table skinfold_measurements (
  id uuid primary key default gen_random_uuid(),
  assessment_id uuid references assessments(id) unique,
  protocol text check (protocol in ('3_sites', '7_sites')) not null,
  -- Cada dobra: 3 medicoes (mm)
  chest_1 numeric(4,1), chest_2 numeric(4,1), chest_3 numeric(4,1),
  midaxillary_1 numeric(4,1), midaxillary_2 numeric(4,1), midaxillary_3 numeric(4,1),
  triceps_1 numeric(4,1), triceps_2 numeric(4,1), triceps_3 numeric(4,1),
  subscapular_1 numeric(4,1), subscapular_2 numeric(4,1), subscapular_3 numeric(4,1),
  abdomen_1 numeric(4,1), abdomen_2 numeric(4,1), abdomen_3 numeric(4,1),
  suprailiac_1 numeric(4,1), suprailiac_2 numeric(4,1), suprailiac_3 numeric(4,1),
  thigh_1 numeric(4,1), thigh_2 numeric(4,1), thigh_3 numeric(4,1),
  -- Resultados calculados
  sum_skinfolds numeric(5,1),
  body_density numeric(7,5),
  body_fat_pct numeric(4,1),
  fat_mass_kg numeric(5,2),
  lean_mass_kg numeric(5,2),
  fat_class text
);

-- Avaliacao Postural
create table postural_assessments (
  id uuid primary key default gen_random_uuid(),
  assessment_id uuid references assessments(id) unique,
  findings jsonb not null default '[]',
  -- jsonb array: [{view, segment, deviation, severity, notes}]
  photos jsonb default '[]',
  -- jsonb array: [{url, view, timestamp}]
  notes text
);

-- Testes Fisicos
create table fitness_tests (
  id uuid primary key default gen_random_uuid(),
  assessment_id uuid references assessments(id) unique,
  -- Flexibilidade
  wells_bench_cm numeric(4,1),
  wells_class text,
  -- Cardio
  resting_hr_bpm integer,
  resting_hr_class text,
  max_hr_bpm integer,
  cooper_distance_m integer,
  vo2max numeric(4,1),
  -- Forca/Resistencia
  pushups_count integer,
  pushups_class text,
  situps_count integer,
  situps_class text,
  -- 1RM estimativas
  rm_estimates jsonb default '[]'
  -- [{exercise, weight_kg, reps, estimated_1rm}]
);
```

---

## Fases de Implementacao

### Fase 1 - Fundacao (setup do projeto)
- [ ] Criar projeto Next.js com TypeScript + Tailwind
- [ ] Configurar shadcn/ui
- [ ] Configurar Supabase (projeto + schema)
- [ ] Configurar autenticacao
- [ ] Configurar PWA (manifest + service worker)
- [ ] Layout base mobile-first com navegacao

### Fase 2 - Gestao de Alunos
- [ ] CRUD de alunos
- [ ] Lista com busca
- [ ] Ficha individual do aluno

### Fase 3 - Fluxo de Avaliacao (core do app)
- [ ] Wizard/stepper de avaliacao
- [ ] Formulario de anamnese (PAR-Q + historico + estilo de vida)
- [ ] Formulario de antropometria (peso, altura, circunferencias)
- [ ] Formulario de dobras cutaneas (Pollock 3 e 7 dobras)
- [ ] Modulo de avaliacao postural (silhueta + checklist)
- [ ] Formulario de testes fisicos

### Fase 4 - Calculos e Classificacoes
- [ ] Implementar todas as formulas (Pollock, Siri, IMC, RCQ, etc.)
- [ ] Tabelas de classificacao por sexo e idade
- [ ] Calculos em tempo real nos formularios

### Fase 5 - Resultado e Relatorio
- [ ] Tela de resumo da avaliacao com dashboard
- [ ] Graficos de composicao corporal
- [ ] Mapa postural visual
- [ ] Geracao de PDF

### Fase 6 - Evolucao e Acompanhamento
- [ ] Graficos de evolucao temporal
- [ ] Comparacao entre avaliacoes
- [ ] Dashboard do aluno

### Fase 7 - Offline e Polish
- [ ] Modo offline com IndexedDB
- [ ] Sincronizacao automatica
- [ ] Testes
- [ ] Ajustes de UX mobile
