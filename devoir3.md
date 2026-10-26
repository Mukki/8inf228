# Devoir 3

### 8INF228 — Adaptation et qualité des applications
### Système de fidélité pour un café · version 3

| | |
|---|---|
| **Pondération** | 7 % de la note finale |
| **Équipes** | Les mêmes qu'aux devoirs 1 et 2 — c'est le même dépôt |
| **Remise** | Dépôt GitHub, *tag* `v3.x.y` |
| **Échéance** | 16 novembre |
| **IA générative** | Autorisée, à déclarer dans `AI-USAGE.md` |

---

## 1. Mise en situation

Les affaires vont bien. Très bien.

> « Je signe le bail la semaine prochaine : j'ouvre un deuxième café. Ça ouvre le mois prochain. Les clients vont pouvoir utiliser leur carte dans les deux places. Pis j'engage une gérante pour le nouveau café, mais elle a pas d'affaire à jouer dans les chiffres de l'autre. »

Pour la première fois, vous avez **du temps devant vous**, donc vous aller essayer de vous rendre la tâche plus facile. Dans le prochain devoir, nous déploierons vos devoir sur le cloud. C'est le moment de faire les choses comme il faut :

- un logiciel qui comprend qu'il y a **plusieurs cafés** ;
- une vraie base de données, **PostgreSQL** ;
- une application qui tourne dans des **conteneurs prêts pour la production**, derrière un proxy inverse, avec l'aide de Docker ou de podman.

Votre v2 est en production au premier café. Comme au devoir 2, **vous n'avez pas le droit de perdre ses données**.

Dites vous que l'application quittera la machine du comptoir au prochain devoir : elle sera déployée sur **un serveur sur le Web**, et **les deux cafés s'y connecteront**. Ce déploiement ne fait pas partie de ce devoir, mais tout ce que vous construisez ici doit le préparer. Si vous faites votre travail correctement, seules des **variables d'environnement** devraient changer dans votre code.

---

## 2. Requis fonctionnels — le deuxième café

Jusqu'ici, votre application supposait qu'il y avait **un seul café**, et qu'elle tournait **sur la machine du comptoir**.

*Quelques questions pour commencer :*

- *Où, dans votre code, l'hypothèse « un seul café » est-elle cachée? Dans le modèle, dans les rapports, dans les permissions, dans la configuration?*
- *Qu'est-ce qui, dans votre projet, ne fonctionne que parce qu'il tourne sur votre propre machine? Un chemin de fichier, une clé secrète, `DEBUG = True`, `runserver`?*

Tout ce qui fonctionnait en v2 doit continuer de fonctionner.

Comme aux devoirs précédents, chaque demande contient au moins une **zone grise**, en italique. Elles n'ont pas de réponse dans cet énoncé : vous tranchez, et vous justifiez dans un ADR.

### 2.1 Plusieurs cafés

> « J'ouvre un deuxième café, mais on sait pas encore comment on vas le nommer. »

- Le système connaît la notion de **succursale**. On peut en ajouter une nouvelle **sans modifier le code**. Peut-être y auras-t-il une troisième succursale bientôt?
- **Chaque transaction appartient à une succursale** : achat, café gratuit, café tiré d'une carte, vente de carte, annulation.
- Toutes les données de la v2 appartiennent au premier café.

*Les deux cafés se connectent au même serveur : comment l'application sait-elle dans quel café une transaction a lieu ? Par la personne connectée, par un choix à l'écran, autrement ? Et une barista qui fait des quarts dans les deux cafés ?*

### 2.2 Une seule fidélité

> « Une étampe, c'est une étampe. Peu importe le café. »

- Les **étampes**, les **cafés gratuits en réserve** et les **cartes prépayées** d'un client sont valables **dans tous les cafés**.
- Un client peut accumuler des étampes dans le café A et réclamer son café gratuit dans le café B*.

*La règle du café gratuit (devoir 2) reste-t-elle la même pour tout le réseau, ou chaque café peut-il avoir la sienne? Et si le nouveau café veut une promotion d'ouverture (deux étampes pour un achat le jour d'ouverture) que le premier café n'a pas?*

### 2.3 Chacun son café

> « La nouvelle gérante, elle gère juste le nouveau café, je veux pas qu'elle puisse modifier des transactions qui ne lui appartiennent pas. »

- Le système distingue désormais au moins trois rôles :
  - **barista** : comme en v2 ;
  - **gérant** : comme en v2, **mais seulement pour sa succursale** — il ne voit pas les rapports de l'autre café et ne peut pas y annuler une transaction ;
  - **propriétaire** : tout, dans toutes les succursales.
- Les restrictions sont vérifiées **côté serveur**, pas seulement dans l'affichage.

*Le gérant de du café A peut-il effacer (devoir 2) ou fusionner (devoir 2) un client qui a aussi des transactions au café B? La fiche client appartient-elle à un café, ou à tout le réseau?*

### 2.4 Les rapports du réseau

> « Je veux voir mes rapports en pouvant filter par café ou les deux ensemble. »

- Les rapports du devoir 2 existent **par succursale** et **pour l'ensemble du réseau**.
- Un gérant ne voit que les rapports de sa succursale ; le propriétaire voit tout.

*Le café gratuit réclamé grâce à des étampes gagnées à un autre café: dans le rapport de quel café apparaît-il?*

### 2.5 Hors périmètre

> **Toujours aucun paiement. Toujours pas d'accès client.**

- Aucun déploiement pour l'instant : **tout tourne en local**, sur une seule machine.
- **Une seule instance** de l'application. Pas de répartition de charge, pas de réplication.
- Pas de Kubernetes, pas de Terraform, pas de nom de domaine, pas de certificat HTTPS.

---

## 3. Des conteneurs prêts pour la production

**Une seule commande** démarre tout le système. Je corrigerai avec :

```bash
cp .env.example .env
docker compose up --build
```

L'application doit alors être accessible sur `http://localhost:8000`.

Vous pouvez également me demander de réaliser certaines opérations, comme générer des clés pour mettre dans mon `.env` avant de faire `docker compose up`, mais n'oublier pas de les documenter!

**Podman est accepté.** Je corrige avec **Docker Compose** : restez dans la spécification Compose standard, évitez les options propres à un seul moteur, et testez avec `docker compose` si vous le pouvez.

### 3.1 Exigences

| # | Exigence |
|---|---|
| **C1** | **PostgreSQL** dans son propre conteneur, avec un **volume nommé** : les données survivent à `docker compose down` puis `up`. |
| **C2** | Un **proxy inverse** (nginx, Traefik, Caddy…) devant l'application. **Seul le proxy est exposé** sur `localhost:8000` ; l'application et la base ne sont accessibles que par le réseau interne des conteneurs. |
| **C3** | Un **serveur d'application de production** (Gunicorn, Uvicorn…) — jamais `runserver`. |
| **C4** | **`DEBUG = False`**. Une page inexistante affiche une erreur 404 normale, pas la page de débogage de Django. |
| **C5** | **Toute la configuration vient de variables d'environnement** : `SECRET_KEY`, connexion à la base, hôtes permis, origines de confiance, etc. Un fichier `.env.example` documente chaque variable. **Aucun secret dans le dépôt ni dans l'image.** Passer de `localhost` à un vrai nom de domaine ne doit demander **aucune modification du code**. |
| **C6** | Une **image construite par un `Dockerfile`** à vous : image de base précise (pas `latest`), **utilisateur non-root**, `.dockerignore`, rien d'inutile dans l'image finale. |
| **C7** | Les **fichiers statiques** (feuilles de style de l'administration, entre autres) s'affichent correctement. |
| **C8** | Des **vérifications de santé** (*healthchecks*) : l'application ne démarre pas tant que la base n'est pas prête, et Docker sait si chaque conteneur est en bonne santé. |
| **C9** | Les **journaux** vont sur la sortie standard : `docker compose logs` montre ce qui se passe. |
| **C10** | Les **migrations** s'appliquent de façon fiable, au démarrage ou par une commande documentée. |
| **C11** | Les **tests** s'exécutent **contre PostgreSQL**, en une commande documentée. |

*Les migrations doivent-elles s'appliquer toutes seules au démarrage, ou par une commande explicite? Où servir les fichiers statiques : par Django, par le proxy, par un conteneur à part?*

### 3.2 Le mode développement

Vous avez le droit de garder une façon de travailler plus confortable pour développer (rechargement automatique, `DEBUG = True`). Mais elle doit être **clairement séparée** de la configuration de production, et ce n'est pas elle que je corrige.

---

## 4. Migrer la v2 vers la v3

Votre v2 tourne sur SQLite au premier café, mais je veux que vous utilisier PostgreSQL pour la v3. Voici ce que je ferai à la correction :

1. Je récupère votre dernière tag `v2.x.y`, je crée la base SQLite et je charge **vos données de démonstration de la v2**.
2. Je note les soldes de quelques clients avec votre commande `python manage.py balances`.
3. Je passe à votre dernière tag `v3.x.y` et je suis **à la lettre** votre procédure de migration (`docs/migration.md`) pour transférer cette base SQLite dans le PostgreSQL de vos conteneurs.
4. Je relance `balances` dans les conteneurs. **Chaque solde doit être identique**, et toutes les transactions existantes doivent appartenir à *Rivière*.

| # | Exigence |
|---|---|
| **C1** | Les migrations Django de la v3 s'appliquent **par-dessus** le schéma de la v2. ⚠️ Toujours **interdit** de supprimer ou de réécrire une migration déjà livrée.
| **C2** | Le transfert de SQLite vers PostgreSQL est **scripté ou documenté pas à pas**.
| **C3** | La commande `balances` existe toujours, fonctionne dans les conteneurs, et peut afficher les soldes **par succursale** ou pour tout le réseau.
| **C4** | `docs/migration.md` montre **votre propre vérification** : les soldes avant et après, pour vos données de démonstration de la v2.

Noter bien que *SQLite est beaucoup plus permissif que PostgreSQL. Si la migration échoue sur des données que SQLite acceptait sans rien dire, c'est une excellente matière pour votre rétrospective.*

---

## 5. Livraison

Tout est dans le même dépôt GitHub. Vous ne devez pas utilisez un autre dépôt. Veuillez utiliser le tag v3.x.y

### 5.1 La rétrospective de la v2 — `docs/retrospective-v2.md`

Une demi-page à une page, à écrire **avant** de coder la v3 :

- Qu'est-ce que vous n'aviez **pas** vu venir ?
- Votre modèle permettait-il d'ajouter une succursale à un seul endroit, ou l'hypothèse « un seul café » était-elle répandue partout ?

Notée sur la **lucidité**, jamais sur la qualité de la v2.

### 5.2 Les ADR — `docs/adr/`

- **Un ADR par zone grise**.
- Si la v3 change une décision d'un ADR précédent, **écrivez un nouvel ADR** et changez le statut de l'ancien pour « Remplacé par ADR-00X ». Comme au devoir 2 : un ADR ne se réécrit pas.

### 5.3 Le document de conception v3 — `docs/conception.md`

Mettez à jour votre document de la v2 :

- le **modèle de domaine** v3, avec la succursale ;
- un **diagramme de déploiement** : les conteneurs, le réseau, les ports exposés, les volumes, le chemin d'une requête du navigateur jusqu'à la base ;
- **où sont vérifiées les permissions par succursale** ;
- des liens vers vos ADR ;

### 5.4 Structure attendue

```
root/
├── README.md                ← mis à jour : démarrage, comptes de démo, tests
├── AI-USAGE.md              ← complété avec une section « Devoir 3 »
├── .env.example             ← nouveau : chaque variable documentée
├── compose.yaml             ← nouveau (ou docker-compose.yml)
├── Dockerfile               ← nouveau
├── docs/
│   ├── conception.md        ← mis à jour (v3), avec le diagramme de déploiement
│   ├── retrospective-v2.md  ← nouveau
│   ├── migration.md         ← mis à jour : v2 → v3
│   └── adr/                 ← complété
└── src/                     ← le code du projet Django
```

L'emplacement exact des fichiers Docker est libre, tant que le README le dit.

### 5.5 Deux chemins, comme au devoir 2

Les deux doivent fonctionner en suivant votre README **à la lettre** :

1. **Une installation neuve de la v3**, avec ses propres données de démonstration.
2. **Une migration d'une base v2** existante.

Les données de démonstration de la v3 contiennent au moins : **deux succursales**, un compte **barista**, un compte **gérant par succursale** et un compte **propriétaire** (identifiants dans le README), un client qui a des transactions **dans les deux cafés**, et des transactions réparties sur plusieurs jours.

⚠️ Les mots de passe de démonstration sont acceptables dans le README pour ce devoir. Les **secrets de l'application** (`SECRET_KEY`, mot de passe de la base), eux, n'ont rien à faire dans le dépôt.
