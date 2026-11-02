# Cartographie de cadrage : Leboncoin → heavy-shop

## 1. Service analysé et repo miroir

| Élément | Contenu |
|---|---|
| Service réel | Leboncoin, plateforme web et mobile d'achat et de vente d'objets d'occasion entre particuliers et professionnels (https://www.leboncoin.fr/) |
| Unité fonctionnelle (UF) | Acheter un objet d'occasion en ligne : rechercher, filtrer et consulter 3 annonces sur mobile, en une session de 5 min |
| Fréquence d'usage | Quelques minutes par session, plusieurs fois par semaine |
| Catégorie retenue | E-commerce / catalogue / réservation |
| Repo miroir | `heavy-shop` |
| Pourquoi pas `heavy-hub` | Le cœur de l'UF (recherche, filtres, listing, fiche, photos) correspond au catalogue de heavy-shop. La messagerie et les alertes relèvent de heavy-hub, mais sont hors UF |

## 2. Transposition leboncoin → heavy-shop

| Leboncoin | heavy-shop | Dans le périmètre |
|---|---|---|
| Annonce | Produit | Oui |
| Recherche et filtres | `/search` et `GET /api/search` | Oui |
| Liste de résultats | `/products` et `GET /api/products` | Oui |
| Fiche annonce, photos | `/products/:id` et assets SVG | Oui |
| Accueil | `/` et `GET /api/shop/home` | Oui |
| Promotions mises en avant | `GET /api/promotions` (polling) | Oui |
| Panier, checkout | `/cart`, `/checkout` | Non (absents de l'UF) |
| Messagerie, dépôt d'annonce | non représentés | Non |

**Limite assumée** : le code réel de Leboncoin n'est pas accessible. Les constats servent à choisir des pistes, les mesures et les gains sont obtenus sur le repo miroir uniquement.

## 3. Contraintes et objectifs

**Contraintes**
- Pas de main sur le code réel : tout se joue sur heavy-shop.
- Aucune fonctionnalité de l'UF ne doit disparaître (recherche, filtres, fiche).
- Mesures reproductibles : build de production, Brave mobile, médiane de 3 passages, tunnel public pour l'extension EcoIndex.
- Ressources limitées : effort exprimé en jours, plan sur 6 mois.

**Objectifs**
1. Réduire les transferts inutiles : cache HTTP, lazy loading, compression, payloads minimaux.
2. Réduire les requêtes : debounce de la recherche, fin du polling, API chargée par écran.
3. Mettre en place un suivi des KPI pour garder les gains dans la durée.

## 4. Lien ACV → bonnes pratiques → actions

| Constat ACV | Phase | Bonne pratique | Action sur heavy-shop | US |
|---|---|---|---|---|
| Photos et pages lourdes à chaque session | Utilisation, Transport | Cache HTTP, lazy loading | `Cache-Control` sur les assets, `loading="lazy"` | US1 |
| Filtres qui multiplient les requêtes | Utilisation | Limiter les requêtes, optimiser la recherche | Debounce, pagination de `/api/search` | US2 |
| Polling et API chargée partout | Utilisation, Transport | Réduire les transferts inutiles | Supprimer le polling, charger l'API par écran | US3 |
| Pages lourdes qui accélèrent le renouvellement des smartphones | Fabrication, Fin de vie | Sobriété des pages | Mêmes actions : moins de CPU et de batterie sur le terminal | US1 à US3 |
| Données et annonces obsolètes | Fin de vie | Réduire le stockage inutile | Purge des annonces expirées (roadmap, hors repo) | Roadmap |
| Hébergement | Fabrication, Utilisation | Serveurs éco-responsables | Recommandation de roadmap, non actionnable sur le repo | Roadmap |

## 5. Indicateurs retenus

| Indicateur | Pourquoi | Baseline (production) |
|---|---|---|
| Score EcoIndex et note | Lecture standard, comparable | 88 à 90, note A sur les 4 pages |
| Requêtes par page | Reflète le trafic et le polling | 12 à 14 |
| Poids transféré (Ko) | Impact réseau et terminal | 266 à 286 Ko sans compression |
| Poids en seconde visite | Mesure du cache | non mesuré (cache désactivé : même poids qu'en première visite) |
| Requêtes `/api/search` par session de filtrage | Mesure du debounce | 1 par caractère saisi |
| Requêtes périodiques à l'arrêt (par minute) | Mesure du polling | 12 (mesuré sur 62 s) |

Le score EcoIndex est déjà A : il sert de baseline mais ne suffit pas à piloter le plan. Les gains porteront surtout sur les autres indicateurs.

## 6. Sources

- Brief #1 : [acv-flash-leboncoin.xlsx](acv-flash-leboncoin.xlsx), dont les phases Transport à Fin de vie sont à valider.
- Bonnes pratiques prioritaires : fichier `mes-10-bp-prioritaires` à la racine du repo.
- Mesures : [baseline-ecoindex-heavy-shop.xlsx](../baseline-ecoindex-heavy-shop.xlsx).
- Backlog : [backlog.md](../backlog.md).
