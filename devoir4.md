# Devoir 4

### 8INF228 — Adaptation et qualité des applications
### Système de fidélité pour un café · version 4 — en ligne

| | |
|---|---|
| **Pondération** | 8 % de la note finale |
| **Bonus** | **+1 %** sur la note finale : votre propre nom de domaine, en HTTPS (§6) |
| **Équipes** | Les mêmes qu'aux devoirs précédents — c'est le même dépôt |
| **Remise** | Dépôt GitHub, *tag* `v4.x.y`, **et l'adresse (URL) de votre application**, déposée sur Moodle |
| **Échéance** | 30 novembre |
| **IA générative** | Autorisée, à déclarer dans `AI-USAGE.md` |
| **Coût** | **Aucun.** Lisez le *Guide — AWS sans frais* avant de créer quoi que ce soit (§7) |

---

## 1. Mise en situation

Le deuxième café ouvre ses portes.

> « Lundi, on ouvre l'autre café. Faut que les deux caisses marchent, pis que je puisse voir mes chiffres de chez nous. »

Au devoir 3, vous avez préparé ce moment : l'application tourne dans des conteneurs, sa configuration vient de variables d'environnement, et un proxy inverse est devant elle. Le jour du déploiement, **seules des variables d'environnement devraient changer**.

Vous déployez votre v3 sur **un serveur dans le nuage (AWS)**, et les deux cafés s'y connectent avec un navigateur. Vous me remettez **l'adresse** de votre application : je la corrige **en ligne**, comme le ferait votre boss.

*Quelques questions pour commencer :*

- *Relisez votre `.env.example` et votre diagramme de déploiement du devoir 3. Qu'est-ce qui change, exactement, quand la machine n'est plus la vôtre ?*
- *Votre application est maintenant visible par n'importe qui sur Internet. Qu'est-ce qui était acceptable en local et ne l'est plus ?*
- *Si le serveur redémarre à 3 h du matin, que se passe-t-il à l'ouverture du café ?*

---

## 2. Ce qui est demandé

**Aucune nouvelle fonctionnalité.** Tout ce qui fonctionnait en v3 — les deux cafés, les rôles, les rapports, les six demandes du devoir 2 — doit fonctionner **en ligne**.

Vous déployez **votre système du devoir 3** — votre `compose.yaml`, votre `Dockerfile` ou `Podfile`, votre proxy inverse, votre PostgreSQL — sur **une seule machine virtuelle Amazon EC2**.

Si votre devoir 3 était bien fait, le déploiement ne demande **aucune modification du code** : seulement un fichier `.env` sur le serveur, et peut-être un fichier Compose de production. Si vous devez modifier le code, faites-le, et **écrivez pourquoi dans un ADR** : c'est une information précieuse sur votre devoir 3, pas une faute et **vous ne serez pas pénalisé**.

---

## 3. Exigences du déploiement

| # | Exigence |
|---|---|
| **A1** | **Une adresse publique qui fonctionne.** L'application répond à votre URL (par exemple `http://15.222.x.x` ou `http://ec2-….compute.amazonaws.com`) **dès la remise**. |
| **A2** | **Une seule instance EC2**, d'un type admissible au palier gratuit (`t3.micro`, `t3.small`, `t4g.micro` ou `t4g.small`). Région recommandée : **Canada (Centre) — `ca-central-1`**. |
| **A3** | **Votre système du devoir 3**, démarré avec Docker Compose ou Podman sur l'instance. PostgreSQL tourne dans son conteneur, avec un volume. Pas de base de données gérée (RDS), pas de *Load balancer*. |
| **A4** | **Seul le proxy est exposé à Internet**, sur le port 80 (et 443 pour le bonus). Le groupe de sécurité (*security group*) n'ouvre **aucun autre port à tout le monde** : PostgreSQL et Gunicorn ne sont jamais accessibles de l'extérieur, et SSH (22) est limité à votre adresse IP — ou fermé, si vous utilisez EC2 Instance Connect ou Session Manager. |
| **A5** | **Configuration de production.** `DEBUG = False` ; `ALLOWED_HOSTS` et `CSRF_TRUSTED_ORIGINS` correspondent à votre URL ; une **nouvelle** `SECRET_KEY` et un **nouveau** mot de passe de base de données, différents de ceux de `.env.example`. Le `.env` du serveur n'est **jamais** dans le dépôt. |
| **A6** | **Les comptes ne sont plus publics.** Votre application est sur Internet : les mots de passe de démonstration écrits dans votre README depuis le devoir 3 n'y sont plus valables. Créez des comptes de correction (propriétaire, un gérant par café, barista) avec des mots de passe **qui ne sont écrits nulle part dans le dépôt**, et transmettez-les-moi **uniquement par courriel**. |
| **A7** | **Le redémarrage.** Si l'instance redémarre, l'application revient **toute seule** (Docker ou Podman démarre au boot, conteneurs avec une politique `restart`), et les données sont intactes. Je le vérifierai en regardant votre preuve. |
| **A8** | **Une sauvegarde réelle.** Vous avez fait au moins une sauvegarde de la base de production (par exemple avec `pg_dump`) et vous l'avez **restaurée** au moins une fois, ailleurs qu'en production. |
| **A9** | **Zéro dollar.** Un **budget AWS** avec alerte par courriel est configuré, et vous savez combien votre déploiement consomme de crédits par jour. |

*Faut-il construire les images sur le serveur, ou les construire ailleurs et les y envoyer ? Faut-il une adresse IP élastique ? Les migrations s'appliquent-elles toutes seules au démarrage en production, comme en local ? Vous tranchez, et vous l'écrivez dans un ADR.*

---

## 4. Documenter le déploiement — à partir de votre devoir 3

Le devoir 3 vous a fait écrire un `README`, un `.env.example`, des ADR et un **diagramme de déploiement**. Ce devoir les prolonge : quelqu'un d'autre doit pouvoir **refaire votre déploiement sans vous**.

### 4.1 Le guide de déploiement — `docs/deploiement.md`

Imaginez que vous l'écriver pour **un coéquipier qui n'a rien fait** : avec ce document et un compte AWS vide, il doit pouvoir remettre l'application en ligne.

- **Créer l'infrastructure** : région, type d'instance, image (AMI), taille du disque, groupe de sécurité (chaque règle et sa raison), paire de clés ou autre accès.
- **Préparer le serveur** : installer Docker ou Podman et Docker Compose, récupérer le code au bon *tag*.
- **Configurer** : chaque variable du `.env` de production, avec ce qui change par rapport au `.env.example` — **sans jamais y écrire une vraie valeur secrète**.
- **Démarrer et vérifier** : les commandes, et comment savoir que tout va bien.
- **Mettre à jour** : déployer une nouvelle version (`v4.0.1`) sans perdre de données.
- **Exploiter** : consulter les journaux, redémarrer, **sauvegarder et restaurer** la base.
- **Démanteler** : tout supprimer après la correction, sans rien oublier qui consomme des crédits.

**Automatiser, c'est mieux.** Une procédure manuelle complète et exacte rapporte la majorité des points. Une procédure **automatisée** — un script AWS CLI, un script de démarrage (*user data*, cloud-init) ou **Terraform** — rapporte le reste (§8). Ce n'est pas obligatoire.

### 4.2 Les preuves — `docs/deploiement.md`, section « Vérification »

Vous ne pouvez pas me donner accès à votre compte AWS. Montrez-moi plutôt, avec des **captures d'écran vidéos** et des **sorties de commandes** (en masquant tout secret) :

- les règles entrantes de votre groupe de sécurité ;
- `docker compose ps` sur le serveur, après un redémarrage de l'instance ;
- votre sauvegarde et votre restauration (commandes et résultat) ;
- votre budget AWS et le solde de vos crédits.

### 4.3 Le document de conception — `docs/conception.md`

- Mettez à jour le **diagramme de déploiement** du devoir 3 : Internet, le groupe de sécurité, l'instance EC2, ses conteneurs, le volume, et le chemin d'une requête du navigateur jusqu'à la base.

### 4.4 Les ADR — `docs/adr/`

- **Un ADR par zone grise** des exigences de la section 3.
- **Un ADR** si vous avez dû modifier le code pour déployer.
- Si le déploiement change une décision d'un ADR précédent, écrivez un nouvel ADR et changez le statut de l'ancien pour « Remplacé par ADR-00X ».

---

## 5. Hors périmètre

> **Toujours aucun paiement. Toujours pas d'accès client.**

- **Une seule instance**, une seule région.
- Pas de base de données gérée (RDS), pas de CDN, pas de stockage S3 obligatoire.
- Pas d'intégration ni de déploiement continus (CI/CD) obligatoires — si vous en faites, documentez-les.
- **Aucune dépense.** Si une étape vous demande de payer quoi que ce soit, arrêtez-vous.

---

## 6. Bonus — votre nom de domaine, en HTTPS (+1 % sur la note finale)

Le patron ne veut pas taper une adresse IP. Pour obtenir le bonus :

1. un **nom de domaine à vous** (par exemple `comptoir-equipe7.me`) — le *GitHub Student Developer Pack* en offre gratuitement, voir le guide AWS ;
2. un **enregistrement DNS de type A** qui pointe vers votre instance (une adresse IP élastique est fortement recommandée : sans elle, l'adresse change à chaque arrêt de l'instance) ;
3. l'application servie en **HTTPS**, avec un certificat valide (Let's Encrypt, par exemple, par Caddy ou par certbot), et `http://` qui redirige vers `https://`.

Le bonus est entier ou nul. Décrivez votre configuration DNS et HTTPS dans `docs/deploiement.md`.

---

## 7. Coût : rien

Lisez le [**Guide — AWS sans frais pour le devoir 4**](aws_setup.md), **avant** de créer votre compte. En résumé :

- **Un seul compte AWS par équipe suffit.** Créez-le avec le coéquipier qui n'a jamais eu de compte AWS : seuls les nouveaux comptes reçoivent les crédits.
- Restez sur le **plan gratuit** (*Free plan*). Sur ce plan, AWS ne vous facture rien : le compte se ferme quand les crédits sont épuisés ou après six mois.
- Les crédits de départ couvrent largement ce devoir. Un déploiement comme celui demandé consomme de l'ordre de **10 à 25 $ US par mois**.

---

## 8. Livraison

### 8.1 Dans le dépôt

```
root/
├── README.md                ← mis à jour : l'URL de production, et où trouver le guide
├── AI-USAGE.md              ← complété avec une section « Devoir 4 »
├── .env.example             ← à jour ; aucune vraie valeur secrète
├── compose.yaml             ← celui du devoir 3
├── compose.prod.yaml        ← facultatif : ce qui diffère en production
├── docs/
│   ├── deploiement.md       ← nouveau
│   ├── conception.md        ← diagramme de déploiement
│   └── adr/                 ← complété
└── src/
```

Si vous automatisez (Terraform, script AWS CLI, *user data*), mettez ces fichiers dans un dossier `infra/` et expliquez comment les utiliser. **Aucun identifiant AWS** (clé d'accès, clé secrète, fichier `.pem`) ne doit jamais se retrouver dans le dépôt.

### 8.2 Par courriel

Une seule remise par équipe, avec :

- l'**URL** de votre application ;
- les **comptes de correction** : identifiants et mots de passe ;
- un lien vers vos vidéos de preuves ;
- le nom de votre étiquette `v4.x.y`.

### 8.3 L'étiquette

```bash
git tag -a v4.0.0 -m "Devoir 4"
git push origin v4.0.0
```

### 8.4 Après la correction

**Gardez l'application en ligne jusqu'à ce que je confirme votre correction.** Ensuite, suivez votre propre procédure de démantèlement.
