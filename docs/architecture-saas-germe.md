# Architecture SaaS — Germe

De ton fichier HTML actuel à un vrai SaaS avec MRR. Document de référence complet.

---

## 1. Vue d'ensemble

```
┌─────────────┐         ┌──────────────────────────────────────┐
│  Frontend    │  HTTPS  │              SUPABASE                │
│  (Vercel)    │────────▶│                                      │
│              │         │  Auth ── comptes email/Google/Apple  │
│  Ton app     │         │  Postgres ── idées, chats, quotas    │
│  actuelle,   │         │  Edge Function "agent" ──────────────┼──▶ API Anthropic
│  quasi telle │         │   (proxy : TA clé, jamais exposée)   │    (TA clé serveur)
│  quelle      │         │  Edge Function "stripe-webhook" ◀────┼─── Stripe
└─────────────┘         └──────────────────────────────────────┘
```

**Principe clé :** ton frontend existant est réutilisable à ~80 %. Les deux changements
structurels : `localStorage` → base Supabase (ce qui donne la sync gratuite entre
appareils), et `fetch(api.anthropic.com)` → `fetch(ton-edge-function)` (ce qui protège
ta clé et permet les quotas).

---

## 2. Stack retenue (et pourquoi)

| Brique         | Choix                  | Pourquoi                                                    | Coût au début |
|----------------|------------------------|-------------------------------------------------------------|---------------|
| Hébergement    | Vercel ou Netlify      | Deploy en git push, HTTPS, CDN                              | 0 €           |
| Backend        | Supabase               | Auth + Postgres + Edge Functions en un seul service, dashboard simple | 0 € (free tier: 50k users, 500 Mo) |
| Paiement       | Stripe                 | Standard absolu, Checkout tout fait, portail client inclus  | 1,5 % + 0,25 €/transaction (cartes EU) |
| IA             | API Anthropic          | Tu l'utilises déjà ; Sonnet pour l'agent, Haiku pour les tâches simples | ~0,50–2 €/mois par utilisateur actif |
| Transcription  | Web Speech API (v1)    | Gratuit ; passer à Whisper API plus tard si besoin           | 0 €           |
| Landing/emails | Formspree puis Resend  | Waitlist d'abord, emails transactionnels ensuite             | 0 € puis ~0 € |

Pas de framework obligatoire : ton HTML/JS actuel + le SDK `@supabase/supabase-js`
en `<script>` suffit pour la v1. Migration vers Next.js possible plus tard, jamais urgente.

---

## 3. Base de données (Postgres / Supabase)

```sql
-- Profil (complète auth.users géré par Supabase)
create table profiles (
  id uuid primary key references auth.users(id) on delete cascade,
  plan text not null default 'free',            -- 'free' | 'pro'
  stripe_customer_id text,
  stripe_subscription_id text,
  created_at timestamptz default now()
);

-- Idées (reprend exactement ta structure JSON actuelle)
create table ideas (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references profiles(id) on delete cascade,
  title text not null,
  cat text default 'Autre',
  status text default 'Nouvelle',
  fav boolean default false,
  pitch text default '', problem text default '',
  solution text default '', who text default '',
  notes text default '',
  steps jsonb default '[]',                      -- [{text, done}]
  chat jsonb default '[]',                       -- [{role, content, hidden}]
  synthesis text, synth_n int default 0, applied_n int default 0,
  memory jsonb,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);
create index on ideas (user_id, updated_at desc);

-- Compteur d'usage IA mensuel (pour les quotas du plan gratuit)
create table usage (
  user_id uuid references profiles(id) on delete cascade,
  month text,                                    -- '2026-07'
  agent_messages int default 0,
  primary key (user_id, month)
);
```

**Row Level Security (indispensable) :**

```sql
alter table ideas enable row level security;
create policy "own ideas" on ideas
  for all using (auth.uid() = user_id) with check (auth.uid() = user_id);
-- idem sur profiles (select/update own) et usage (select own)
```

Avec RLS, le frontend peut lire/écrire directement dans Postgres via le SDK Supabase :
pas d'API à écrire pour le CRUD des idées. `localStorage` est remplacé par ~30 lignes.

---

## 4. Edge Function « agent » — le proxy IA

C'est la pièce maîtresse : ta clé Anthropic n'existe QUE côté serveur.

```
POST /functions/v1/agent
Authorization: Bearer <jwt supabase de l'utilisateur>
Body: { system, messages, max_tokens }
```

Logique (TypeScript/Deno, ~60 lignes) :

1. Vérifier le JWT → récupérer `user_id` (fait automatiquement par Supabase).
2. Charger `profiles.plan` + `usage` du mois courant.
3. **Quota** : si plan `free` et `agent_messages >= 10` → `402 { error: "quota" }`
   → le frontend affiche l'écran d'upgrade.
4. Appeler `api.anthropic.com` avec `ANTHROPIC_API_KEY` (secret d'environnement,
   jamais dans le code).
5. Incrémenter `usage.agent_messages`, renvoyer la réponse.

Garde-fous supplémentaires : `max_tokens` plafonné serveur (ex. 1200), longueur du
`system` et des `messages` plafonnée, rate-limit 20 req/min/user. Ça borne ton coût
maximum par utilisateur — personne ne peut te ruiner.

Dans ton frontend, la fonction `apiCall()` change de 3 lignes : l'URL et le header.
Tout le reste (agent, synthèse, priorisation, vocal) marche sans modification.

---

## 5. Stripe — l'abonnement

**Produits :** un seul produit « Germe Pro », prix récurrent 7 €/mois
(+ option annuelle 70 €/an plus tard, ça réduit le churn).

**Flux d'upgrade :**
1. L'utilisateur clique « Passer Pro » → Edge Function `create-checkout` crée une
   Stripe Checkout Session (avec `client_reference_id = user_id`) → redirection.
2. Paiement sur la page hébergée Stripe (tu ne touches jamais une carte bancaire).
3. **Webhook** `stripe-webhook` (Edge Function) écoute :
   - `checkout.session.completed` → `profiles.plan = 'pro'` + stocke les IDs Stripe
   - `customer.subscription.deleted` / `invoice.payment_failed` → `plan = 'free'`
4. « Gérer mon abonnement » → lien vers le **Stripe Customer Portal** (résiliation,
   changement de carte : zéro code, zéro SAV).

Le plan est TOUJOURS lu depuis `profiles` côté serveur (jamais confiance au client).

---

## 6. Migration depuis l'app actuelle

| Actuel                          | Cible                                             |
|---------------------------------|---------------------------------------------------|
| `localStorage` (load/persist)   | SDK Supabase : `select/upsert` sur `ideas`        |
| Clé API saisie par l'utilisateur| Supprimée — proxy avec TA clé                     |
| `apiCall()` → Anthropic direct  | `apiCall()` → Edge Function `agent`               |
| Export/Import JSON              | Conservé (portabilité = argument de vente) + **écran d'import au premier login** pour récupérer les idées du fichier actuel |
| Fichier HTML transféré à la main| `git push` → Vercel déploie ; PWA (manifest + service worker) pour l'icône et l'offline |

Stratégie : garde une couche `store.js` avec les mêmes fonctions (`loadIdeas`,
`saveIdea`…) — le reste de ton code ne voit pas la différence. Cache local
(localStorage) conservé pour l'affichage instantané, sync en arrière-plan.

---

## 7. Coûts et point mort

**Coûts fixes :** domaine ~12 €/an. Supabase, Vercel : 0 € jusqu'à ~des milliers
d'utilisateurs. Total : **~1 €/mois**.

**Coût variable :** l'IA. Par utilisateur Pro actif : 30-80 conversations/mois
× ~2-4k tokens ≈ **0,50 à 2 €/mois** (Sonnet). Marge par abonné : **5-6,5 €/mois**.
Utilisateur free : plafonné à ~0,15 €/mois par le quota.

**Projection prudente** (conversion free→pro 4 %, churn 6 %/mois) :

| Visiteurs/mois | Inscrits | Abonnés (cumul stabilisé) | MRR      |
|----------------|----------|---------------------------|----------|
| 1 000          | ~150     | ~10                       | ~70 €    |
| 5 000          | ~750     | ~55                       | ~385 €   |
| 15 000         | ~2 250   | ~170                      | ~1 190 € |

La colonne de gauche est le vrai combat : contenu (X/TikTok « je développe mes idées
avec mon agent IA »), Product Hunt, SEO (« que faire de ses idées », « app idées IA »).

---

## 8. Roadmap réaliste (soirées + week-ends)

**Semaine 0 — Validation (AVANT tout code)**
Landing en ligne (domaine + Vercel + Formspree) → poster partout → objectif 100+ emails.
❌ Si < 30 emails après un vrai effort de partage : pivote le message ou la cible, pas le code.

**Semaines 1-2 — Fondations**
Projet Supabase : tables + RLS + Auth (email magic link + Google).
Frontend : écran login, couche `store.js`, import du JSON existant.

**Semaines 3-4 — Proxy & quotas**
Edge Function `agent` + suppression de la clé côté client + compteur d'usage
+ écran « quota atteint → passe Pro ».

**Semaine 5 — Stripe**
Checkout + webhook + portail client. Tester en mode test de bout en bout.

**Semaine 6 — PWA & polish**
Manifest, service worker, icône, onboarding 3 écrans, page compte.

**Semaine 7 — Lancement early access**
Email à la waitlist (offre fondateur : -50 % à vie pour les 50 premiers).
Puis : parler aux utilisateurs chaque semaine, corriger, itérer. Product Hunt quand stable.

---

## 9. Sécurité — la checklist non négociable

- [ ] Clé Anthropic : variable d'environnement serveur uniquement, jamais dans le repo
- [ ] RLS activé sur TOUTES les tables (tester : un user A ne lit jamais les idées de B)
- [ ] Quotas + rate-limit + `max_tokens` plafonné dans l'Edge Function
- [ ] Signature des webhooks Stripe vérifiée (`stripe.webhooks.constructEvent`)
- [ ] Plan lu côté serveur, jamais depuis le client
- [ ] Export de données en un clic + suppression de compte (RGPD)
- [ ] Mention légale + politique de confidentialité (générables, mais obligatoires)

---

## 10. Ce que tu as déjà vs ce qu'il reste

**Déjà fait (l'app actuelle)** : tout le produit — UX mobile, agent, wizard, synthèse,
revue, priorisation, vocal, export. C'est la partie que la plupart des gens n'atteignent jamais.

**Reste à faire** : comptes, sync, proxy, paiement, distribution. Environ 6 semaines
de soirées. Aucune de ces briques n'est difficile — elles sont juste nouvelles.

La seule étape risquée est la semaine 0. Tout le reste est de l'exécution.
