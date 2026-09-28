# Devoir 2

### 8INF228 — Adaptation et qualité des applications
### Système de fidélité pour un café · version 2

| | |
|---|---|
| **Pondération** | 8 % de la note finale |
| **Équipes** | Les mêmes qu'au devoir 1, même dépôt |
| **Remise** | Dépôt GitHub, *tag* `v2.x.y` |
| **Échéance** | 19 octobre |
| **IA générative** | Autorisée, à déclarer dans `AI-USAGE.md` |

---

## 1. Mise en situation

Votre logiciel tourne au comptoir depuis quelques semaines. Le proprio est content. Tellement content qu'il revient vous voir avec une liste.

> « C'est pas mal pantoute, ton affaire. Mais là, j'ai des demandes. Hier, j'ai étampé le mauvais client, comment j'annule ça? Pis mon comptable veut savoir combien de cafés on a servis mardi passé. Ah, pis j'ai engagé du monde: je veux pas que n'importe qui puisse tout faire dans le système. »

Le logiciel tourne toujours localement, sur la machine du comptoir. Rien ne change de ce côté.

Ce qui change, c'est que **votre v1 est maintenant en production.** Il y a de vrais clients dedans, avec de vraies étampes et des cartes prépayées déjà payées. **Vous n'avez pas le droit de perdre ces données**.

---

## 2. La question centrale: un compteur ou un historique?

Que vous l'ayez fait exprès ou non, votre v1 a fait un choix:

- **Garder l'état actuel** - le nombre d'étampes d'un client, ses récompenses en réserve, les cafés qui restent sur sa carte. On met ces nombres à jour à chaque achat.
- **Garder l'historique** - chaque transaction est enregistrée, et l'état d'un client se *calcule* à partir de toutes ses transactions.

Les six demandes du patron (que vous verrez dans le point suivant) mettent ce choix à l'épreuve, **dans les deux sens**. Certaines sont beaucoup plus faciles avec un historique. D'autres deviennent plus difficiles avec un historique. Il n'y a pas de bonne réponse.

Vous devez **réexaminer votre choix de la v1** - le confirmer ou le remplacer - et le défendre dans un _Architecture Decision Record (ADR)_.

*Quelques questions pour vous aider :*

- *Si vous ne gardez que l'état actuel, comment annulez-vous une transaction d'hier? Comment répondez-vous à « combien de cafés mardi passé »?*
- *Si vous ne gardez que l'historique, que coûte l'affichage du solde d'un client qui vient tous les jours depuis trois ans et qui a des centaines de transactions? Que devient cet historique quand un client demande l'effacement de ses renseignements?*
- *Si vous gardez les deux, lequel fait foi le jour où ils ne s'accordent plus?*

---

## 3. Nouveaux requis fonctionnels pour la v2

Tout ce qui fonctionnait en v1 doit continuer de fonctionner.

Chaque demande contient au moins une **zone grise**, en italique. Comme au devoir 1, elles n'ont pas de réponse dans cet énoncé, et n'en auront pas si vous me les posez: vous tranchez, et vous justifiez votre choix dans l'ADR de la demande.

### 3.1 Annuler une transaction

> « Hier, j'ai étampé le mauvais client. Faut que j'annule ça. »

- Un gérant peut annuler une transaction passée : un achat, un café gratuit, un café tiré d'une carte prépayée, ou la vente d'une carte.
- Après l'annulation, les soldes du client sont corrects.
- On doit pouvoir savoir, après coup, qu'une annulation a eu lieu, **qui** l'a faite et **quand**.

*Et si l'achat annulé était la 10ᵉ étampe, et que le client a déjà bu le café gratuit qu'elle lui avait donné? Peut-on annuler une transaction d'il y a trois mois?*

### 3.2 Les rapports du patron

> « Mon comptable veut savoir combien de cafés on a servis mardi passé. Pis combien de cartes prépayées on a vendues ce mois-ci. »

- Un gérant peut consulter, pour une période qu'il choisit (un jour, une semaine, un mois, par exemple):
  - le nombre de cafés payés,
  - le nombre de cafés gratuits donnés,
  - le nombre de cafés tirés d'une carte prépayée,
  - le nombre de cartes prépayées vendues.
- Rappel : le système ne traite toujours **aucun paiement**. On compte des cafés et des cartes, jamais des dollars.

*Une transaction annulée apparaît-elle dans le rapport? Et si on annule aujourd'hui un achat de mardi passé, le rapport de mardi passé doit-il changer?*

### 3.3 Changer la règle du café gratuit

> « À partir du 1ᵉʳ novembre, nos marges ont changées, ça va prendre 12 étampes au lieu de 10. Le 13ᵉ café sera gratuit. »

- Un gérant peut modifier le nombre d'étampes requis pour un café gratuit, avec une **date d'entrée en vigueur**, sans modifier le code.
- Les rapports et l'historique des clients restent cohérents avant et après le changement.

*La nouvelle règle s'applique-t-elle aux étampes déjà accumulées? Un client qui a 10 étampes le 31 octobre perd-t-il son café gratuit le 1ᵉʳ novembre?*

### 3.4 Effacer un client

> « Une cliente m'a demandé d'effacer ses informations. J'ai-tu le droit de dire non? »

- Un gérant peut effacer un client à sa demande: son nom, son téléphone, son courriel, et tout autre renseignement qui permet de l'identifier, doivent **disparaître du système**.
- **Les statistiques du patron ne doivent pas changer**: les cafés servis à ce client restent comptés dans les rapports. Les données qui peuvent identifier le client doivent disparaitre.
- Le devoir _ne vous demande pas_ une conformité juridique complète. Mais la demande s'inscrit dans l'esprit de la **Loi 25** sur la protection des renseignements personnels au Québec (c'est une demande que vous recevrez peut-être dans votre travail professionnel).

*Effacer ou anonymiser? Que devient la carte prépayée à moitié utilisée d'un client qui part - elle a été payée? Et si votre système garde quelque part un historique des modifications (par exemple un historique des modifications de numéro de téléphone), les anciennes valeurs y sont-elles encore?*

### 3.5 Fusionner deux fiches

> « Mme Tremblay a deux fiches: une avec son cellulaire, une avec le téléphone de la maison. »

- Un gérant peut fusionner deux fiches client en une seule.
- Après la fusion, le client garde tout ce qu'il avait sur les deux fiches: étampes, récompenses, cartes prépayées et historique.

*Quel identifiant survit? 6 étampes plus 7 étampes font 13: le client gagne-t-il un café sur-le-champ? Et si on fusionne deux fiches par erreur, peut-on revenir en arrière?*

### 3.6 Des employés, et des permissions

> « J'ai engagé deux étudiants. Je veux pas que n'importe qui puisse annuler des transactions ou effacer des clients. »

- Le système distingue au moins deux rôles:
  - **barista** : chercher et créer des clients, enregistrer des achats, donner un café gratuit, vendre une carte prépayée;
  - **gérant** : tout ce que fait un barista, plus les demandes 3.1 à 3.5.
- Chaque personne se connecte avec **son propre compte**. Les écrans du comptoir ne sont plus accessibles sans connexion.
- Chaque transaction retient **qui** l'a faite.

*Un barista qui vient de se tromper de client peut-il annuler sa propre erreur de sa dernière entrée, ou doit-il toujours aller chercher le gérant?*

---

## 4. Migrer les données de la v1

Votre v1 est en production. Voici exactement ce que je ferai à la correction:

1. Je récupère votre dernière étiquette `v1.x.y`, je crée la base et je charge **vos données de démonstration de la v1**.
2. Je note les soldes de quelques clients : étampes, récompenses en réserve, cafés restants sur les cartes prépayées.
3. Je passe à votre dernière étiquette `v2.x.y` et j'applique les migrations **sur cette même base**, sans rien recharger.
4. Je compare. **Chaque solde doit être identique.**

**Exigences:**

- Les migrations de la v2 doivent s'appliquer **par-dessus** une base v1 existante.
- La v2 fournit une commande qui affiche, pour chaque client, ses étampes, ses récompenses en réserve et les cafés restants sur ses cartes prépayées — par exemple `python manage.py balances`. Documentez-la dans le README.
- Si votre modèle change de forme (par exemple, un compteur qui devient un historique), vous devez **reconstituer** les données qui n'existaient pas en v1. Expliquez comment dans un ADR.
- Le fichier `docs/migration.md` décrit la procédure pas à pas, et montre votre propre vérification : les soldes avant et après, pour vos données de démonstration de la v1.

⚠️ **Ne supprimez et ne réécrivez jamais une migration déjà livrée dans la v1.** Si vous « repartez à zéro » avec une nouvelle migration initiale, la mise à jour d'une base existante devient impossible et c'est précisément ce que je vérifie.

*Si votre v1 n'avait pas de migrations propres, c'est le moment de le découvrir. C'est une excellente matière pour une rétrospective.*

---

## 5. Livraison

Tout est dans le même dépôt GitHub. **Vous ne devez pas utilisez un autre dépôt. Veuillez utiliser le tag v2.x.y**

### 5.1 La rétrospective de la v1 (`docs/retrospective-v1.md`)

Une page, à écrire **avant** de coder la v2:

- Pour chacune des six demandes de la section 3: votre v1 permettait-elle de la satisfaire? Sinon, qu'est-ce qui bloquait?
- Qu'est-ce qui vous a surpris en lisant la liste du patron?
- Une décision de la v1 que vous prendriez autrement aujourd'hui, et pourquoi.

La rétrospective est notée sur la **lucidité**, jamais sur la qualité de la v1. Une v1 qui ne permettait presque rien, bien analysée, rapporte tous les points.

### 5.2 Les ADR (`docs/adr/`)

Un **ADR** (*Architecture Decision Record*) documente **une** décision d'architecture: le contexte qui l'impose, ce que vous avez décidé, ce que vous avez écarté, et ce que ça coûte. Une demi-page à une page suffit. Un gabarit est fourni en annexe.

**Exigences :**

- **Un ADR par demande** de la section 3 — six au minimum. Chacun tranche les zones grises de sa demande.
- **Un ADR qui réexamine votre choix de la v1** entre compteur et historique, à la lumière des six demandes. Vous pouvez **remplacer** votre choix ou le **confirmer**; si vous le confirmez, dites comment vous répondez aux demandes qui s'y opposent.
- **Un ADR ne se réécrit pas.** Si vous changez d'idée en cours de route, écrivez un nouvel ADR qui remplace l'ancien, et changez le statut de l'ancien pour « Remplacé par ADR-00X ». Ce changement d'idée est une information: ne la cachez pas.

### 5.3 Le document de conception v2 (`docs/conception.md)`

Mettez à jour votre document de la v1.

### 5.4 Les tests

Les tests automatisés deviennent **obligatoires** pour les règles d'affaires.

- Au moins **un test par demande** de la section 3, plus les règles de la v1 (le café gratuit, la carte prépayée).
- Les tests se lancent en **une seule commande**, documentée dans le README.
- Un test par zone grise tranchée est une excellente idée: il prouve que votre code fait ce que votre ADR dit.

Le _framework_ de test est libre : `python manage.py test` ou `pytest`.

### 5.5 Structure attendue

```
root/
├── README.md               ← mis à jour : comptes de démo, tests, commande des soldes
├── AI-USAGE.md             ← complété avec une section « Devoir 2 »
├── docs/
│   ├── conception.md       ← mis à jour (v2)
│   ├── retrospective-v1.md ← nouveau
│   ├── migration.md        ← nouveau
│   └── adr/                ← nouvea
│       ├── 0001-….md
│       └── …
└── src/                    ← le code du projet Django
```

### 5.6 Deux chemins d'installation

Les deux doivent fonctionner en suivant votre README:

1. **Une installation nouvelle**, avec ses propres données de démonstration.
2. **Une mise à jour d'une base v1** existante. On peux télécharger la v1 avec votre tag, puis faire la mise à jour.

Les données de démonstration de la v2 contiennent au moins : un compte **barista** et un compte **gérant** (identifiants dans le README), une transaction à annuler, deux fiches à fusionner, un client à effacer, et des transactions réparties sur plusieurs jours pour les rapports.

---

## Annexe — Gabarit d'ADR

Ce format vient de Michael Nygard, « Documenting Architecture Decisions » (2011). Numérotez vos ADR dans l'ordre où vous les prenez : `0001-…`, `0002-…`.

```markdown
# ADR-0003 — Comment annuler une transaction

- **Statut :** Accepté        (Proposé · Accepté · Remplacé par ADR-00X)
- **Date :** 2026-10-07
- **Demande concernée :** §3.1

## Contexte
Ce qui vous oblige à décider : la demande du patron, les contraintes,
ce que votre v1 permettait ou non.

## Décision
Ce que vous avez choisi, en une ou deux phrases. « Nous allons… »

## Options écartées
Chaque alternative sérieuse que vous avez considérée, et pourquoi
vous ne l'avez pas retenue.

## Conséquences
Ce que cette décision rend facile, ce qu'elle rend difficile,
et ce qu'elle vous coûte.
```
