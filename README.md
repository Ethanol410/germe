# 🌱 Germe

**L'app qui empêche tes idées de mourir.**

Tu notes tes idées, puis tu les oublies. Germe les capture à la voix, les cultive avec
toi grâce à un agent IA (le jardinier), et te ramène vers elles avant qu'elles ne
s'éteignent.

## Structure du dépôt

```
germe/
├── index.html        → la landing page (waitlist)
├── app/
│   └── index.html    → l'application complète (mobile + desktop)
└── docs/
    ├── architecture-saas-germe.md   → plan technique vers le SaaS
    └── brief-design-germe.md        → brief de l'identité visuelle
```

Tout est en **fichiers HTML autonomes** : aucun build, aucune dépendance, aucun
framework. Ouvre le fichier, ça marche.

## L'application (`app/index.html`)

Le cycle complet d'une idée :

- **Sème** — capture au clavier ou à la voix (dictée type mémo vocal, transcription
  corrigée par l'IA), wizard de structuration une question à la fois
- **Cultive** — le jardinier (agent IA) analyse l'idée, t'interviewe question par
  question, tient une mémoire du projet, produit synthèses et mises à jour du canevas
- **Récolte** — plan d'action cochable, score de croissance, prochaine action
  recommandée, célébration à la réalisation
- **Rien ne meurt oublié** — revue des idées dormantes (« elle a soif »), compost
  replantable, priorisation assistée du jardin, export Markdown

**Configuration requise :** une clé API Anthropic (console.anthropic.com), à coller
dans les options de l'app. Elle reste dans le navigateur, facturation à l'usage.
Les données vivent en localStorage — pense à exporter des sauvegardes.

## La landing (`index.html`)

Page waitlist autonome. Avant mise en ligne : créer un formulaire sur
[formspree.io](https://formspree.io) et remplacer `TON_ID_FORMSPREE` dans le fichier.

## Déploiement via GitHub Pages

Settings → Pages → Source : `main`, dossier `/ (root)` →
la landing est en ligne sur `https://<ton-user>.github.io/germe/`
et l'app sur `https://<ton-user>.github.io/germe/app/`.

## Roadmap

Voir `docs/architecture-saas-germe.md` : comptes et sync (Supabase), proxy API,
abonnement Stripe, PWA.

---

*Fait avec obsession pour les idées qui méritent de vivre.*
