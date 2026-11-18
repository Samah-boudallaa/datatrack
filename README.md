<h1 align="center">Data Track</h1>

<p align="center">
  Un laboratoire de poche pour manipuler les indicateurs de la mesure web :<br>
  formules, entonnoirs, événements et lecture de rapports.
</p>

<p align="center">
  <img alt="Licence MIT" src="https://img.shields.io/badge/licence-MIT-0a6f77">
  <img alt="Un seul fichier" src="https://img.shields.io/badge/fichier-index.html-0d1a22">
  <img alt="Sans build" src="https://img.shields.io/badge/build-aucun-1b7f4b">
  <img alt="Sans backend" src="https://img.shields.io/badge/backend-aucun-1b7f4b">
</p>

---

## En bref

Une application web **monofichier** : on ouvre `index.html`, tout fonctionne. Pas de serveur, pas de compte, pas d'étape de build. L'état est conservé dans le navigateur.

Elle réunit quatre outils qui partagent le même suivi de progression :

| Outil | Ce qu'il apporte |
|---|---|
| **Fiches** | Résumés illustrés (chaînes de collecte, entonnoirs, tableaux de cibles) et une page « Flash final » |
| **Questions** | 149 questions à choix multiple corrigées, rangées par thème |
| **Calculs générés** | 9 générateurs de cas chiffrés, nouveaux à chaque tirage |
| **Cartes** | 45 cartes recto/verso en répétition espacée |
| **Mode chrono** | Séries de 10, 20 ou 40 questions avec minuteur, marquage et correction détaillée |

## Le générateur de calculs

Les valeurs sont tirées au hasard, la bonne réponse est calculée, et **chaque mauvaise réponse correspond à une erreur réelle** : mauvais dénominateur, oubli du × 100, base inversée.

```text
Une annonce affiche 22 000 impressions et obtient 840 clics,
pour 596 sessions mesurées. Quel est le CTR ?

  A  3,8 %      ← clics ÷ impressions × 100
  B  140,9 %    ← impressions ÷ sessions
  C  2,7 %      ← sessions ÷ impressions (mauvaise base)
  D  0,04 %     ← oubli du × 100
```

Un tirage où deux options s'affichent à l'identique est rejeté et refait. Les neuf familles : CTR, CPC, conversion, abandon, rebond, coût par conversion, plus forte perte d'un entonnoir, perte entre deux étapes, nombre de requêtes d'une page.

## Répétition espacée

Chaque question et chaque carte vit dans une « boîte » (système de Leitner). Réussir fait monter d'une boîte, se tromper ramène en boîte 0.

| Boîte | 0 | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|---|
| Prochain rappel | tout de suite | +1 j | +2 j | +4 j | +8 j | +16 j |

Une question est dite **solide** à partir de la boîte 2, c'est-à-dire après deux réussites de suite. La « session du jour » sert d'abord les questions dues, puis des questions neuves.

## Architecture

```text
┌────────────────────────────── index.html ──────────────────────────────┐
│                                                                        │
│   Q[]  banque de questions ──┐                                         │
│   GENS[] générateurs ────────┼──► items ──► vues ──► innerHTML         │
│   FC[]  cartes ──────────────┘      ▲          │                       │
│                                     │          ▼                       │
│                       DB (progression) ◄── actions  (data-a="…")       │
│                          │                                             │
│                          ▼                                             │
│                    localStorage  ⇄  export / import JSON               │
└────────────────────────────────────────────────────────────────────────┘
```

- **Routage** par ancre (`#accueil`, `#fiches`, `#cartes`, `#entrainement`, `#examen`, `#erreurs`, `#reglages`).
- **Une seule délégation de clic** : chaque bouton porte un `data-a` qui désigne une fonction de l'objet `A`.
- **Thèmes** déduits du texte de chaque question par des règles simples (`TH_RULES`), ce qui donne le tableau de points faibles sans étiquetage manuel.
- **Thème clair/sombre** par variables CSS, avec le mode automatique du système par défaut.

## Données et vie privée

Rien ne quitte le navigateur. La progression est écrite dans le `localStorage` sous la clé `wa_prep_v2` (un champ par question, par carte et par séance chrono).

Elle ne se synchronise pas entre appareils. L'onglet **Réglages** exporte tout en JSON et le réimporte, avec une validation qui ne garde que les champs numériques attendus.

> La seule ressource externe est la police IBM Plex, chargée depuis Google Fonts. Sans réseau, la page retombe sur la police du système.

## Lancer

```bash
git clone https://github.com/Samah-boudallaa/datatrack
cd datatrack
# ouvrir index.html dans un navigateur, ou :
python3 -m http.server 8000
```

### Publier avec GitHub Pages

*Settings → Pages → Deploy from a branch → `main` / `(root)`*

## Raccourcis clavier

| Contexte | Touches |
|---|---|
| Questions | `A`–`D` ou `1`–`4` répondre · `Entrée` suivant |
| Mode chrono | `A`–`D` répondre · `←` `→` naviguer · `F` marquer |
| Cartes | `Espace` retourner · `1` à revoir · `2` je savais |

## Ajouter du contenu

Une question est une ligne, la bonne réponse toujours en premier (l'ordre est mélangé à l'affichage) :

```js
q(3, "Énoncé de la question ?",
  "Bonne réponse", "Piège 1", "Piège 2", "Piège 3",
  "Explication affichée après la réponse.");
```

- Le premier argument est le numéro de séance (1 à 4). Le thème est calculé automatiquement.
- Une carte s'ajoute dans `FC` : `[séance, "recto", "verso"]`.
- Un générateur s'ajoute dans `GENS` : une fonction qui retourne `mk(séance, énoncé, [bonne, p1, p2, p3], explication)`, ou `null` pour demander un nouveau tirage.

## Pistes

- [ ] Mode « plusieurs bonnes réponses »
- [ ] Questions à saisie numérique libre pour les calculs
- [ ] Export de la progression en fichier plutôt qu'en texte
- [ ] Statistiques de temps de réponse par question

## Licence

Distribué sous licence [MIT](LICENSE).
