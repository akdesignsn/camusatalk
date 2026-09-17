# CAMUSAT TALK

**CAMUSAT TALK** est une application interne développée pour **Camusat Sénégal**. Elle sert de plateforme au programme du même nom : un rendez-vous de prise de parole, de partage d'expérience et de développement des compétences, dans lequel les collaborateurs proposent et présentent un sujet, accompagnés d'un mentor.

> Share. Learn. Inspire.

L'application permet à un collaborateur de proposer un sujet en moins de deux minutes, à tout le monde de consulter les présentations en cours (filtrées par thématique et par statut), et à l'équipe RH / Direction de suivre et piloter l'ensemble des propositions depuis un tableau de bord dédié. **Aucun compte n'est requis pour proposer un sujet** : n'importe qui avec le lien peut soumettre une proposition.

## Fonctionnalités

- **Page d'accueil** : présentation du programme, statistiques en direct, accès rapide aux deux actions principales.
- **Formulaire de proposition** : informations du présentateur, sujet, thématique, mentor accompagnateur, objectif, durée et date souhaitée — avec validation des champs et message de confirmation.
- **Page Présentateurs** : les propositions sous forme de cartes, avec filtres par thématique et par statut, et une recherche libre.
- **Dashboard administrateur** (protégé par un code d'accès) : vue d'ensemble chiffrée, modification du statut et du mentor de chaque proposition, suppression d'une proposition, export des données en CSV et en Excel (.xlsx).
- Identité visuelle Camusat (bleu foncé `#203261`, rouge accent), interface responsive (mobile / tablette / desktop), thème clair et sombre.

## Technologies utilisées

- HTML5 / CSS3 (variables CSS, grille, media queries) — aucun framework CSS, styles écrits à la main.
- JavaScript (ES5/ES6, vanilla — sans framework front-end).
- Police [Google Fonts](https://fonts.google.com/) : Sora (titres) et Inter (texte).
- [SheetJS / xlsx](https://github.com/SheetJS/sheetjs) (chargé depuis un CDN, uniquement au moment de l'export Excel).
- **[Supabase](https://supabase.com/)** ([supabase-js](https://github.com/supabase/supabase-js) v2, chargé depuis un CDN) : base de données partagée où sont stockées les propositions — voir la section suivante.

## Comment ça marche : une application autonome, hébergeable n'importe où

Contrairement à une version antérieure de ce projet, `index.html` ne dépend d'aucune fonction propre à Claude : c'est une page web autonome, hébergeable sur GitHub Pages, Netlify, Vercel ou n'importe quel hébergeur statique. Le stockage des propositions passe par un projet **Supabase** (PostgreSQL + API auto-générée), configuré directement dans le code via deux constantes en haut du script :

```js
var SUPABASE_URL = "https://zisitmonkozxnjlioaga.supabase.co";
var SUPABASE_ANON_KEY = "sb_publishable_...";
```

`SUPABASE_ANON_KEY` est une clé **publique** par conception (visible dans le code source de la page, comme n'importe quelle clé côté client) : la vraie protection vient des règles de sécurité (RLS, *Row Level Security*) définies sur la table `propositions` dans le projet Supabase, pas du secret de cette clé. **Ne jamais** remplacer cette valeur par la clé secrète (`sb_secret_...` / anciennement `service_role`) : celle-ci donnerait un accès total et sans filtre à la base à quiconque lit cette page.

### Reproduire la base de données Supabase

Si tu dois recréer le projet Supabase (nouveau projet, migration, environnement de test), exécute ce script SQL dans l'éditeur SQL de Supabase :

```sql
create table propositions (
  id uuid primary key default gen_random_uuid(),
  presenter_name text not null,
  presenter_role text not null,
  presenter_email text not null,
  title text not null,
  description text not null,
  theme text not null,
  mentor_name text not null,
  mentor_dept text not null,
  objective text not null,
  duration text not null,
  desired_date date not null,
  status text not null default 'attente',
  created_at timestamptz not null default now()
);

alter table propositions enable row level security;

create policy "public insert" on propositions for insert with check (true);
create policy "public select" on propositions for select using (true);
create policy "public update" on propositions for update using (true) with check (true);
create policy "public delete" on propositions for delete using (true);
```

Puis copie **Project URL** et la clé **anon public / publishable** depuis *Project Settings → API Keys*, et remplace les deux constantes dans `index.html`.

**Note sur le modèle de confiance** : ces règles RLS sont volontairement ouvertes (comme demandé : aucun compte requis pour soumettre une proposition), donc la protection réelle repose sur le fait que l'URL du projet et la clé publique restent connues seulement des personnes ayant le lien de l'application — exactement le même niveau de protection que le code d'accès du Dashboard (`ADMIN_CODE`). Ce n'est pas un mécanisme de sécurité de niveau entreprise ; pour des données sensibles, il faudrait ajouter une vraie authentification Supabase (hors périmètre de cette version).

### Rafraîchissement des données

L'application interroge Supabase à l'ouverture, puis toutes les 15 secondes (`SUPABASE_POLL_MS`), et immédiatement après chaque ajout/modification/suppression — pas de mise à jour instantanée seconde par seconde entre deux personnes connectées en même temps, mais un délai maximal de 15 secondes pour voir apparaître les nouvelles propositions des autres.

### Historique : ancienne version "Claude Artifact"

Une version antérieure de cette application était conçue pour tourner comme **Claude Artifact** (stockage via les capacités `db`/`downloads`/`user` de Claude). Cette approche a été abandonnée pour l'usage réel : elle exige que chaque personne soit connectée à Claude et membre de la même organisation, ce qui exclut les collaborateurs sans compte Claude. La version Supabase de ce dépôt n'a plus cette limite.

## Installation locale

```bash
git clone https://github.com/<ton-compte>/camusat-talk-sn.git
cd camusat-talk-sn
```

Aucune installation de dépendances n'est nécessaire (pas de `npm install`) : Supabase est chargé directement depuis un CDN dans la page.

## Lancer l'application

**Option 1 — ouverture directe**
Double-clique sur `index.html`, ou ouvre-le depuis ton navigateur. L'application est pleinement fonctionnelle (le stockage Supabase ne dépend pas d'un hébergement particulier).

**Option 2 — via un petit serveur local** (évite certaines restrictions de navigateur sur les fichiers `file://`) :

```bash
python3 -m http.server 8000
# puis ouvrir http://localhost:8000
```

**Option 3 — en ligne (production)**
Déploie `index.html` sur GitHub Pages (*Settings → Pages*, branche `main`, dossier `/`), ou sur Netlify/Vercel en glissant le dossier. C'est le mode d'usage réel à partager avec les collaborateurs.

## Structure du projet

```
camusat-talk-sn/
├── index.html              # Application complète (HTML + CSS + JS + intégration Supabase)
├── README.md                # Ce fichier
├── .gitignore                # Adapté à un usage React/Next.js si le projet évolue
└── docs/
    └── screenshots/          # Captures d'écran de référence (accueil, formulaire, présentateurs, dashboard)
```

## Compte administrateur (Dashboard)

Le Dashboard est protégé par un code d'accès défini dans `index.html` (constante `ADMIN_CODE`, valeur par défaut : `CAMUSAT2026`). Ce n'est **pas** un mécanisme de sécurité réel — comme tout code présent dans une page web, il est visible par quiconque consulte le source. Change cette valeur avant toute diffusion large de l'application.

## Limites connues / pistes d'évolution

- Pas de suite de tests automatisés.
- Pas de vraie authentification par utilisateur (l'identité du présentateur est simplement saisie dans le formulaire, et n'importe qui connaissant le code peut accéder au Dashboard).
- Les règles Supabase (RLS) sont ouvertes en lecture/écriture : suffisant pour un outil interne diffusé par lien, mais pas pour des données sensibles.
- Rafraîchissement par sondage toutes les 15 secondes plutôt que du temps réel instantané (réalisable via les canaux *Realtime* de Supabase si besoin, non activés ici).
- Pour une évolution plus poussée (comptes individuels, journal d'audit, notifications par e-mail...), Supabase propose l'authentification et les *Edge Functions* nécessaires ; ou une reconstruction en React/Next.js reste possible en gardant la même base Supabase.

## Captures d'écran

| Accueil | Proposer un sujet |
|---|---|
| ![Accueil](docs/screenshots/accueil.png) | ![Formulaire](docs/screenshots/proposer-un-sujet.png) |

| Présentateurs | Dashboard |
|---|---|
| ![Présentateurs](docs/screenshots/presentateurs.png) | ![Dashboard](docs/screenshots/dashboard.png) |

---

Projet interne — Camusat Sénégal, Bureau d'Études.
