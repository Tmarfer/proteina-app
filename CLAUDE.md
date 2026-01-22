# CLAUDE.md - Guia para Assistentes de IA do ProteínaMax

## Visão Geral do Projeto

**ProteínaMax** é uma aplicação web single-page completa projetada para ajudar usuários a otimizar sua ingestão proteica e planejar refeições balanceadas para hipertrofia muscular. O app calcula necessidades nutricionais baseadas em dados corporais e objetivos, sugerindo combinações inteligentes de alimentos.

**Idioma Principal:** Português (Brasil)

**Usuários-Alvo:** Praticantes de musculação, atletas, entusiastas de fitness e qualquer pessoa que queira otimizar sua nutrição para ganho de massa muscular

**Funcionalidades Principais:**
- Calculadora TMB/TDEE com fórmula Mifflin-St Jeor
- Calculadora de macronutrientes personalizada
- Dashboard visual com gráficos de progresso
- Base de dados com 32 alimentos (proteínas, carbs, gorduras)
- Sistema de favoritos com localStorage
- Comparador de alimentos
- 10 receitas ricas em proteína
- **Planejador inteligente de refeições** com sugestões automáticas
- Histórico completo de planos alimentares
- Compartilhamento via WhatsApp formatado
- Suporte a dark/light theme
- Design responsivo mobile-first
- Referências científicas validadas

## Estrutura do Projeto

```
proteina-app/
├── .git/                 # Repositório Git
│   └── hooks/           # Git hooks padrão
├── index.html           # APLICAÇÃO COMPLETA (HTML + CSS + JavaScript)
└── CLAUDE.md           # Este arquivo
```

### Arquitetura: Aplicação Single-File

**CRÍTICO:** Esta é uma aplicação monolítica single-page. Todo o código está em `index.html`:
- **Linhas 1-700+:** Estrutura HTML e estilos CSS incorporados
- **Linhas 700+:** Corpo HTML (elementos de UI, 7 abas de navegação)
- **Linhas 1123+:** JavaScript incorporado (lógica da aplicação)

**Sem dependências externas:**
- Sem package.json
- Sem processo de build
- Sem npm/yarn
- Sem frameworks (React, Vue, etc.)
- Sem preprocessadores CSS
- Sem bundlers (Webpack, Vite, etc.)

**Deploy:** Simplesmente servir `index.html` via qualquer servidor web ou abrir diretamente no navegador.

## Stack Tecnológica

### Frontend
- **HTML5:** Marcação semântica com labels em português
- **CSS3:**
  - CSS Custom Properties (Variáveis CSS) para temas
  - Layouts Flexbox e Grid
  - Design responsivo mobile-first
  - Gradientes e animações
  - Incorporado em tag `<style>`

- **JavaScript (ES6+):**
  - JavaScript vanilla puro (sem bibliotecas)
  - Features modernas: arrow functions, template literals, destructuring
  - APIs de manipulação DOM
  - **LocalStorage** usado para persistência

### APIs do Navegador Usadas
- `document.querySelector/querySelectorAll` - Seleção DOM
- `addEventListener/onclick` - Tratamento de eventos
- `localStorage.getItem/setItem` - Persistência de dados
- `window.open()` - Compartilhamento WhatsApp
- `window.location.href` - URL atual para compartilhar
- `scrollIntoView()` - Scroll suave para resultados
- `JSON.parse/JSON.stringify` - Serialização de dados

### Sem Backend
- Arquivo HTML estático apenas
- Todos os cálculos executados client-side
- Sem banco de dados
- Sem chamadas de API
- Sem lógica server-side

## Modelo de Dados

### 1. Base de Dados de Alimentos: `foodsDatabase` (linha ~850)

Base expandida com 32 alimentos categorizados. Schema:

```javascript
{
  id: number,              // Identificador único (1-32)
  nome: string,            // Nome do alimento em português
  categoria: string,       // Categoria: 'carne', 'frango', 'peixe', 'suplemento',
                           //            'ovo', 'laticinio', 'vegetal'
  proteina: number,        // Conteúdo proteico em gramas por porção
  carbs: number,           // Carboidratos em gramas
  gordura: number,         // Gorduras em gramas
  calorias: number,        // Calorias totais
  porcao: string,          // Descrição da porção (ex: '100g', '1 scoop')
  custo: number            // Custo em Reais (R$)
}
```

**Exemplo:**
```javascript
{
  id: 5,
  nome: 'Peito de Frango Grelhado',
  categoria: 'frango',
  proteina: 32,
  carbs: 0,
  gordura: 3.6,
  calorias: 165,
  porcao: '100g',
  custo: 1.5
}
```

**Categorias:**
- `carne` - Carnes vermelhas
- `frango` - Carnes de frango
- `peixe` - Peixes e frutos do mar
- `suplemento` - Whey, albumina, caseína, creatina
- `ovo` - Ovos e claras
- `laticinio` - Queijos, iogurtes
- `vegetal` - Proteínas vegetais

### 2. Sistema de Unidades Realistas: `foodUnits` (linha ~1125)

Configuração de unidades e limites máximos por alimento:

```javascript
{
  'Nome do Alimento': {
    unit: string,           // 'scoop', 'unidade', 'colher', 'gramas'
    gramsPerUnit: number,   // Gramas por unidade
    maxUnits: number,       // Quantidade máxima realista
    displayName: string     // Nome para exibição
  }
}
```

**Exemplos:**
```javascript
{
  'Whey Protein': { unit: 'scoop', gramsPerUnit: 30, maxUnits: 2, displayName: 'scoop' },
  'Ovo Inteiro Cozido': { unit: 'unidade', gramsPerUnit: 50, maxUnits: 3, displayName: 'unidade' },
  'Queijo Cottage': { unit: 'colher', gramsPerUnit: 25, maxUnits: 2, displayName: 'colher de sopa' },
  'Atum Enlatado': { unit: 'gramas', gramsPerUnit: 1, maxUnits: 100, displayName: 'g' }
}
```

**Limites Realistas Aplicados:**
- Whey/Albumina: máx 2 scoops (60g)
- Ovos inteiros: máx 3 unidades
- Claras: máx 6 unidades
- Cottage/Pasta amendoim: máx 2 colheres de sopa
- Atum: máx 100g

### 3. Base de Carboidratos: `carbsDatabase` (linha ~1150)

12 fontes de carboidratos saudáveis:

```javascript
{
  id: number,
  nome: string,           // Nome em português
  carbs: number,          // Carboidratos por 100g
  proteina: number,       // Proteína por 100g
  gordura: number,        // Gordura por 100g
  calorias: number,       // Calorias por 100g
  porcao: string          // Descrição da porção
}
```

**Alimentos incluídos:**
- Arroz Branco/Integral Cozido
- Batata Doce/Inglesa Cozida
- Aveia em Flocos
- Macarrão Integral Cozido
- Pão Integral
- Tapioca Hidratada
- Banana
- Batata Baroa Cozida
- Granola
- Quinoa Cozida

### 4. Base de Gorduras Saudáveis: `fatsDatabase` (linha ~1200)

10 fontes de gorduras saudáveis:

```javascript
{
  id: number,
  nome: string,
  gordura: number,        // Gordura por 100g/100ml
  carbs: number,
  proteina: number,
  calorias: number,
  porcao: string
}
```

**Alimentos incluídos:**
- Azeite Extra Virgem
- Abacate
- Castanha do Pará
- Amêndoas
- Manteiga de Amendoim Natural
- Sementes de Chia
- Sementes de Linhaça
- Salmão (gorduras boas)
- Atum (ômega-3)
- Gema de Ovo

### 5. Sugestões Inteligentes por Refeição: `mealSuggestions` (linha ~1250)

Mapeamento de sugestões contextuais de carbs e gorduras por tipo de refeição:

```javascript
{
  cafe: {
    carbs: ['Aveia em Flocos', 'Pão Integral', 'Tapioca Hidratada', 'Banana', 'Granola'],
    fats: ['Manteiga de Amendoim Natural', 'Abacate', 'Castanha do Pará', 'Amêndoas', 'Sementes de Chia']
  },
  almoco: {
    carbs: ['Arroz Integral Cozido', 'Batata Doce Cozida', 'Batata Inglesa Cozida', 'Macarrão Integral Cozido'],
    fats: ['Azeite Extra Virgem', 'Abacate']
  },
  // ... outras refeições
}
```

### 6. Configuração de Refeições: `meals` (usado em várias funções)

Define 6 refeições diárias com distribuição de macros:

```javascript
[
  { key: 'cafe', label: '☕ Café da Manhã', weight: 0.20 },       // 20% proteína
  { key: 'lancheManha', label: '🍎 Lanche da Manhã', weight: 0.10 },  // 10%
  { key: 'almoco', label: '🍽️ Almoço', weight: 0.30 },           // 30%
  { key: 'lancheTarde', label: '🥪 Lanche da Tarde', weight: 0.10 },  // 10%
  { key: 'jantar', label: '🍲 Jantar', weight: 0.25 },            // 25%
  { key: 'ceia', label: '🌙 Ceia', weight: 0.05 }                 // 5%
]
```

**Total weight deve somar 1.0 (100%)**

### 7. Gerenciamento de Estado

**Variáveis Globais:**
- `foodsDatabase` - Array com 32 alimentos
- `favorites` - Array de IDs favoritos (localStorage)
- `history` - Array de planos salvos (localStorage, máx 30)
- `userMacros` - Objeto com metas calculadas do usuário
- `currentTheme` - Tema atual ('dark' ou 'light')
- `window.currentPlan` - Plano atual gerado

**Persistência:**
- `favorites` e `history` são salvos em localStorage
- Dados persistem entre sessões
- Limite de 30 planos no histórico

### 8. Estrutura de Plano Salvo (Formato Completo)

```javascript
{
  date: string,              // ISO timestamp
  protein: string,           // Total proteínas (g)
  carbs: string,             // Total carboidratos (g)
  fat: string,               // Total gorduras (g)
  calories: string,          // Total calorias
  cost: string,              // Custo total/dia (R$)
  meals: [                   // Array de refeições detalhadas
    {
      key: string,           // Identificador da refeição
      label: string,         // Label com emoji
      proteins: [            // Array de proteínas selecionadas
        {
          id: number,
          nome: string,
          quantidade: string,     // Formatado: "2 scoops", "3 unidades"
          grams: number,          // Gramas reais
          macros: {
            proteina: string,     // "25.0"
            carbs: string,        // "3.0"
            gordura: string       // "1.5"
          }
        }
      ],
      carb: {                // Carboidrato sugerido (pode ser null)
        nome: string,
        quantidade: string,  // "100g"
        macros: string,      // "23.0g carb"
        calorias: string     // "112 kcal"
      },
      fat: {                 // Gordura sugerida (pode ser null)
        nome: string,
        quantidade: string,
        macros: string,
        calorias: string
      },
      totals: {              // Totais da refeição
        proteina: string,
        carbs: string,
        gordura: string,
        calorias: string
      }
    }
  ]
}
```

## Funcionalidades Principais

### 1. Calculadora de TMB/TDEE (Aba Calculadora)

**Função:** `calculateMacros()` (linha ~1400+)

- Calcula Taxa Metabólica Basal usando **Fórmula Mifflin-St Jeor**
- Multiplica por fator de atividade (Harris-Benedict)
- Calcula macronutrientes baseado em objetivo
- Armazena em `userMacros` global

**Fórmula TMB (Mifflin-St Jeor):**
- Homens: (10 × peso) + (6.25 × altura) - (5 × idade) + 5
- Mulheres: (10 × peso) + (6.25 × altura) - (5 × idade) - 161

**Fatores de Atividade:**
- Sedentário: 1.2
- Levemente ativo: 1.375
- Moderadamente ativo: 1.55
- Muito ativo: 1.725
- Extremamente ativo: 1.9

**Distribuição de Macros (Hipertrofia):**
- Proteínas: 2.0g/kg de peso corporal
- Gorduras: 25% das calorias totais
- Carboidratos: restante das calorias

**Referências Científicas Exibidas:**
- Fórmula Mifflin-St Jeor (PubMed)
- Fatores de atividade Harris-Benedict (PNAS)
- Recomendações ISSN proteína (JISSN)
- Distribuição macronutrientes (JISSN)
- Gorduras essenciais (Nutrients)

### 2. Dashboard Visual (Aba Dashboard)

**Função:** `updateDashboard()` (linha ~1600+)

- Gráfico de pizza CSS puro para macros
- Cards com totais de alimentos, favoritos, planos salvos
- Visual moderno com gradientes
- Atualizado automaticamente após calcular macros

### 3. Tabela de Alimentos (Aba Fontes Proteicas)

**Função:** `renderFoodsTable()` (linha ~1300+)

- Renderiza 32 alimentos com todos os macros
- Sistema de favoritos (estrela clicável)
- Ordenação por proteína, carbs, gordura, calorias, custo
- Filtros por categoria
- Busca em tempo real
- Badges coloridos por categoria
- Persistência de favoritos em localStorage

**Features:**
- Click na estrela adiciona/remove favoritos
- Filtros: todas, carne, frango, peixe, suplemento, ovo, laticinio, vegetal
- Ordenação ascendente/descendente
- Busca case-insensitive

### 4. Favoritos (Aba Favoritos)

**Função:** `renderFavorites()` (linha ~1450+)

- Lista alimentos marcados como favoritos
- Permite remover favoritos
- Estado vazio amigável
- Sincronizado com tabela principal

### 5. Comparador de Alimentos (Aba Comparador)

**Funções:** `populateCompareSelects()`, `compareFoods()` (linha ~1550+)

- Seleciona 2 alimentos para comparar lado a lado
- Exibe todos os macros
- Mostra custo por porção
- Cálculo de custo/g de proteína
- Visual com cores diferenciadas

### 6. Receitas (Aba Receitas)

**Função:** `renderRecipes()` (linha ~1700+)

- 10 receitas ricas em proteína
- Ingredientes e modo de preparo
- Valores nutricionais totais
- Cards expansíveis

**Receitas incluídas:**
1. Omelete Proteico
2. Frango Grelhado com Legumes
3. Smoothie Proteico
4. Salada de Atum
5. Panqueca de Banana e Aveia
6. Peito de Frango com Batata Doce
7. Wrap de Frango
8. Vitamina de Whey com Frutas
9. Ovo Mexido com Cottage
10. Bowl de Quinoa Proteico

### 7. Planejador Inteligente de Refeições (Aba Planejador) ⭐

**Função:** `initMealPlanner()` (linha ~1795)
**Função:** `calculateSmartMealPlan()` (linha ~2043)

**Fluxo Completo:**

**A. Inicialização:**
- Cria 6 cards de refeição (café, lanche manhã, almoço, lanche tarde, jantar, ceia)
- Cada card tem **2 selects de proteína** (principal + secundária opcional)
- Popula com alimentos que têm proteína >= 8g

**B. Seleção do Usuário:**
- Usuário escolhe 1 ou 2 proteínas por refeição
- Pode deixar refeições vazias
- Feedback visual ao selecionar

**C. Cálculo Inteligente (ao clicar "Gerar Plano Inteligente"):**

1. **Valida se usuário calculou macros** na aba Calculadora
2. **Coleta proteínas selecionadas** de todos os selects
3. **Para cada refeição:**
   - Divide meta de proteína igualmente entre proteínas selecionadas
   - Calcula quantidade necessária de cada proteína
   - **Aplica limites realistas** via `calculateRealGrams()`
   - Calcula macros reais de cada proteína
   - Calcula déficit de carbs e gorduras restante
   - **Seleciona carb e gordura sugeridos** baseado no tipo de refeição
   - Calcula quantidades de carb e gordura para completar macros
   - Armazena detalhes completos da refeição

4. **Renderiza plano:**
   - Card para cada refeição
   - Todas as proteínas com quantidades formatadas
   - Carboidrato sugerido
   - Gordura sugerida
   - Totais da refeição
   - Resumo do dia com totais e percentuais

5. **Armazena plano em `window.currentPlan`** com estrutura completa

**D. Funções Auxiliares:**

**`formatQuantity(foodName, grams)`:**
- Formata quantidade com unidade correta
- Retorna: "2 scoops (60g)", "3 unidades", "1.5 colheres de sopa (37g)", "150g"

**`calculateRealGrams(foodName, targetGrams)`:**
- Aplica limite máximo do alimento
- Calcula gramas reais respeitando limites
- Retorna quantidade ajustada

**Lógica de Sugestões:**
- Café da manhã → Aveia, pão integral, tapioca + amendoim, abacate, castanhas
- Almoço/Jantar → Arroz, batata doce, macarrão + azeite, abacate
- Lanches → Frutas, granola + oleaginosas
- Ceia → Aveia, banana + gorduras saudáveis

### 8. Histórico de Planos (Aba Histórico) ⭐

**Função:** `savePlanToHistory()` (linha ~2333)
**Função:** `renderHistory()` (linha ~2346)
**Função:** `viewHistoryDetails(index)` (linha ~2400)
**Função:** `shareHistoryToWhatsApp(index)` (linha ~2591)

**A. Salvamento:**
- Salva plano completo em `history` array
- Persiste em localStorage
- Mantém últimos 30 planos
- Estrutura detalhada com todos os alimentos

**B. Visualização da Lista:**
- Lista todos os planos salvos
- Mostra data/hora, resumo de macros
- Cada item é clicável
- Botão de deletar individual
- Botão de limpar histórico

**C. Visualização Detalhada:**
- Click no item abre detalhes completos
- Mostra resumo nutricional destacado
- Lista todas as refeições
- Exibe todas as proteínas, carbs e gorduras
- Quantidades formatadas e macros individuais
- Totais por refeição
- Botão de fechar
- Scroll suave

**D. Compartilhamento WhatsApp:**
- Botão dentro da visualização de detalhes
- Gera mensagem formatada corretamente
- Usa quebras de linha reais (`\n` não `\\n`)
- Emojis literais (não códigos Unicode)
- Inclui:
  - Título e data
  - Resumo nutricional completo
  - Todas as refeições detalhadas
  - Proteínas com quantidades e macros
  - Carbs e gorduras sugeridos
  - Totais por refeição
  - Link do app

**Formato da Mensagem:**
```
*🎯 MEU PLANO ALIMENTAR*
📅 Data: 22/01/2026

*📊 RESUMO NUTRICIONAL*
🥩 Proteínas: 180g
🍚 Carboidratos: 250g
🥑 Gorduras: 60g
🔥 Calorias: 2200 kcal
💰 Custo/dia: R$ 25.50

*🍽️ REFEIÇÕES DETALHADAS*

☕ Café da Manhã:
  🥩 Ovo Inteiro Cozido - 3 unidades
     15.0g prot | 1.5g carb | 10.5g gord
  🍚 Aveia em Flocos - 50g
     25.5g carb
  📊 Total: 39g prot | 30g carb | 19g gord | 450 kcal

━━━━━━━━━━━━━━━
💪 Gerado pelo ProteínaMax
https://...
```

**Compatibilidade:**
- Detecta automaticamente formato novo (detalhado) ou antigo (só proteínas)
- Renderização e compartilhamento funcionam em ambos

### 9. Tema Claro/Escuro

**Função:** `toggleTheme()` (linha ~1390)

- Alterna entre 'dark' e 'light'
- Atualiza atributo `data-theme` no `<html>`
- CSS variables atualizam automaticamente
- Tema escuro como padrão

**Cores (Dark Theme):**
- Primary: #00ff88 (verde neon)
- Background: #050505
- Surface: #0f0f0f
- Text: #e0e0e0

## Fluxo de Desenvolvimento

### Fazendo Alterações

1. **Editar `index.html` diretamente** - todo código está neste arquivo único
2. **Testar no navegador** - refresh para ver mudanças
3. **Sem build step** - mudanças são imediatas

### Adicionando Novos Alimentos

**Localização:** `foodsDatabase` array (linha ~850)

**Passos:**
1. Adicionar novo objeto ao array `foodsDatabase`
2. Atribuir próximo ID sequencial
3. Preencher todos os campos obrigatórios
4. Escolher categoria apropriada
5. Refresh do navegador para ver mudanças

**Exemplo:**
```javascript
{
  id: 33,
  nome: 'Salmão Grelhado',
  categoria: 'peixe',
  proteina: 25,
  carbs: 0,
  gordura: 12,
  calorias: 206,
  porcao: '100g',
  custo: 8.50
}
```

### Adicionando Limites Realistas

**Localização:** `foodUnits` object (linha ~1125)

```javascript
'Nome Exato do Alimento': {
  unit: 'unidade',        // ou 'scoop', 'colher', 'gramas'
  gramsPerUnit: 50,       // gramas por unidade
  maxUnits: 3,            // quantidade máxima realista
  displayName: 'unidade'  // nome para exibição
}
```

### Adicionando Carboidratos/Gorduras

**Carboidratos:** `carbsDatabase` (linha ~1150)
**Gorduras:** `fatsDatabase` (linha ~1200)

Seguir schema existente com id, nome, macros, calorias, porção.

### Modificando Sugestões de Refeições

**Localização:** `mealSuggestions` (linha ~1250)

Adicionar/remover alimentos nos arrays `carbs` e `fats` de cada refeição.

### Modificando Distribuição de Proteína

**Localização:** Array `meals` em `initMealPlanner()` e `calculateSmartMealPlan()`

**Regra:** Soma de todos os `weight` deve ser 1.0 (100%)

```javascript
{ key: 'cafe', label: '☕ Café da Manhã', weight: 0.25 },  // Aumentado para 25%
{ key: 'almoco', label: '🍽️ Almoço', weight: 0.25 },      // Reduzido para 25%
// ... ajustar outros para somar 1.0
```

### Modificando Multiplicador de Proteína

**Localização:** Função `calculateMacros()` (linha ~1400+)

Atualmente usa **2.0g/kg** para hipertrofia. Para mudar:

```javascript
const proteinGrams = weight * 2.2; // Alterar de 2.0 para 2.2
```

## Convenções de Código

### Convenções JavaScript

**Nomenclatura:**
- **Funções:** camelCase, verbos descritivos
  - `calculateMacros()`, `renderFoodsTable()`, `viewHistoryDetails()`
- **Variáveis:** camelCase, substantivos descritivos
  - `foodsDatabase`, `userMacros`, `currentTheme`
- **Constantes:** camelCase (não UPPER_CASE neste codebase)

**Organização do Código:**
1. Declarações de dados (foodsDatabase, carbsDatabase, etc.)
2. Variáveis de estado
3. Funções de inicialização
4. Event handlers
5. Funções auxiliares
6. Event listener DOMContentLoaded no final

**Estilo:**
- Preferir arrow functions para callbacks
- Template literals para strings com variáveis
- Nomes descritivos que indicam ação
- Semicolons opcionais (estilo inconsistente no código)

### Convenções HTML

**Ordem de Atributos:**
1. id
2. class
3. data-* attributes
4. outros atributos
5. event handlers (onclick, onchange)

**IDs:** kebab-case
- `history-list`, `plan-results`, `meal-planner-container`

**Classes:** kebab-case
- `nav-tab`, `card`, `btn-primary`, `form-select`

### Convenções CSS

**Variáveis:** CSS custom properties para temas
- Sempre definir para ambos light e dark themes
- Cores: `--primary`, `--bg`, `--surface`, `--text`

**Naming de Classes:** Estilo BEM simplificado
- Block: `.card`
- Element: `.card-header`, `.card-title`
- Modifier: `.btn-primary`, `.nav-tab.active`

**Unidades:**
- Font sizes: `px`
- Spacing: `px`
- Borders: `px`
- Border radius: `px`

### Português do Brasil

**Todo texto user-facing DEVE estar em português:**
- Labels de UI
- Textos de botões
- Mensagens de erro/sucesso
- Comentários no código (opcional mas recomendado)
- Console logs (se houver)

**Nomes de variáveis/funções:**
- Codebase usa mix de português e inglês
- Novos nomes podem ser em qualquer idioma, mas seja consistente

## Testes e Deploy

### Testes

**Não há testes automatizados.**

**Checklist de testes manuais:**

**Calculadora:**
- [ ] Inserir dados válidos
- [ ] Verificar cálculo de TMB
- [ ] Verificar cálculo de macros
- [ ] Testar todos os níveis de atividade
- [ ] Validar referências científicas

**Tabela de Alimentos:**
- [ ] Ordenar por cada coluna
- [ ] Filtrar por categoria
- [ ] Buscar alimentos
- [ ] Adicionar/remover favoritos
- [ ] Verificar persistência de favoritos

**Planejador:**
- [ ] Selecionar 1 proteína por refeição
- [ ] Selecionar 2 proteínas por refeição
- [ ] Calcular plano inteligente
- [ ] Verificar quantidades formatadas (scoops, unidades, colheres)
- [ ] Verificar limites realistas aplicados
- [ ] Verificar sugestões de carbs e gorduras
- [ ] Verificar totais e percentuais

**Histórico:**
- [ ] Salvar plano
- [ ] Visualizar lista de planos
- [ ] Clicar para ver detalhes
- [ ] Verificar exibição completa (proteínas, carbs, gorduras)
- [ ] Compartilhar no WhatsApp
- [ ] Verificar formatação da mensagem no WhatsApp
- [ ] Deletar plano individual
- [ ] Limpar histórico completo

**Geral:**
- [ ] Alternar tema claro/escuro
- [ ] Testar em viewport mobile (320px+)
- [ ] Testar em tablet
- [ ] Testar em desktop
- [ ] Verificar persistência em localStorage

### Compatibilidade de Navegadores

**Navegadores Alvo:**
- Chrome/Edge (latest)
- Firefox (latest)
- Safari (latest)
- Mobile browsers (iOS Safari, Chrome Mobile)

**Features que requerem navegador moderno:**
- CSS Grid
- CSS Custom Properties
- ES6+ JavaScript
- Flexbox
- LocalStorage API

**Sem suporte para IE11** (usa CSS e JS modernos)

### Deploy

**Opções de Hospedagem Estática:**
- GitHub Pages
- Netlify
- Vercel
- Qualquer servidor web
- Pode rodar de URL `file://`

**Passos:**
1. Commitar mudanças no repositório
2. Push para serviço de hospedagem ou servidor web
3. Sem build step necessário
4. `index.html` é o entry point

## Convenções Git

### Mensagens de Commit

**Idioma:** Português (Brasil)

**Estilo:** Modo imperativo, descritivo
- ✅ "Adiciona seleção de 2 proteínas por refeição"
- ✅ "Corrige formatação da mensagem WhatsApp"
- ✅ "Implementa planejador inteligente com sugestões automáticas"
- ❌ "Updated stuff"
- ❌ "WIP"
- ❌ "fix bug"

**Padrão:** `[Ação] [mudança específica]`
- Ações comuns: `Adiciona`, `Corrige`, `Implementa`, `Atualiza`, `Remove`, `Refatora`

**Commits complexos:** Usar mensagem de commit multilinha com detalhes

### Nomenclatura de Branches

**Branches de AI Assistant:** `claude/[nome-descritivo]-[session-id]`
- Exemplo: `claude/add-meal-planner-features-abc123`
- Session ID é auto-gerado e deve corresponder para push funcionar

**Convenção:**
- Feature branches: kebab-case descritivo
- Sem números de versão em nomes de branch
- Manter nomes concisos mas claros

### Workflow Git

1. **Sempre verificar branch atual** antes de fazer mudanças
2. **Commitar frequentemente** com mensagens claras
3. **Push para branch designado** usando `git push -u origin <branch-name>`
4. **Nunca force push** para main/master
5. **Usar mensagens descritivas** em português

**Para AI Assistants:**
- Desenvolver em branch claude/* especificado
- Commitar após mudanças lógicas
- Push quando trabalho estiver completo
- Criar PR com resumo em português

## Tarefas Comuns & Exemplos

### Tarefa 1: Adicionar Nova Proteína

```javascript
// Em foodsDatabase array (linha ~850), adicionar:
{
  id: 33,
  nome: 'Filé Mignon',
  categoria: 'carne',
  proteina: 29,
  carbs: 0,
  gordura: 8,
  calorias: 195,
  porcao: '100g',
  custo: 12.00
}
```

### Tarefa 2: Adicionar Limite Realista para Novo Alimento

```javascript
// Em foodUnits (linha ~1125):
'Filé Mignon': {
  unit: 'gramas',
  gramsPerUnit: 1,
  maxUnits: 200,  // máx 200g por refeição
  displayName: 'g'
}
```

### Tarefa 3: Alterar Ordem de Ordenação Padrão

```javascript
// Na função renderFoodsTable(), modificar sort inicial:
const sortedFoods = [...foodsDatabase].sort((a, b) => b.proteina - a.proteina);
// Ordena por proteína descendente como padrão
```

### Tarefa 4: Adicionar Nova Categoria

```javascript
// 1. Adicionar alimento com nova categoria em foodsDatabase
{ id: 33, nome: 'Lentilha', categoria: 'leguminosa', ... }

// 2. Adicionar badge CSS (se necessário)
.badge-leguminosa { background: #8b4513; color: white; }

// 3. Adicionar filtro na UI (modificar HTML)
<button class="filter-btn" onclick="filterCategory('leguminosa')">
  🌱 Leguminosas
</button>
```

### Tarefa 5: Modificar Cores do Tema

```css
/* Lines 9-35 - modificar CSS variables */
:root[data-theme="dark"] {
  --primary: #00ffff;      /* Mudar cor primária para cyan */
  --bg: #000000;           /* Fundo mais escuro */
  --surface: #1a1a1a;      /* Surface mais claro */
}
```

### Tarefa 6: Adicionar Nova Receita

```javascript
// Em recipes array (linha ~1750+):
{
  id: 11,
  titulo: 'Bolinho de Batata Doce com Atum',
  categoria: 'Lanche',
  tempo: '25 min',
  ingredientes: [
    '200g de batata doce cozida e amassada',
    '1 lata de atum',
    '1 ovo',
    'Temperos a gosto'
  ],
  preparo: [
    'Misture todos os ingredientes',
    'Faça bolinhos',
    'Asse no forno a 180°C por 20 minutos'
  ],
  proteinas: 35,
  carboidratos: 30,
  gorduras: 8,
  calorias: 340
}
```

## Notas Importantes para AI Assistants

### Regras Críticas

1. **NUNCA dividir o arquivo** - Esta é intencionalmente uma app monolítica single-file
2. **Testar no navegador** - Mudanças não podem ser validadas sem inspeção visual
3. **Preservar português** - Todo texto user-facing DEVE permanecer em português
4. **Manter estrutura de dados** - Não mudar schemas sem atualizar todos os usos
5. **Formato WhatsApp** - Usar `\n` (quebra real), não `\\n` (escape literal)
6. **Emojis WhatsApp** - Usar emojis literais, não códigos Unicode
7. **Limites realistas** - Sempre aplicar via `calculateRealGrams()`
8. **Histórico completo** - Salvar plano detalhado, não apenas IDs
9. **Mensagens de commit em português** - Seguir estilo existente

### Armadilhas Comuns

❌ **Não fazer:** Criar arquivos .js, .css separados
✅ **Fazer:** Editar código incorporado em index.html

❌ **Não fazer:** Adicionar dependências ou npm packages
✅ **Fazer:** Usar apenas JavaScript vanilla

❌ **Não fazer:** Usar inglês em texto user-facing
✅ **Fazer:** Manter todo UI text em português

❌ **Não fazer:** Usar `\\n` em mensagens WhatsApp
✅ **Fazer:** Usar `\n` (quebra de linha real)

❌ **Não fazer:** Salvar apenas IDs no histórico
✅ **Fazer:** Salvar estrutura detalhada completa

❌ **Não fazer:** Assumir que dados persistem sem localStorage
✅ **Fazer:** Entender o que é session-only vs persistido

❌ **Não fazer:** Adicionar build tools ou transpilação
✅ **Fazer:** Escrever código ES6+ compatível com navegadores

### Melhores Práticas

1. **Antes de editar:** Ler função inteira para entender contexto
2. **Depois de editar:** Verificar side effects em outras funções
3. **Formato de dados:** Sempre fornecer estrutura completa
4. **Testes:** Sempre fornecer passos de testes manuais para mudanças
5. **Documentação:** Atualizar este arquivo ao fazer mudanças arquiteturais
6. **Cálculos:** Double-check matemática para precisão
7. **Design responsivo:** Testar mudanças em viewports mobile
8. **Compatibilidade:** Manter compatibilidade com dados antigos quando possível

### Ao Fazer Mudanças, Sempre Verificar

- [ ] Código é ES6+ JavaScript válido
- [ ] Estrutura HTML está semanticamente correta
- [ ] CSS não quebra layout existente
- [ ] Todo texto está em português (exceto código)
- [ ] Emojis renderizam corretamente em UI e WhatsApp
- [ ] Cálculos matemáticos estão precisos
- [ ] Sem erros no console do navegador
- [ ] Layout mobile ainda funciona
- [ ] Alternância de tema ainda funciona
- [ ] Integridade da estrutura de dados mantida
- [ ] LocalStorage funcionando corretamente
- [ ] Formatação WhatsApp está correta

### Debugging Tips

1. **Abrir console do navegador** - Verificar erros JavaScript
2. **Inspecionar elemento** - Verificar CSS aplicado corretamente
3. **Verificar fluxo de dados:**
   - Peso inputado atualiza `userMacros`?
   - Tabela renderiza com dados corretos?
   - Cálculos produzem resultados esperados?
   - Histórico salva estrutura completa?
4. **Testar ordenação/filtragem** - Verificar operações de array
5. **Validar links WhatsApp** - Testar compartilhamento real para garantir encoding
6. **Verificar localStorage** - DevTools > Application > Local Storage
7. **Testar formatação** - Enviar mensagem WhatsApp de teste

## Recursos Adicionais

### Padrões Referenciados
- **Ingestão de proteína:** [Diretrizes ISSN (1.6-2.2g/kg)](https://jissn.biomedcentral.com/articles/10.1186/s12970-017-0177-8)
- **Fórmula TMB:** [Mifflin-St Jeor (PubMed)](https://pubmed.ncbi.nlm.nih.gov/2305711/)
- **Fatores de atividade:** [Harris-Benedict (PNAS)](https://www.pnas.org/doi/10.1073/pnas.1305908110)
- **Idioma:** Português (Brasil) - pt-BR
- **Moeda:** Real Brasileiro (R$)

### Links Úteis
- Especificação HTML5: https://html.spec.whatwg.org/
- Referência JavaScript MDN: https://developer.mozilla.org/pt-BR/docs/Web/JavaScript
- CSS Custom Properties: https://developer.mozilla.org/pt-BR/docs/Web/CSS/--*
- API WhatsApp: https://faq.whatsapp.com/general/chats/how-to-use-click-to-chat
- LocalStorage API: https://developer.mozilla.org/pt-BR/docs/Web/API/Window/localStorage

## Histórico de Versões

**Versão Atual:** Commit mais recente em `claude/add-claude-documentation-0yi0K`

**Mudanças Recentes Notáveis:**
- **2026-01-22:** Salva plano completo no histórico e corrige formatação WhatsApp
- **2026-01-22:** Adiciona seleção de 2 proteínas e visualização detalhada do histórico
- **2026-01-22:** Implementa planejador inteligente de refeições com sugestões automáticas
- **2026-01-21:** Adiciona referências científicas à Calculadora de Macros
- **2026-01-21:** TRANSFORMAÇÃO RADICAL: ProteínaMax - App Nutricional Completo 🚀
  - Expansão de 14 para 32 alimentos
  - Adição de calculadora TMB/TDEE
  - Dashboard visual
  - Sistema de favoritos
  - Comparador de alimentos
  - 10 receitas proteicas
  - Histórico de planos
  - Sistema de planejamento inteligente

**Status de Desenvolvimento:** Ativo
**Última Atualização deste Documento:** 2026-01-22

---

## Cartão de Referência Rápida

**Arquivo:** `index.html` apenas
**Idioma:** Português (pt-BR)
**Framework:** Nenhum (Vanilla JS)
**Dependências:** Nenhuma
**Build Process:** Nenhum
**Testes:** Manuais apenas
**Deploy:** Hospedagem estática

**Funções-Chave:**
- `calculateMacros()` - Calcular TMB/TDEE e macros
- `renderFoodsTable()` - Renderizar tabela de alimentos
- `initMealPlanner()` - Inicializar planejador
- `calculateSmartMealPlan()` - Gerar plano inteligente
- `savePlanToHistory()` - Salvar no histórico
- `viewHistoryDetails(index)` - Ver detalhes do plano
- `shareHistoryToWhatsApp(index)` - Compartilhar no WhatsApp
- `formatQuantity()` - Formatar com unidade correta
- `calculateRealGrams()` - Aplicar limites realistas

**Arrays de Dados:**
- `foodsDatabase` (linha ~850) - 32 alimentos
- `carbsDatabase` (linha ~1150) - 12 carboidratos
- `fatsDatabase` (linha ~1200) - 10 gorduras
- `foodUnits` (linha ~1125) - Unidades e limites
- `mealSuggestions` (linha ~1250) - Sugestões contextuais

**Variáveis de Estado:**
- `userMacros` - Metas calculadas do usuário
- `favorites` - IDs favoritos (localStorage)
- `history` - Planos salvos (localStorage, máx 30)
- `currentTheme` - Tema atual
- `window.currentPlan` - Plano gerado atual

---

*Esta documentação é mantida para assistentes de IA trabalhando neste codebase. Mantenha-a atualizada ao fazer mudanças significativas.*
