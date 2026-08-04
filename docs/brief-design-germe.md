# Brief design — Germe 🌱

> Document de contexte complet à donner à un designer (humain ou IA) pour créer
> l'identité visuelle, les illustrations et les animations de Germe.

---

## 1. Le produit en une phrase

**Germe est l'app qui empêche tes idées de mourir** : tu sèmes une idée (à la voix ou
au clavier), un agent IA la cultive avec toi (il t'interviewe, la structure, en tire un
plan d'action), et l'app te ramène vers elle si tu la laisses dormir.

## 2. Le problème et la cible

Les gens créatifs notent leurs idées dans l'app Notes de leur téléphone… et ne les
relisent jamais. Aucun outil ne les aide à *développer* une idée : les notes stockent,
point. Germe transforme le stockage en culture.

**Cible :** créatifs solo, entrepreneurs, side-projecteurs, 20-40 ans, à l'aise avec
la tech, déjà utilisateurs de ChatGPT/Claude, sensibles au beau design (ils aiment
Linear, Notion, Arc). Marché francophone d'abord.

**Différenciation :** pas une app de notes de plus, pas un chatbot de plus — un
*cycle complet* : capture vocale → interview par l'agent → synthèse → plan d'action →
revue anti-oubli → priorisation entre idées.

## 3. La métaphore centrale : la germination

Le nom « Germe » porte tout l'univers. Chaque idée est une graine qui traverse un
cycle de vie, et TOUTE l'interface doit raconter cette croissance :

| Étape produit                    | Métaphore        | Statut dans l'app    |
|----------------------------------|------------------|----------------------|
| Idée capturée, brute             | La graine        | Nouvelle             |
| En discussion avec l'agent       | La pousse        | En développement     |
| Plan d'action en cours           | La plante        | En action            |
| Idée aboutie                     | La floraison / récolte | Réalisée       |
| Idée en pause                    | En dormance (bulbe sous terre) | En pause |
| Idée abandonnée                  | Compost (elle nourrit les suivantes) | — |
| Idée qui dort 7+ jours           | Elle a soif → l'app « l'arrose » (revue) | — |
| Score de progression (0-100 %)   | La hauteur de la tige qui pousse | —     |
| L'agent IA                       | Le jardinier     | —                    |

Vocabulaire produit : **Sème / Cultive / Récolte**. Le jardin = la liste des idées.

## 4. Direction visuelle souhaitée

**Base actuelle (à faire évoluer, pas à jeter) :** dark mode élégant type Linear —
fond quasi-noir, surfaces discrètes, bordures fines 1px, coins 10px, police Inter,
letterspacing serré, focus rings subtils, micro-animations 150-200ms.

**Évolution demandée :** garder cette rigueur mais y insuffler l'organique :
- **Palette :** fond sombre teinté de vert forêt (#0b0e0c), accent vert végétal
  (#3fae74 → #4fc286), touches lime (#a8e063) pour la croissance/succès, terre cuite
  (#f2994a) pour les alertes « à arroser », crème pour d'éventuelles variantes claires.
- **Formes :** introduire des courbes organiques (tiges, feuilles) en contrepoint de la
  grille rigoureuse. Les progress bars peuvent devenir des tiges qui poussent.
- **Iconographie :** style trait fin (type Lucide, stroke 2px) mais avec un set custom
  sur le thème végétal : graine, pousse, feuille, arrosoir, fleur, compost, soleil.

**Illustrations souhaitées (le gros manque actuel) :**
- Empty state du jardin : une parcelle de terre qui attend sa première graine
- Écran de fin de wizard : la graine plantée qui commence à germer
- Revue des idées dormantes : plante qui penche, gouttes d'arrosoir
- Idée réalisée : floraison, confettis de pétales
- Hero de la landing : une graine → pousse → plante stylisée, potentiellement animée
- Style suggéré : illustrations vectorielles minimalistes, 2-3 couleurs de la palette,
  trait fin, légèrement texturées — PAS de flat design générique corporate, PAS de 3D.

**Animations souhaitées :**
- La signature : une **tige qui pousse** (progress bar de progression de l'idée,
  transitions de score, splash screen)
- Graine qui tombe et se plante quand on capture une idée (feedback de capture)
- Feuilles qui frémissent subtilement au hover des cartes
- La pastille de statut qui « éclot » quand une idée change d'étape
- Micro-particules type spores/pollen dans le hero de la landing (discret, lent)
- Respecter `prefers-reduced-motion`

**Ton de voix (déjà en place, à conserver) :** tutoiement, direct, complice, un brin
poétique sur la métaphore mais jamais mièvre. Exemples réels : « Chaque idée est une
graine. Fais-la germer. » / « Abandonner en conscience, c'est aussi avancer. » /
« Rien ne meurt oublié ici. » Pas d'emojis dans l'UI (icônes SVG uniquement).

## 5. Inventaire des écrans à designer

### App (mobile-first, une seule page web)
1. **Le jardin (accueil)** — header sticky flouté, barre de capture (texte + micro + bouton),
   filtres en une rangée (statuts avec pastilles colorées, séparateur, catégories avec icônes),
   bannière « X idées ont soif » si idées dormantes, cartes d'idées (titre, pitch tronqué,
   statut, catégorie, nb messages, étapes cochées, date relative, prochaine action suggérée
   « → Synthétiser », barre de progression), FAB « + », état vide avec CTA.
2. **Fiche idée** — barre sticky (retour, mémoire, favori, supprimer), titre, panneau héro
   (score de progression + LA prochaine action recommandée en un bouton), 3 onglets :
   **Aperçu** (lecture : pitch en exergue, problème/solution/pour qui, étapes cochables,
   synthèse, notes, export .md), **Canevas** (édition des champs, auto-save),
   **Discussion** (chat avec l'agent, réponses rapides tapables, bouton synthèse).
3. **Wizard de création** — une question par écran (titre, catégorie, pitch, problème,
   solution, pour qui, première action), barre de progression, puis phase agent :
   analyse → questions une par une avec réponses rapides.
4. **Dictée vocale** — micro pulsant + chronomètre (rien ne s'affiche pendant),
   écran transcription en cours, relecture/correction, « En faire une idée ».
5. **Revue des idées dormantes** — une idée à la fois : reprendre / pause / plus tard /
   abandonner (avec confirmation).
6. **Panneaux latéraux** — Mémoire du projet (ce que l'agent a retenu : VISION, DÉCIDÉ,
   POINTS OUVERTS, ACTIONS), Priorisation (l'agent classe les idées et recommande).
7. **Divers** — modale de confirmation (bottom sheet mobile), toasts (en haut),
   menu options (clé API, export/import, test connexion), squelettes de chargement,
   indicateur « l'agent écrit » (3 points).

### Landing page
Nav sticky, hero + formulaire waitlist, mockup téléphone, section problème (3 stats),
4 features, 3 étapes (Sème/Cultive/Récolte), pricing (0 € / 7 €/mois), FAQ (4 questions),
CTA final, footer.

## 6. Design system actuel (tokens de départ)

```css
--bg: #0b0e0c;        /* fond, teinté vert */
--surface: #131714;   /* cartes */
--surface2: #1b201c;  /* éléments imbriqués */
--border: #242b26;    /* bordures 1px */
--text: #f5f8f5;
--muted: #8b968d;
--accent: #3fae74;    /* vert végétal, actions primaires */
--accent-h: #4fc286;  /* hover */
--lime: #a8e063;      /* croissance, succès, gradients */
--orange: #f2994a;    /* alerte « a soif » */
--red: #eb5757;       /* danger */
--radius: 10px;
/* Police : Inter (400-800), letter-spacing -0.011em, titres -0.03em */
```

Statuts et couleurs : Nouvelle (gris), En développement (jaune #f2c94c),
En action (vert accent), En pause (orange), Réalisée (lime).
Catégories : Business, Créatif, Perso, Tech, Autre (icônes trait fin).

## 7. Contraintes techniques

- L'app est un **fichier HTML unique** (CSS + JS inline), sans framework ni build.
  Les propositions doivent être réalisables en CSS/SVG/JS vanilla.
- **Mobile-first absolu** : cibles tactiles ≥ 44px, inputs ≥ 16px (anti-zoom iOS),
  safe areas iPhone, bottom sheets, clavier géré.
- Animations en CSS (transform/opacity) — pas de librairie lourde (Lottie acceptable
  en SVG/CSS export si léger). Illustrations en **SVG inline** de préférence.
- Dark mode par défaut (une variante claire est un bonus, pas une exigence).
- Performance : la page doit rester < 500 Ko tout compris.

## 8. Livrables attendus

1. **Identité** : logo (la pousse), déclinaisons, favicon/icône PWA
2. **Set d'icônes custom** thème végétal (12-15 icônes, stroke cohérent)
3. **5-6 illustrations SVG** pour les états clés listés en section 4
4. **Les animations signatures** (tige qui pousse, graine plantée, éclosion de statut)
5. **La landing page redessinée** avec ces assets (fichier HTML autonome)
6. **Maquette des 3 écrans clés de l'app** : jardin, fiche (Aperçu), wizard agent
7. Tokens finaux du design system

## 9. Références d'inspiration

Rigueur d'interface : Linear, Arc, Notion. Chaleur organique : Headspace (illustrations),
Gentler Streak (métaphore vivante), Plant Nanny (concept d'arrosage, en moins enfantin).
L'équilibre cible : **« Linear qui aurait poussé dans une serre »** — la précision d'un
outil pro, la vie d'un jardin.
