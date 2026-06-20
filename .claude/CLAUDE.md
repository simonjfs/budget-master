# Budget Tracker — Claude Code Context

## Projet

Application web personnelle de gestion de comptes bancaires :
- Import de fichiers CSV/OFX exportés depuis la banque
- Catégorisation des transactions (manuelle + règles automatiques)
- Analyse des dépenses mois par mois
- Visualisations (charts, tendances)

Objectifs : outil utile au quotidien + référence personnelle sur cette stack moderne + montée en compétences continue.

---

## Stack cible

### Frontend
- **React 18 + TypeScript + Vite**
- **shadcn/ui + Tailwind CSS** pour les composants UI
- **Recharts** pour les visualisations
- **Zustand** pour le state management (léger, moderne)
- **React Query (TanStack Query)** pour la gestion des appels API

### Backend
- **Node.js + Hono + TypeScript** (API REST légère et moderne)
- **Drizzle ORM** (type-safe, TypeScript-first)
- **SQLite** (fichier local, pas de serveur à gérer)

### Parsing
- **papaparse** pour les fichiers CSV
- **ofx-js** pour les fichiers OFX (format bancaire standard)

### Monorepo
- **pnpm workspaces** (simple, sans Nx ni Turborepo au départ)

---

## Structure cible du monorepo

```
/
├── apps/
│   ├── web/          # React frontend (Vite)
│   └── api/          # Hono backend
├── packages/
│   ├── db/           # Schéma Drizzle + migrations
│   └── domain/       # Types partagés, règles métier
├── pnpm-workspace.yaml
├── package.json
└── CLAUDE.md
```

---

## Domain model (point de départ)

```typescript
// Account — un compte bancaire importé
type Account = {
  id: string
  name: string        // ex: "Compte courant BNP"
  currency: string    // "EUR"
  createdAt: Date
}

// Transaction — une opération bancaire
type Transaction = {
  id: string
  accountId: string
  date: Date
  label: string       // libellé brut de la banque
  amount: number      // négatif = dépense, positif = crédit
  categoryId?: string
  source: 'csv' | 'ofx'
  importedAt: Date
}

// Category — catégorie de dépense
type Category = {
  id: string
  name: string        // ex: "Alimentation", "Transport"
  color: string
  parentId?: string   // pour les sous-catégories
}

// Rule — règle de catégorisation automatique
type Rule = {
  id: string
  pattern: string     // pattern sur label (ex: "CARREFOUR")
  categoryId: string
  priority: number
}
```

---

## Principes à respecter

- **TypeScript strict** partout — pas de `any`
- **Séparation claire** domain / infrastructure / présentation
- **API REST bien nommée** : ressources nommées au pluriel, verbes HTTP sémantiques
- Commits atomiques et messages clairs — ce repo sert de référence, le code doit rester lisible dans le temps
- Pas de sur-ingénierie : SQLite suffit, pas besoin de PostgreSQL pour ce scope
- Commentaires en **français** dans le code métier, **anglais** dans le code technique

---

## Par où commencer (ordre suggéré)

1. **Initialisation monorepo** — pnpm workspaces, tsconfig partagé, eslint
2. **Package `domain`** — types partagés, validations Zod
3. **Package `db`** — schéma Drizzle, migrations SQLite
4. **App `api`** — endpoints CRUD transactions, comptes, catégories, règles
5. **App `web`** — UI : import CSV/OFX, liste transactions, catégorisation
6. **Feature catégorisation auto** — moteur de règles sur les libellés
7. **Feature analytics** — agrégats par mois, par catégorie, charts Recharts

---

## Routes API cibles

```
GET    /api/accounts
POST   /api/accounts
GET    /api/accounts/:id/transactions
POST   /api/accounts/:id/import        # upload CSV ou OFX

GET    /api/transactions
PATCH  /api/transactions/:id           # mise à jour catégorie
DELETE /api/transactions/:id

GET    /api/categories
POST   /api/categories
PUT    /api/categories/:id
DELETE /api/categories/:id

GET    /api/rules
POST   /api/rules
PUT    /api/rules/:id
DELETE /api/rules/:id

GET    /api/analytics/monthly          # dépenses par mois
GET    /api/analytics/by-category      # dépenses par catégorie
```

---

## Ce qu'on veut éviter

- Next.js (inutile sans SSR/SEO)
- Connexion directe API bancaire (Bridge, Plaid) — hors scope v1
- Auth utilisateur — app locale, pas multi-user
- Over-engineering : pas de message queue, pas de cache distribué, pas de microservices
