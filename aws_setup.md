# Guide AWS sans frais pour le devoir 4

### 8INF228 — Adaptation et qualité des applications · Automne 2026

> **But de ce guide :** que votre équipe déploie le devoir 4 sur AWS **sans payer**. Lisez-le en entier **avant** de créer un compte. Si une étape vous demande de payer quoi que ce soit, vous n'avez pas bien suivi les instructions.

*Informations vérifiées le 2 novembre 2026. AWS change ses offres régulièrement : si un écran ou une offre ne correspond pas à ce guide, fiez-vous à l'écran et prévenez-moi par courriel!*

---

## 1. Résumé

| | |
|---|---|
| **Combien de comptes ?** | **Un seul par équipe.** Les deux coéquipiers y accèdent. |
| **Qui le crée ?** | Un coéquipier qui **n'a jamais eu de compte AWS**. Seuls les nouveaux comptes reçoivent les crédits. |
| **Combien de crédits ?** | **100 $ US** à l'inscription, et **jusqu'à 100 $ US de plus** en faisant cinq petites activités. |
| **Combien coûte le devoir ?** | Une petite instance EC2 consomme de l'ordre de **10 à 25 $ US de crédits par mois**. Vous en avez largement assez. |
| **Peut-on recevoir une facture ?** | **Non, tant que vous restez sur le plan gratuit (*Free plan*).** Sur ce plan, AWS ne facture rien : le compte se ferme quand les crédits sont épuisés ou après six mois. |
| **Le piège à éviter** | Passer au **plan payant** (*Paid plan*). Ne cliquez **jamais** sur *Upgrade plan*. |

---

## 2. Comment fonctionne le plan gratuit d'AWS

Depuis juillet 2025, tout **nouveau** compte AWS commence sur le **plan gratuit** (*Free plan*) :

- vous recevez **100 $ US de crédits** dès l'inscription, et pouvez en gagner **jusqu'à 100 $ US de plus** ;
- le plan dure **6 mois**, ou jusqu'à ce que les crédits soient épuisés — le premier des deux ;
- **aucun frais** pendant ce temps: le plan gratuit *« vous garantit de ne subir aucun frais »* tant que vous ne passez pas au plan payant ;
- à la fin, le compte **se ferme tout seul**. Vos ressources et vos données sont conservées 90 jours, puis supprimées ;
- certains services coûteux ne sont pas offerts sur ce plan. Ceux nécessaire pour le devoir 4 (EC2, un disque, un adresse IP) sont disponible.

**Ce que ça veut dire pour vous :** le devoir dure environ un mois. Même en laissant tourner votre instance jusqu'au 21 décembre, vous n'utiliserez qu'une partie de vos crédits, et vous ne pouvez pas être facturés.

---

## 3. Créer le compte (≈ 15 minutes)

**Avant de commencer, ayez sous la main :**

- une **adresse courriel** qui n'a jamais servi pour AWS (votre courriel UQAC convient très bien, mais n'est pas obligatoire) ;
- un **téléphone** pour recevoir un code de vérification ;
- possiblement une **carte de paiement**. AWS dit qu'un moyen de paiement *n'est pas exigé pour la plupart des nouveaux clients*, mais peut en demander un **pour vérifier votre identité**. Sur le plan gratuit, **rien n'est prélevé**.

**Les étapes :**

1. Allez sur **aws.amazon.com/free** et choisissez **Create a free account** (*Créer un compte gratuit*).
2. Entrez le courriel et un nom de compte (par exemple `8inf228-equipe-café`). Confirmez le code reçu par courriel.
3. Choisissez un **mot de passe fort** pour l'utilisateur *root*. Notez-le préférablement dans un gestionnaire de mots de passe.
4. Coordonnées : choisissez **Personnel** (*Personal*).
5. **Si on vous demande de choisir un plan, choisissez *Free plan*.** (Les nouveaux comptes y sont normalement inscrits automatiquement.)
6. Vérification d'identité par texto ou appel.
7. Plan de soutien (*Support plan*) : **Basic — gratuit**. Rien d'autre.

---

## 4. Gagner les 100 $ supplémentaires (≈ 30 minutes, facultatif mais utile)

Dans la console AWS, sur la page d'accueil, le panneau **Explore AWS** propose des activités (*Earn AWS credits*). Chacune rapporte **20 $ US**. Les activitées d'AWS sont :

| Activité | Utile pour le devoir ? |
|---|---|
| **AWS Budgets** — créer un budget avec alerte | ✅ Oui  |
| **Amazon EC2** — lancer une instance, puis la **terminer** | ✅ Oui |
| **Amazon RDS** — découvrir la configuration d'une base | Facultatif |
| **AWS Lambda** — une petite application avec une URL de fonction | Facultatif |
| **Amazon Bedrock** — envoyer une requête dans le terrain de jeu texte | Facultatif |

Les activités doivent être faites dans les **6 mois** suivant la création du compte. Elles ne sont **pas** notées.

---

## 5. Sécuriser le compte et y donner accès à votre coéquipier

Votre compte AWS va héberger une application ouverte sur Internet. Dix minutes de sécurité maintenant vous évitent bien des ennuis.

1. **Activez l'authentification multifacteur (MFA) sur l'utilisateur root** : menu du compte (en haut à droite) → *Security credentials* → *Assign MFA device*. Utilisez une application comme Google Authenticator, Microsoft Authenticator ou votre gestionnaire de mots de passe.
2. **N'utilisez plus l'utilisateur racine au quotidien.** Dans **IAM**, créez **un utilisateur par coéquipier**, avec accès à la console et la permission `AdministratorAccess`. Activez la MFA sur chacun.
3. **Ne créez pas de clés d'accès** (*access keys*) à moins d'en avoir vraiment besoin (par exemple pour Terraform ou l'AWS CLI sur votre ordinateur). Si vous en créez :
   - elles ne vont **jamais** dans le dépôt Git, ni dans un `.env` commité, ni dans une capture d'écran ;
   - supprimez-les à la fin du devoir.

> ⚠️ Des robots parcourent GitHub en permanence à la recherche de clés AWS publiées par erreur. Une clé exposée est utilisée en quelques minutes. Si ça vous arrive, **supprimez la clé dans IAM immédiatement**!

---

## 6. Mettre un filet de sécurité : le budget

1. Dans la console, ouvrez **Billing and Cost Management** → **Budgets** → **Create budget**.
2. Choisissez un modèle (*template*) :
   - **Zero spend budget** — vous avertit dès que **le moindre sou** serait dépensé au-delà de vos crédits ; ou
   - **Monthly cost budget** — par exemple 10 $ US par mois.
3. Entrez **les courriels des deux coéquipiers** pour les alertes.
4. Faites une **capture d'écran** de votre budget pour l'ajouter dans le devoir.

**Pour suivre vos crédits :** *Billing and Cost Management* → *Credits* (le solde restant) et *Bills* ou *Cost Explorer* (ce que vous consommez par jour). Notez le solde au début et à la fin du devoir.

---

## 7. Choisir sans se ruiner

**La région :** prenez **Canada (Centre) — `ca-central-1`**, à Montréal. Vos données restent au Canada (c'est dans l'esprit de la loi 25, vue au devoir 2), et la latence est faible depuis le Saguenay. Vérifiez **toujours** la région affichée en haut à droite de la console avant de créer quelque chose : une instance oubliée dans une autre région continue de consommer des crédits.

**Le type d'instance :** l'énoncé exige un type **admissible au palier gratuit**. Pour les comptes créés depuis juillet 2025, AWS liste notamment `t3.micro`, `t3.small`, `t4g.micro` et `t4g.small` (la console les marque *Free tier eligible*).

| Type | Mémoire | Pour le devoir 4 |
|---|---|---|
| `t3.micro` | 1 Gio | Assez serré : la construction des images Docker peut manquer de mémoire. Ajoutez un fichier d'échange (*swap*) si vous prenez celui-ci. |
| **`t3.small`** | **2 Gio** | **Recommandé** : PostgreSQL, Django et le proxy y tiennent à l'aise. |
| `t4g.micro`, `t4g.small` | 1–2 Gio | Processeur ARM : fonctionne si vos images se construisent sur l'instance, mais une image construite sur votre PC x86-64 ne démarrera pas. |

**Le disque :** 20 à 30 Gio en `gp3` suffisent. Le disque de 8 Gio proposé par défaut se remplit vite avec les images Docker.

**Ce qui consomme des crédits — par ordre de taille :**

| Ressource | Ordre de grandeur | À retenir |
|---|---|---|
| L'instance EC2 qui tourne | ~0,01 à 0,025 $ US de l'heure | Une instance **arrêtée** (*stopped*) ne coûte plus rien en calcul ; son disque, si. |
| L'adresse IPv4 publique | 0,005 $ US de l'heure (~3,60 $ par mois) | Une **adresse IP élastique** non associée à une instance coûte aussi : libérez-la à la fin. |
| Le disque (EBS) | Quelques dollars par mois | Il reste facturé tant qu'il existe, même instance arrêtée. |

**À ne jamais créer pour ce devoir :** un *NAT Gateway*, un *Load Balancer*, une base de donnée RDS « Multi-AZ », une grosse instance (`xlarge` et plus). Ce sont les façons les plus rapides de vider vos crédits.

---

## 8. Le bonus : un nom de domaine gratuit

Pour le bonus, il vous faut **un nom de domaine à vous**. Inutile de le payer :

1. Demandez le **GitHub Student Developer Pack** sur **education.github.com/pack**, avec votre courriel UQAC. ⚠️ **La vérification peut prendre plusieurs jours : faites la demande dès maintenant.**
2. Le Pack offre notamment, au moment d'écrire ces lignes :
   - un domaine **`.me`** gratuit pendant 1 an chez **Namecheap** ;
   - un domaine **`.tech`** gratuit pendant 1 an ;
   - un domaine gratuit dans une liste d'extensions chez **Name.com**.
3. Chez votre registraire, créez un **enregistrement A** qui pointe vers l'**adresse IP élastique** de votre instance.
4. Obtenez un certificat HTTPS **gratuit** de **Let's Encrypt** ou avec **certbot** et nginx.

---

## 9. Après la correction : tout démanteler

Gardez l'application en ligne **jusqu'à ce que je vous indique que j'ai corrigé votre devoir**. Ensuite, vous pouvez :

1. faites une dernière sauvegarde de la base, si vous voulez garder vos données ;
2. **terminez** (*Terminate*) l'instance EC2 — pas seulement « arrêter » ;
3. **libérez** (*Release*) l'adresse IP élastique ;
4. supprimez les volumes EBS et les instantanés (*snapshots*) qui resteraient ;
5. supprimez les clés d'accès IAM que vous auriez créées ;
6. vérifiez, **dans chaque région où vous avez travaillé**, que la page EC2 n'affiche plus rien.

Si vous ne faites rien, le compte se fermera de toute façon à la fin des six mois, sans frais. Mais démanteler proprement fait partie du métier — et de votre guide de déploiement que vous devez réaliser pour votre devoir.
