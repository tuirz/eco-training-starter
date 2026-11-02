# Backlog heavy-shop

## Contexte du projet

Le projet représente une boutique catalogue avec listes, recherche, détail produit, panier et checkout simplifié.

Service réel analysé (Brief #1) : Leboncoin. UF : acheter un objet d'occasion en ligne, soit rechercher, filtrer et consulter 3 annonces sur mobile en une session de 5 min.
Correspondance avec le repo miroir : annonce = produit ; recherche, filtres, listing et fiche = écrans heavy-shop. Panier et checkout hors périmètre.

Baseline production (build, Brave mobile, médiane de 3 mesures, 2 nov. 2026), détail dans baseline-ecoindex-heavy-shop.xlsx :

| Page | Requêtes | Poids | DOM | EcoIndex |
|---|---|---|---|---|
| Accueil | 14 | 286 Ko | 107 | 88,5 (A) |
| Listing | 12 | 266 Ko | 123 | 88,4 (A) |
| Fiche | 13 | 268 Ko | 130 | 88,0 (A) |
| Recherche | 13 | 282 Ko | 127 | 88,1 (A) |

Le score EcoIndex est déjà A : il sert de baseline mais ne suffit pas à piloter le plan. Les KPI du backlog portent sur les requêtes, les octets (API, JS), le polling et le cache à la seconde visite.
Constats : JS de 172 Ko sans compression ; API chargée sur toutes les pages (`/api/shop/home` 14,1 Ko, `/api/products` 16,7 Ko, `/api/cart` 3,8 Ko, `/api/checkout` 10,4 Ko) ; 6 SVG de 10,3 Ko chargés dès l'accueil ; `Cache-Control: no-store` partout ; `/api/promotions` appelé toutes les 5 s.

## User story 1

- Contexte : En tant qu'acheteur sur mobile, je veux que les listes d'annonces et leurs visuels se chargent sans retélécharger ce que j'ai déjà vu, afin de consulter mes 3 annonces avec moins de données et de batterie.
- Objectif : passer de 286 Ko à moins de 30 Ko transférés en seconde visite de l'accueil (assets et JS servis depuis le cache), et de 6 SVG chargés à l'accueil à 2 au plus au premier rendu.
- Bonne pratique d'éco-conception ciblée : utiliser le cache HTTP et le lazy loading (RGESN, phase Utilisation).
- KPI associé : poids transféré (Ko) et nombre de requêtes sur le parcours accueil, puis listing, puis fiche, en visite 1 vs visite 2.
- Repo ou écran concerné : backend/src/index.ts (en-têtes `Cache-Control`, `express.static` des assets) et ShopApp.tsx (listing, images `loading="lazy"`).
- Critère de réussite : en visite 2, 0 requête réseau pour le JS, le CSS et les images (cache), et images hors écran non chargées au premier rendu.
- Niveau de priorité : haute

## User story 2

- Contexte : En tant qu'acheteur, je veux filtrer et rechercher des annonces sans déclencher un appel serveur à chaque touche, afin d'obtenir mes résultats avec moins de requêtes et de calculs serveur.
- Objectif : passer de 1 requête par caractère saisi à 1 requête au plus par saisie validée (debounce), et de 16,8 Ko à moins de 4 Ko pour la réponse de `/api/search` (pagination).
- Bonne pratique d'éco-conception ciblée : limiter les requêtes HTTP et optimiser les requêtes de recherche (pagination, champs utiles uniquement).
- KPI associé : nombre de requêtes `/api/search` par session de filtrage et taille de la réponse (Ko).
- Repo ou écran concerné : ShopApp.tsx (SearchPage) et GET /api/search.
- Critère de réussite : saisir 10 caractères puis changer de catégorie génère 2 requêtes au plus ; réponse paginée, sans `filtersEcho`.
- Niveau de priorité : haute

## User story 3

- Contexte : En tant qu'acheteur sur mobile, je veux que la page d'accueil et la fiche produit n'envoient que les données utiles, afin de limiter le trafic réseau et la consommation de mon terminal.
- Objectif : supprimer le polling `/api/promotions` (12 requêtes/min à l'arrêt sur l'accueil → 0), ne plus charger `/api/cart` et `/api/checkout` hors panier et checkout (-14 Ko par page), et passer `/api/shop/home` de 14,1 Ko à moins de 6 Ko.
- Bonne pratique d'éco-conception ciblée : réduire les transferts inutiles (suppression du polling, payload minimal).
- KPI associé : nombre de requêtes par minute à l'arrêt sur l'accueil et taille de `/api/shop/home` (Ko).
- Repo ou écran concerné : HomePage (`setInterval` vers `/api/promotions`) et GET /api/shop/home.
- Critère de réussite : 0 requête périodique à l'arrêt sur l'accueil ; aucun appel `/api/cart` ni `/api/checkout` sur accueil, listing, fiche et recherche ; `/api/shop/home` sous 6 Ko ; moins de 10 requêtes par page.
- Niveau de priorité : moyenne

## Notes

- Bonnes pratiques retenues : fusion des doublons (images, requêtes/lazy loading). Hors backlog : serveurs éco-responsables, non actionnable sur le repo (à garder comme recommandation de roadmap).
- Mesure EcoIndex calculée avec la formule officielle (cnumr/GreenIT-Analysis) sur les valeurs Lighthouse ; capture de l'extension EcoIndex à ajouter pour comparaison.
- Polling mesuré le 2 nov. 2026 : 12 requêtes `/api/promotions` en 62 s sur l'accueil (build de production, Brave headless, journal du backend), soit 1 toutes les 5 s.
- Le niveau de détail attendu doit permettre une priorisation exploitable.
