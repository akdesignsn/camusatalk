# CAMUSAT TALK

**CAMUSAT TALK** est une application interne développée pour **Camusat Sénégal**. Elle sert de plateforme au programme du même nom : un rendez-vous de prise de parole, de partage d'expérience et de développement des compétences, dans lequel les collaborateurs proposent et présentent un sujet, accompagnés d'un mentor.

> Share. Learn. Inspire.

L'application permet à un collaborateur de proposer un sujet en moins de deux minutes, à tout le monde de consulter les présentations en cours (filtrées par thématique et par statut), et à l'équipe RH / Direction de suivre et piloter l'ensemble des propositions depuis un tableau de bord dédié. **Aucun compte n'est requis pour proposer un sujet** : n'importe qui avec le lien peut soumettre une proposition.

## Fonctionnalités

- **Page d'accueil** : présentation du programme, statistiques en direct, accès rapide aux actions principales, bannière de la prochaine session, aperçu des dernières affiches et des 6 derniers médias publiés.
- **Formulaire de proposition** : informations du présentateur, sujet, thématique, mentor accompagnateur, objectif, durée et date souhaitée — avec validation des champs et message de confirmation. Accessible depuis les boutons de la page d'accueil (bandeau principal et encart de bas de page), pas depuis la barre de navigation.
- **Page Présentateurs** : les propositions sous forme de cartes, avec filtres par thématique et par statut, et une recherche libre.
- **Page Sessions** : gestion des sessions CAMUSAT TALK (une session = une date, un lieu, un lien Teams, et une organisation de rôles propre — voir plus bas), avec une bannière sur la page d'accueil annonçant la prochaine session à venir.
- **Affiches** : galerie publique accessible depuis la barre de navigation et depuis la page d'accueil (« Nos affiches »), et gestion des affiches (ajout, remplacement, suppression, association à une session) réservée à l'espace administrateur — voir plus bas.
- **Média** : galerie publique (photos + vidéos YouTube) avec filtres Tout/Photos/Vidéos, lightbox photo et lecteur vidéo intégré ; section « Nos médias » sur l'accueil (6 derniers médias publiés) ; gestion complète (ajout, modification, publication/masquage, suppression) réservée à l'espace administrateur — voir plus bas. Fonctionnalité totalement indépendante des Affiches.
- **Fiches de rôle** : depuis la page d'une session, le bloc « Mes missions » affiche directement les rôles d'observation disponibles (Grammarian, Évaluateur, Posture Visualizer, Timekeeper, Mentor) — un clic sur son rôle ouvre la fiche numérique correspondante, le nom (sans compte) n'étant demandé qu'à ce moment-là, pour identifier qui remplit. Le Mentor n'étant rattaché à aucune session en particulier (pas de lien formel entre sessions et propositions, voir plus haut), son chip ouvre la liste de tous les mentorats à remplir, tous sujets confondus ; cette fiche reste aussi accessible depuis le détail d'une proposition et directement sur chaque carte présentateur. Mêmes champs que les fiches papier d'origine (tableaux d'observation, cases à cocher, notes, tableaux répétables), sauvegarde à tout moment (« Enregistrer », « Enregistrer et continuer plus tard »), puis validation définitive (« Valider ma fiche ») qui verrouille la fiche. Statuts À remplir / En cours / Terminée, horodatage de la dernière sauvegarde, et données conservées en base (persistantes après fermeture/rechargement de l'app ; le nom est mémorisé sur l'appareil pour ne pas avoir à le ressaisir à chaque fiche). Consultation par l'administrateur depuis l'onglet « Fiches » du dashboard (toutes sessions, y compris les archives) et depuis un encart dédié sur la page de chaque session ; suppression possible depuis ces deux vues, ainsi que par la personne qui a rempli la fiche (directement sur sa fiche), avec confirmation en deux clics.
- **Dashboard administrateur** (protégé par un code d'accès) : vue d'ensemble chiffrée, modification du statut et du mentor de chaque proposition, suppression d'une proposition, export des données en CSV et en Excel (.xlsx), onglet de gestion des affiches, onglet de gestion des médias, et onglet de consultation des fiches de rôle.
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

create table sessions (
  id uuid primary key default gen_random_uuid(),
  session_number text,
  name text,
  session_datetime timestamptz,
  location text,
  teams_link text,
  description text,
  roles jsonb not null default '{}'::jsonb,
  created_at timestamptz not null default now()
);

alter table sessions enable row level security;

create policy "public insert" on sessions for insert with check (true);
create policy "public select" on sessions for select using (true);
create policy "public update" on sessions for update using (true) with check (true);
create policy "public delete" on sessions for delete using (true);

create table affiches (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  session_id uuid references sessions(id) on delete set null,
  file_url text not null,
  file_path text not null,
  mime_type text,
  created_at timestamptz not null default now()
);

alter table affiches enable row level security;

create policy "public insert" on affiches for insert with check (true);
create policy "public select" on affiches for select using (true);
create policy "public update" on affiches for update using (true) with check (true);
create policy "public delete" on affiches for delete using (true);

-- Bucket de stockage pour les fichiers des affiches (PNG/JPG/PDF).
insert into storage.buckets (id, name, public) values ('affiches', 'affiches', true)
  on conflict (id) do nothing;

create policy "public read affiches bucket" on storage.objects for select using (bucket_id = 'affiches');
create policy "public insert affiches bucket" on storage.objects for insert with check (bucket_id = 'affiches');
create policy "public update affiches bucket" on storage.objects for update using (bucket_id = 'affiches');
create policy "public delete affiches bucket" on storage.objects for delete using (bucket_id = 'affiches');

create table medias (
  id uuid primary key default gen_random_uuid(),
  type text not null check (type in ('photo','video')),
  title text not null,
  description text,
  media_date date,
  status text not null default 'publie' check (status in ('publie','masque')),
  image_url text,
  image_path text,
  youtube_url text,
  youtube_id text,
  thumbnail_url text,
  created_at timestamptz not null default now()
);

alter table medias enable row level security;

create policy "public insert" on medias for insert with check (true);
create policy "public select" on medias for select using (true);
create policy "public update" on medias for update using (true) with check (true);
create policy "public delete" on medias for delete using (true);

-- Bucket de stockage pour les photos de la rubrique Média (les vidéos ne
-- sont jamais téléversées : seul le lien YouTube et l'ID extrait sont
-- stockés, la miniature est calculée depuis img.youtube.com).
insert into storage.buckets (id, name, public) values ('medias', 'medias', true)
  on conflict (id) do nothing;

create policy "public read medias bucket" on storage.objects for select using (bucket_id = 'medias');
create policy "public insert medias bucket" on storage.objects for insert with check (bucket_id = 'medias');
create policy "public update medias bucket" on storage.objects for update using (bucket_id = 'medias');
create policy "public delete medias bucket" on storage.objects for delete using (bucket_id = 'medias');

-- Fiches de rôle (Grammarian, Mentor, Évaluateur, Posture Visualizer,
-- Timekeeper) remplies depuis l'app par les participants d'une session.
-- session_id / proposition_id restent volontairement de simples uuid (sans
-- contrainte de clé étrangère) : comme pour `affiches`/`sessions`, il n'y a
-- pas de lien formel en base entre `sessions` et `propositions` dans cette
-- app (voir plus haut), et une fiche peut se rattacher à l'un ou à l'autre
-- selon son type (roleKey "mentor" -> proposition_id ; les quatre autres
-- rôles -> session_id).
create table fiche_reponses (
  id uuid primary key default gen_random_uuid(),
  session_id uuid,
  proposition_id uuid,
  role_key text not null check (role_key in ('grammarian','mentor','evaluateur','posture_visualizer','timekeeper')),
  participant_name text not null,
  intervenant_name text,
  status text not null default 'a_remplir' check (status in ('a_remplir','en_cours','termine')),
  form_data jsonb not null default '{}'::jsonb,
  submitted_at timestamptz,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

alter table fiche_reponses enable row level security;

create policy "public insert" on fiche_reponses for insert with check (true);
create policy "public select" on fiche_reponses for select using (true);
create policy "public update" on fiche_reponses for update using (true) with check (true);
create policy "public delete" on fiche_reponses for delete using (true);
```

Puis copie **Project URL** et la clé **anon public / publishable** depuis *Project Settings → API Keys*, et remplace les deux constantes dans `index.html`.

**Si le projet Supabase existe déjà** (c'est le cas de CAMUSAT TALK en production) et que tu ajoutes seulement la fonctionnalité Média ou Fiches : il suffit d'exécuter le bloc `medias` ou `fiche_reponses` ci-dessus (table + policies, + bucket pour `medias`) dans l'éditeur SQL de ce projet — les tables déjà existantes ne sont pas concernées et n'ont pas besoin d'être rejouées.

**À propos de la table `sessions`** : elle est totalement indépendante de `propositions` — aucune clé étrangère, aucun champ dupliqué. Les rôles d'organisation (Animatrice principale, Grammarian, Timekeeper & Posture Visualizer, Évaluateurs, Responsable Feedback, Responsable Logistique & Digital, Responsable Communication, Photographe / Documentation) sont stockés dans une seule colonne `roles` au format JSON (`{ "animatrice": ["Nom"], "evaluateurs": ["Nom 1","Nom 2"], ... }`), plutôt que dans une table relationnelle séparée, pour rester cohérent avec le reste de l'application (une ligne = un enregistrement complet). Les présentateurs et mentors ne sont ni dupliqués ni ré-affichés sur les pages Sessions : ils restent uniquement rattachés aux propositions.

**À propos de la table `affiches`** et du bucket de stockage : le fichier lui-même (PNG, JPG ou PDF, 8 Mo max, contrôlé côté client dans `index.html` par `AFFICHE_MAX_BYTES`) est envoyé dans le bucket Supabase Storage `affiches` ; la table ne garde que le titre, l'URL publique du fichier (`file_url`), son chemin dans le bucket (`file_path`, utilisé pour le supprimer ou le remplacer) et le type MIME. L'association à une session est optionnelle (`session_id` nullable, `on delete set null` : si une session est supprimée, ses affiches ne sont pas perdues, seulement détachées) ; une affiche peut donc être générale ou liée à une session existante. Remplacer le fichier d'une affiche supprime l'ancien fichier du bucket après confirmation de l'envoi du nouveau ; supprimer une affiche supprime aussi son fichier du bucket.

**À propos de la table `medias`** et du bucket de stockage : chaque ligne est soit une photo (`type = 'photo'`, fichier dans le bucket `medias`, PNG/JPG/WEBP, 8 Mo max, contrôlé côté client par `MEDIA_MAX_BYTES`), soit une vidéo YouTube (`type = 'video'`, seuls `youtube_url`/`youtube_id`/`thumbnail_url` sont renseignés, aucun fichier n'est envoyé). Le statut (`status`, `publie` ou `masque`) contrôle uniquement l'affichage public (page Média et section « Nos médias » de l'accueil) ; un média masqué reste visible et modifiable dans le Dashboard. Supprimer une photo supprime aussi son fichier du bucket.

**Note sur le modèle de confiance** : ces règles RLS sont volontairement ouvertes (comme demandé : aucun compte requis pour soumettre une proposition), donc la protection réelle repose sur le fait que l'URL du projet et la clé publique restent connues seulement des personnes ayant le lien de l'application — exactement le même niveau de protection que le code d'accès du Dashboard (`ADMIN_CODE`). Ce n'est pas un mécanisme de sécurité de niveau entreprise ; pour des données sensibles, il faudrait ajouter une vraie authentification Supabase (hors périmètre de cette version).

### Rafraîchissement des données

L'application interroge Supabase à l'ouverture, puis toutes les 15 secondes (`SUPABASE_POLL_MS`), et immédiatement après chaque ajout/modification/suppression — pas de mise à jour instantanée seconde par seconde entre deux personnes connectées en même temps, mais un délai maximal de 15 secondes pour voir apparaître les nouvelles propositions ou les nouvelles sessions des autres. Comme pour le formulaire de proposition, ce rafraîchissement périodique ne réécrit jamais un champ en cours de saisie (ex. le formulaire d'ajout d'une personne à un rôle).

### Historique : ancienne version "Claude Artifact"

Une version antérieure de cette application était conçue pour tourner comme **Claude Artifact** (stockage via les capacités `db`/`downloads`/`user` de Claude). Cette approche a été abandonnée pour l'usage réel : elle exige que chaque personne soit connectée à Claude et membre de la même organisation, ce qui exclut les collaborateurs sans compte Claude. La version Supabase de ce dépôt n'a plus cette limite.

Cette ancienne version reste tenue à jour en parallèle (même fonctionnalités), mais avec une limite propre aux Affiches : n'ayant pas accès à un stockage de fichiers dédié (contrairement à Supabase Storage côté `index.html`), le fichier de l'affiche y est encodé et stocké directement dans le document (base64), ce qui plafonne la taille acceptée à 700 Ko au lieu de 8 Mo. Pour des affiches plus lourdes, utiliser la version en ligne.

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

Ce même code d'accès (le même déverrouillage que le Dashboard) protège la **création, la modification et la suppression d'une session** sur la page Sessions. En revanche, **l'ajout et le retrait d'une personne à un rôle sont ouverts à tout le monde**, sans code d'accès : n'importe qui peut consulter une session et modifier qui est affecté à chaque rôle (utile en pratique pour que les organisateurs d'une session ajustent eux-mêmes les rôles sans devoir être administrateur).

Ce même code protège aussi l'onglet **Affiches** du Dashboard (ajout, remplacement, suppression, association à une session) : seul l'administrateur peut gérer les affiches. La galerie publique (page d'accueil → « Nos affiches », et l'affiche affichée automatiquement dans les détails d'une session) reste, elle, consultable par tout le monde sans code d'accès.

Ce même code protège aussi l'onglet **Médias** du Dashboard (ajout d'une photo, ajout d'une vidéo YouTube, modification, publication/masquage, suppression) : seul l'administrateur peut gérer les médias. La page publique **Média** et la section « Nos médias » de l'accueil restent consultables par tout le monde sans code d'accès, et n'affichent que les médias au statut « Publié ».

## Limites connues / pistes d'évolution

- Pas de suite de tests automatisés.
- Pas de vraie authentification par utilisateur (l'identité du présentateur est simplement saisie dans le formulaire, et n'importe qui connaissant le code peut accéder au Dashboard).
- Les règles Supabase (RLS) sont ouvertes en lecture/écriture : suffisant pour un outil interne diffusé par lien, mais pas pour des données sensibles.
- Rafraîchissement par sondage toutes les 15 secondes plutôt que du temps réel instantané (réalisable via les canaux *Realtime* de Supabase si besoin, non activés ici).
- Affiches limitées à 8 Mo par fichier (PNG, JPG, PDF) côté Supabase, et à 700 Ko sur l'ancienne version Claude Artifact (stockage en base64 dans le document, sans bucket dédié). Même limite pour les photos de la rubrique Média (8 Mo côté Supabase, 700 Ko côté Artifact).
- Les vidéos YouTube n'ont pas de titre/description récupérés automatiquement (pas d'appel à l'API YouTube, pour éviter toute clé d'API à gérer) : l'administrateur les saisit lui-même, la miniature seule est calculée automatiquement depuis le lien.
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
