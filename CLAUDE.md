# CLAUDE.md - AI Assistant Guide for Proteína Eficiente

## Project Overview

**Proteína Eficiente** is a single-page web application designed to help users optimize their protein intake for muscle building (hypertrophy). The app calculates daily protein needs based on body weight and helps users plan meals by comparing cost-effectiveness of different protein sources.

**Primary Language:** Portuguese (Brazil)

**Target Users:** Fitness enthusiasts, bodybuilders, and anyone tracking protein intake for health goals

**Key Features:**
- Daily protein requirement calculator (2.0g/kg for hypertrophy)
- Protein source comparison table with cost analysis
- Interactive meal planner with 6 meals/day
- WhatsApp sharing functionality for meal plans
- Dark/Light theme support
- Mobile-first responsive design

## Codebase Structure

```
proteina-app/
├── .git/                 # Git repository
│   └── hooks/           # Standard git hooks (samples only)
├── index.html           # ENTIRE APPLICATION (HTML + CSS + JavaScript)
└── CLAUDE.md           # This file
```

### Architecture: Single-File Application

**CRITICAL:** This is a monolithic single-page application. All code lives in `index.html`:
- **Lines 1-204:** HTML structure and embedded CSS styles
- **Lines 205-294:** HTML body content (UI elements)
- **Lines 296-612:** Embedded JavaScript (application logic)

**No external dependencies:**
- No package.json
- No build process
- No npm/yarn
- No frameworks (React, Vue, etc.)
- No CSS preprocessors
- No bundlers (Webpack, Vite, etc.)

**Deployment:** Simply serve `index.html` via any web server or open directly in browser.

## Technology Stack

### Frontend
- **HTML5:** Semantic markup with Portuguese labels
- **CSS3:**
  - CSS Custom Properties (CSS Variables) for theming
  - Flexbox and Grid layouts
  - Mobile-first responsive design
  - Embedded in `<style>` tag (lines 7-203)

- **JavaScript (ES6+):**
  - Pure vanilla JavaScript (no libraries)
  - Modern features: arrow functions, template literals, destructuring
  - DOM manipulation APIs
  - LocalStorage NOT used (state resets on reload)

### Browser APIs Used
- `document.querySelector/querySelectorAll` - DOM selection
- `addEventListener/onclick` - Event handling
- `window.open()` - WhatsApp sharing
- `window.location.href` - Current URL for sharing
- `scrollIntoView()` - Smooth scrolling to results

### No Backend
- Static HTML file only
- All calculations performed client-side
- No database
- No API calls
- No server-side logic

## Data Model

### Main Data Structure: `dados` Array (line 298)

Each food item is an object with the following schema:

```javascript
{
  id: number,              // Unique identifier (1-14)
  fonte: string,           // Food name in Portuguese
  categoria: string,       // Category: 'carne', 'frango', 'peixe', 'suplemento', 'outros'
  proteina: number,        // Protein content in grams
  porcaoDesc: string,      // Portion description (e.g., '100g', '1 unid')
  gPorUnidade: number,     // Grams per unit (for calculations)
  unitType: string,        // 'gramas', 'unidade', 'scoop', 'colher'
  custo: number,           // Cost in Brazilian Reais (R$)
  custoProteina: number    // Cost per gram of protein (R$/g)
}
```

**Example:**
```javascript
{
  id: 5,
  fonte: 'Peito frango (Grelh.)',
  categoria: 'frango',
  proteina: 32,
  porcaoDesc: '100g',
  gPorUnidade: 100,
  unitType: 'gramas',
  custo: 1.5,
  custoProteina: 0.046
}
```

### Meals Configuration: `mealsConfig` Array (line 315)

Defines 6 daily meals with protein distribution weights:

```javascript
{
  key: string,     // Unique meal identifier
  label: string,   // Display name with emoji
  weight: number   // Protein distribution weight (0.10 to 0.30)
}
```

**Total weight must sum to 1.0 (100%)**

### State Management

**Global State Variables** (lines 324-327):
- `filtroCategoriaAtual: string` - Current category filter
- `ordenacaoAtual: string` - Current sort column ('custoProteina' default)
- `direcaoOrdenacao: string` - Sort direction ('asc' or 'desc')
- `currentTheme: string` - Current theme ('dark' or 'light')

**No persistence:** All state is lost on page reload.

## Key Features & Functionality

### 1. Protein Calculator (lines 329-338)

**Function:** `atualizarGlobal()`
- Triggered by weight input change
- Formula: `protein_goal = weight_kg × 2.0`
- Based on scientific reference: 1.6-2.2g/kg (Journal of Sports Nutrition)
- Validation: Weight must be > 30kg

### 2. Table Sorting (lines 400-408)

**Function:** `ordenar(campo)`
- Bidirectional sorting (ascending/descending)
- Sortable columns: 'fonte', 'proteina', 'custoProteina'
- Visual indicators with arrows (▲▼)
- Default sort: by `custoProteina` ascending (cheapest first)

### 3. Category Filtering (lines 393-398)

**Function:** `filtrarCategoria(cat)`
- Categories: all, carne, frango, peixe, suplemento, outros
- Updates dynamic stats (cheapest, highest protein in filtered set)
- Active category highlighted with primary color

### 4. Search Functionality (lines 426-433)

**Function:** `filtrarTabela()`
- Real-time search as user types
- Searches across all visible text in rows
- Case-insensitive matching
- Filters table rows without re-rendering

### 5. Meal Planner (lines 440-458, 460-607)

**Core Logic:**

**A. Initialization** (`initPlanner()`):
- Creates 6 meal cards
- Each card has 3 food selection dropdowns
- Populated from `dados` array

**B. Calculation** (`calcularDieta()`):
- **Step 1:** Calculate total protein goal (weight × 2.0)
- **Step 2:** Identify active meals (meals with at least one food selected)
- **Step 3:** Distribute protein proportionally based on `meal.weight`
- **Step 4:** For each meal, divide protein equally among selected foods
- **Step 5:** Calculate required grams based on protein density
- **Step 6:** Apply portion limits:
  - Tuna (Atum): max 45g
  - Cottage: max 60g (max 2 tablespoons)
- **Step 7:** Handle unit conversions:
  - Round units/scoops to whole numbers
  - Round tablespoons to nearest 0.5
  - Display grams for weight-based foods
- **Step 8:** Track protein deficit and roll forward to next meals
- **Step 9:** Generate summary with totals

**Deficit Rolling Logic** (lines 487-545):
- If a meal can't meet its protein target (due to limits), the deficit carries forward
- Next meals get extra protein allocation to compensate
- Ensures maximum protein delivery within food constraints

### 6. WhatsApp Sharing

**Two sharing modes:**

**A. Share Table** (`compartilharTabelaZap()` - lines 435-438):
- Shares app URL with message
- Opens WhatsApp web/app in new tab

**B. Share Meal Plan** (lines 590-594):
- Generates detailed meal plan text
- Uses Unicode escapes for emojis (e.g., `\uD83C\uDFAF` for 🎯)
- Formats as WhatsApp message with markdown-style bold (`*text*`)
- Includes app URL footer
- Uses `api.whatsapp.com` API endpoint
- Direct link activation (not onclick handler)

**IMPORTANT:** WhatsApp sharing uses Unicode escape sequences, not literal emojis in JavaScript strings. This ensures proper encoding across all platforms.

### 7. Theme Switching (lines 340-343)

**Function:** `toggleTheme()`
- Toggles between 'light' and 'dark' themes
- Updates `data-theme` attribute on `<html>` element
- CSS variables automatically update via `:root[data-theme="..."]`
- Default: Dark theme

### 8. Tab Navigation (lines 345-350)

**Function:** `switchTab(tabName)`
- Two tabs: 'fontes' (protein sources) and 'planejamento' (meal planning)
- Shows/hides content sections with fade-in animation
- Updates active tab styling

## Development Workflows

### Making Changes

1. **Edit `index.html` directly** - all code is in this single file
2. **Test in browser** - refresh to see changes
3. **No build step required** - changes are immediate

### Adding New Food Items

**Location:** `dados` array (line 298)

**Steps:**
1. Add new object to `dados` array
2. Assign next sequential `id`
3. Fill all required fields (see Data Model section)
4. Calculate `custoProteina = custo / proteina`
5. Choose appropriate `categoria` and `unitType`
6. Refresh browser to see changes

**Example:**
```javascript
{
  id: 15,
  fonte: 'Salmão grelhado',
  categoria: 'peixe',
  proteina: 25,
  porcaoDesc: '100g',
  gPorUnidade: 100,
  unitType: 'gramas',
  custo: 8.5,
  custoProteina: 0.340
}
```

### Adding New Meal Slots

**Location:** `mealsConfig` array (line 315)

**Requirements:**
- All `weight` values must sum to 1.0
- Use emoji icons for visual appeal
- Key must be unique and URL-safe (no spaces)

### Modifying Styling

**CSS Variables** (lines 9-35):
- Light theme: `:root[data-theme="light"]`
- Dark theme: `:root[data-theme="dark"]`

**To change colors:**
1. Locate the appropriate CSS variable
2. Update color value in both theme sections
3. Changes apply globally via `var(--variable-name)`

### Modifying Calculations

**Protein multiplier** (line 333, 464):
- Current: 2.0g/kg
- To change: update both instances in `atualizarGlobal()` and `calcularDieta()`

**Portion limits** (lines 514-521):
- Tuna: 45g max
- Cottage: 60g max
- To adjust: modify if conditions in `calcularDieta()`

## Git Conventions

### Commit Messages

**Language:** Portuguese (Brazil)

**Style:** Imperative mood, descriptive
- ✅ "Adiciona ordenação bidirecional na tabela de fontes"
- ✅ "Fix encoding de emojis para WhatsApp usando Unicode Escapes"
- ✅ "Corrige cálculo de proteína real e lógica de arredondamento"
- ❌ "Updated stuff"
- ❌ "WIP"

**Pattern:** `[Action] [specific change]`
- Common actions: `Adiciona`, `Fix`, `Corrige`, `Atualiza`, `Remove`

### Branch Naming

**AI Assistant Branches:** `claude/[descriptive-name]-[session-id]`
- Example: `claude/add-claude-documentation-0yi0K`
- Session ID is auto-generated and must match for push to succeed

**Convention:**
- Feature branches: descriptive kebab-case
- No version numbers in branch names
- Keep branch names concise but clear

### Git Workflow

1. **Always check current branch** before making changes
2. **Commit frequently** with clear messages
3. **Push to designated branch** using `git push -u origin <branch-name>`
4. **Never force push** to main/master
5. **Use descriptive commit messages** in Portuguese

**For AI Assistants:**
- Develop on specified claude/* branch
- Commit after logical changes
- Push when work is complete
- Create PR with summary in Portuguese

## Coding Conventions

### JavaScript Style

**Naming Conventions:**
- **Functions:** camelCase, descriptive verbs
  - `atualizarGlobal()`, `calcularDieta()`, `renderizarTabela()`
- **Variables:** camelCase, descriptive nouns
  - `filtroCategoriaAtual`, `direcaoOrdenacao`, `metaTotal`
- **Constants:** camelCase (not UPPER_CASE in this codebase)
  - `dados`, `mealsConfig`

**Code Organization:**
1. Data declarations first (`dados`, `mealsConfig`)
2. State variables
3. Initialization functions
4. Event handlers
5. Helper functions
6. Main execution at bottom

**Functions:**
- Keep functions focused (single responsibility)
- Use descriptive names that indicate action
- Prefer arrow functions for callbacks
- No semicolons (style is inconsistent but leans toward optional)

### HTML Conventions

**Attributes Order:**
1. id
2. class
3. data-* attributes
4. other attributes
5. event handlers (onclick, onchange)

**IDs:** Use kebab-case
- `peso-usuario`, `result-area`, `plan-text`

**Classes:** Use kebab-case
- `weight-card`, `nav-tabs`, `btn-action`

### CSS Conventions

**Variables:** Use CSS custom properties for theming
- Colors: `--primary`, `--bg`, `--text`, `--border`
- Always define for both light and dark themes

**Class naming:** BEM-inspired but simplified
- Block: `.meal-card`
- Element: `.meal-title`, `.meal-selects`
- Modifier: `.nav-tab.active`, `.filter-btn.active`

**Units:**
- Font sizes: `px` (not rem)
- Spacing: `px`
- Borders: `px`
- Border radius: `px`

### Portuguese Language

**All user-facing text must be in Portuguese (Brazil):**
- UI labels
- Button text
- Error messages
- Comments in code
- Console logs (if any)

**Keep variable/function names in English or Portuguese:**
- This codebase uses Portuguese: `dados`, `fonte`, `proteina`
- Be consistent with existing naming

## Testing & Deployment

### Testing

**No automated tests exist.**

**Manual testing checklist:**
1. Open `index.html` in browser
2. Test weight calculator:
   - Enter valid weight (> 30kg)
   - Verify protein calculation (weight × 2.0)
3. Test table features:
   - Sort by each column
   - Filter by category
   - Search functionality
4. Test meal planner:
   - Select foods for multiple meals
   - Calculate diet plan
   - Verify totals and quantities
   - Test portion limits (atum, cottage)
5. Test WhatsApp sharing:
   - Share table
   - Share meal plan (verify emojis render correctly)
6. Test theme switching:
   - Toggle light/dark
   - Verify all colors update
7. Test responsive design:
   - Mobile viewport (320px+)
   - Tablet viewport
   - Desktop viewport

### Browser Compatibility

**Target Browsers:**
- Chrome/Edge (latest)
- Firefox (latest)
- Safari (latest)
- Mobile browsers (iOS Safari, Chrome Mobile)

**Features requiring modern browser:**
- CSS Grid
- CSS Custom Properties
- ES6+ JavaScript features
- Flexbox

**No IE11 support** (uses modern CSS and JS)

### Deployment

**Static Hosting Options:**
- GitHub Pages
- Netlify
- Vercel
- Any web server
- Can even run from `file://` URL

**Steps:**
1. Commit changes to repository
2. Push to hosting service or web server
3. No build step required
4. `index.html` is the entry point

## Common Tasks & Examples

### Task 1: Add a New Protein Source

```javascript
// In dados array (line 298), add:
{
  id: 15,
  fonte: 'Filé de frango',
  categoria: 'frango',
  proteina: 31,
  porcaoDesc: '100g',
  gPorUnidade: 100,
  unitType: 'gramas',
  custo: 2.0,
  custoProteina: 0.064
}
```

### Task 2: Change Default Sort Order

```javascript
// Line 325 - change to sort by protein content descending:
let ordenacaoAtual = 'proteina';
let direcaoOrdenacao = 'desc';
```

### Task 3: Adjust Protein Multiplier

```javascript
// Line 333 in atualizarGlobal():
const meta = (peso * 2.2).toFixed(0); // Changed from 2.0 to 2.2

// Line 464 in calcularDieta():
const metaTotal = peso * 2.2; // Changed from 2.0 to 2.2
```

### Task 4: Add New Category

```javascript
// 1. Add food with new category to dados array
{ id: 15, fonte: 'Lentilhas', categoria: 'leguminosa', ... }

// 2. Add filter button in HTML (line ~254):
<button class="filter-btn" onclick="filtrarCategoria('leguminosa')">🥜 Legum.</button>
```

### Task 5: Modify Meal Distribution

```javascript
// Line 315 - adjust weight values (must sum to 1.0):
const mealsConfig = [
  { key: 'cafe', label: '☕ Café da Manhã', weight: 0.20 },      // increased
  { key: 'lancheManha', label: '🍎 Lanche Manhã', weight: 0.10 },
  { key: 'almoco', label: '🍽️ Almoço', weight: 0.25 },           // decreased
  { key: 'lancheTarde', label: '🪪 Lanche Tarde', weight: 0.10 },
  { key: 'jantar', label: '🍲 Jantar', weight: 0.25 },
  { key: 'ceia', label: '🌙 Ceia', weight: 0.10 }
];
```

### Task 6: Change Theme Colors

```javascript
// Lines 9-35 - modify CSS variables
:root[data-theme="dark"] {
  --primary: #4ade80;      // Change primary color
  --bg: #0a0a0a;          // Darker background
  --surface: #1a1a1a;     // Darker surface
  --text: #ffffff;        // Brighter text
}
```

## Important Notes for AI Assistants

### Critical Rules

1. **NEVER split the file** - This is intentionally a monolithic single-file app
2. **Test in browser** - Changes cannot be validated without visual inspection
3. **Preserve Portuguese** - All user-facing text must remain in Portuguese
4. **Maintain data structure** - Don't change object schemas without updating all usages
5. **Calculate custoProteina** - Always compute `custo / proteina` when adding food items
6. **Sum weights to 1.0** - Meal weights must always total 100%
7. **Use Unicode escapes for emojis** in JavaScript strings for WhatsApp (not literal emojis)
8. **Commit messages in Portuguese** - Follow existing commit style

### Common Pitfalls

❌ **Don't:** Create separate .js, .css files
✅ **Do:** Edit the embedded code in index.html

❌ **Don't:** Add dependencies or npm packages
✅ **Do:** Use vanilla JavaScript only

❌ **Don't:** Assume data persists across page reloads
✅ **Do:** Understand this is session-only state

❌ **Don't:** Use English in user-facing text
✅ **Do:** Keep all UI text in Portuguese

❌ **Don't:** Add build tools or transpilation
✅ **Do:** Write browser-compatible ES6+ code directly

### Best Practices

1. **Before editing:** Read the entire function to understand context
2. **After editing:** Check for side effects in other functions
3. **Emoji handling:** Use Unicode escapes (`\uD83C\uDF7D\uFE0F`) for WhatsApp, HTML entities (`&#x1F4CA;`) for HTML
4. **Testing:** Always provide manual testing steps for changes
5. **Documentation:** Update this file when making architectural changes
6. **Protein calculations:** Double-check math for accuracy
7. **Responsive design:** Test changes on mobile viewports

### When Making Changes

**Always verify:**
- [ ] Code is valid ES6+ JavaScript
- [ ] HTML structure is semantically correct
- [ ] CSS doesn't break existing layout
- [ ] All text is in Portuguese (except code)
- [ ] Emojis render correctly in both UI and WhatsApp
- [ ] Math calculations are accurate
- [ ] No console errors
- [ ] Mobile layout still works
- [ ] Theme switching still works
- [ ] Data structure integrity maintained

### Debugging Tips

1. **Open browser console** - Check for JavaScript errors
2. **Inspect element** - Verify CSS is applied correctly
3. **Check data flow:**
   - Is weight input updating `metaProteinaDisplay`?
   - Is table rendering with correct data?
   - Are calculations producing expected results?
4. **Test sorting/filtering** - Verify array operations work correctly
5. **Validate WhatsApp links** - Test actual sharing to ensure encoding is correct

## Additional Resources

### Referenced Standards
- **Protein intake:** [ISSN Guidelines (1.6-2.2g/kg)](https://link.springer.com/article/10.1186/s12970-018-0215-1)
- **Language:** Portuguese (Brazil) - pt-BR
- **Currency:** Brazilian Real (R$)

### Useful Links
- HTML5 Spec: https://html.spec.whatwg.org/
- MDN JavaScript Reference: https://developer.mozilla.org/en-US/docs/Web/JavaScript
- CSS Custom Properties: https://developer.mozilla.org/en-US/docs/Web/CSS/--*
- WhatsApp API: https://faq.whatsapp.com/5913398998672934/

## Version History

**Current Version:** Latest commit on branch `claude/add-claude-documentation-0yi0K`

**Recent Notable Changes:**
- Fix: WhatsApp button converted to direct link (href) instead of onclick
- Fix: Emoji encoding for WhatsApp using Unicode escapes
- Feature: Separate text formatting for screen display vs WhatsApp
- Feature: Bidirectional sorting (Asc/Desc) on table columns
- Feature: Dark theme as default with dynamic rebalancing

**Development Status:** Active
**Last Updated:** 2026-01-21

---

## Quick Reference Card

**File:** `index.html` only
**Language:** Portuguese (pt-BR)
**Framework:** None (Vanilla JS)
**Dependencies:** None
**Build Process:** None
**Testing:** Manual only
**Deployment:** Static hosting

**Key Functions:**
- `atualizarGlobal()` - Update protein goal display
- `calcularDieta()` - Calculate meal plan
- `renderizarTabela()` - Render protein table
- `ordenar(campo)` - Sort table by column
- `filtrarCategoria(cat)` - Filter by category
- `toggleTheme()` - Switch light/dark theme

**Data Arrays:**
- `dados` (line 298) - 14 protein sources
- `mealsConfig` (line 315) - 6 daily meals

**State Variables:**
- `filtroCategoriaAtual` - Current filter
- `ordenacaoAtual` - Current sort column
- `direcaoOrdenacao` - Sort direction (asc/desc)
- `currentTheme` - Current theme (light/dark)

---

*This documentation is maintained for AI assistants working on this codebase. Keep it updated when making significant changes.*
