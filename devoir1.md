# Devoir 1

### 8INF228 — Adaptation et qualité des applications
### Système de fidélité pour un café · version 1

| | |
|---|---|
| **Pondération** | 7 % de la note finale |
| **Équipes** | 2 personnes |
| **Durée** | 2 semaines |
| **Remise** | Dépôt GitHub, *tag* `v1.x.y` |
| **Échéance** | 28 septembre |
| **IA générative** | Autorisée, à déclarer (voir §6) |

---

## 1. Mise en situation

Vous êtes barista dans un petit café indépendant. Vous savez coder — c'est pour ça que le propriétaire est venu vous voir.

> « On a des cartes en carton avec des étampes. Les clients les perdent. Moi je perds le compte. J'aimerais ça avoir ça dans un ordinateur. Tu penses que tu peux me faire quelque chose ? »

Vous avez deux semaines et votre ordinateur. Le logiciel tournera en local **sur la machine du comptoir (votre machine)**, rien de plus. Il n'y a pas de serveur, pas de nuage, pas de client à part vous et le proprio.

⚠️ **Avant de commencer**

Ce devoir est le **premier d'une série** qui va durer toute la session. Le même dépôt Git, le même produit, qui grossit à chaque devoir : plus de fonctionnalités, plus d'utilisateurs, plus d'exigences.

**Cela ne veut pas dire qu'il faut tout prévoir maintenant.** Au contraire : construisez ce qu'on vous demande **aujourd'hui**, en vous assurant de faire des choix judicieux d'implémentation.

---

## 2. Requis fonctionnels

Ces exigences sont **fermes**. Elles constituent le minimum attendu.

### 2.1 Clients

- Enregistrer un client dans le système.
- Retrouver un client existant rapidement, au comptoir, pendant qu'il attend son café.

### 2.2 Un programme de fidélité - le onzième gratuit

- Enregistrer qu'un client a acheté un café.
- **Après 10 cafés achetés, le 11e est gratuit.**

### 2.3 Le programme de fidélité — la carte prépayée

- Le café vend aussi des **cartes prépayées : 11 cafés pour le prix de 10.**
- Le système doit permettre d'émettre une telle carte à un client et d'en suivre l'utilisation.

### 2.4 Administration

- Une interface d'administration permettant de **voir la liste des clients** et d'en **créer un rapidement**.
- L'administration de Django répond à cette exigence, à condition d'être **configurée**.

### 2.5 Écrans du comptoir

**Deux écrans sont obligatoires**, en dehors de l'administration :

1. **Trouver ou créer un client** — un barista pressé doit y arriver en quelques secondes.
2. **Enregistrer un achat** — et, quand le client y a droit, lui donner son café gratuit.

### 2.6 Hors périmètre

> **Le système ne traite aucun paiement. Ni maintenant, ni dans les devoirs suivants.**

Pas de carte de crédit, pas de terminal de paiement, pas de caisse enregistreuse, Quand une carte prépayée est vendue, l'argent change de mains **à la caisse, hors du système** ; votre application ne fait qu'en **prendre acte**.

Si vous vous surprenez à écrire une classe `Payment`, arrêtez-vous et relisez ce paragraphe.

---

## 3. Les zones grises — à trancher vous-mêmes

L'énoncé ci-dessus est volontairement incomplet. Par exemple, ces trois questions n'ont **pas** de réponse dans ce document, et n'en auront pas si vous me les posez.

Ce sont des questions d'implémentation. Il n'y a pas une bonne réponse : il y a des réponses **défendables** et des réponses **irréfléchies**. Vous devez trancher, implémenter votre choix, et le **justifier par écrit** dans votre documentation.

### Le onzième café gratuit et la carte prépayée : un seul mécanisme ou deux ?

Dans les deux cas, le client finit avec des cafés « déjà payés ». Est-ce que ce sont deux façons d'alimenter le même compteur, ou deux concepts distincts qui coexistent ?

*Quelques questions qui devraient vous aider à trancher : si un client a une carte prépayée entamée et qu'il achète un café au comptoir, ça compte pour ses tampons ? Le proprio veut-il pouvoir distinguer les deux dans ses chiffres ?*

### Le café gratuit : automatique ou en réserve ?

Au 11e passage, le café est-il **automatiquement** gratuit ? Ou le client accumule-t-il une **récompense** qu'il dépense quand il le décide — peut-être pas aujourd'hui, peut-être sur une boisson plus chère ?

*Que se passe-t-il si un client accumule trois récompenses sans les utiliser ? Est-ce que ça expire ?*

### Comment reconnaît-on un client au comptoir ?

Numéro de téléphone ? Courriel ? Nom ? Un code sur une carte physique ? Autre chose ?

*Votre choix a des conséquences sur la rapidité au comptoir, sur les doublons dans la base, et sur ce qui arrive quand deux clients s'appellent Tremblay.*

---

## 4. Contraintes techniques

| Élément | Exigence |
|---|---|
| **Langage / cadriciel** | Python et **Django**. Obligatoire. |
| **Gabarit de projet** | **`cookiecutter-django` est fortement conseillé**, sans être obligatoire. `django-admin startproject` reste accepté. |
| **Base de données** | Libre. **SQLite suffit** et convient parfaitement au devoir 1. |
| **Exécution** | En local, sur votre machine. Aucun déploiement sur serveur n'est demandé, ni de containerisation. |
| **Tests automatisés** | **Non exigés** pour ce devoir. Rien ne vous empêche d'en écrire. |
| **Hébergement du code** | **GitHub.** Dépôt public, ou privé avec l'enseignant ajouté comme collaborateur. |

### Sur cookiecutter-django

C'est un générateur de projets Django utilisé en industrie. Il produit une structure complète — configuration séparée par environnement, gestion des utilisateurs, outillage de qualité — au lieu du squelette minimal de `startproject`.

**Pourquoi je le conseille :** vous n'aurez pas nécessairement à réorganiser votre projet lors des devoirs subséquent.
**Pourquoi il n'est pas obligatoire :** il génère beaucoup de fichiers, dont certains ne vous serviront pas avant plusieurs semaines. Si cette abondance vous paralyse, partez de `startproject` — ce n'est pas une grave.

```bash
pip install cookiecutter
cookiecutter https://github.com/cookiecutter/cookiecutter-django
```

Les réponses a ces questions sont des choix architecturaux de conception qui est intéressant de documenter.

---

## 5. Livraison

Tout est dans le dépôt GitHub. **Il n'y a rien à déposer ailleurs.** Vous devez cependant m'ajouter dans votre projet.

### 5.1 Exemple de dépôt

```
root/
├── README.md              ← comment faire tourner le projet
├── AI-USAGE.md            ← votre déclaration d'usage de l'IA
├── docs/
│   └── conception.md      ← le document de conception
└── src/                   ← le code du projet Django
```
