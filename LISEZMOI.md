# XAU Guardian — mise en ligne

Le site a 4 fichiers : `index.html` (page d'accueil), `auth.html` (connexion / inscription), `dashboard.html` (espace après connexion), `firebase-config.js` (tes clés).

## Étape 1 — Créer le projet Firebase (gratuit)

1. Va sur https://console.firebase.google.com et connecte-toi avec un compte Google.
2. Clique sur **Ajouter un projet**, nomme-le par exemple `xau-guardian`, termine la création.
3. Dans le menu de gauche : **Build > Authentication** → **Get started**.
4. Onglet **Sign-in method** → active **E-mail/Mot de passe**.
5. Retourne à la racine du projet (icône maison), clique l'icône **engrenage** en haut à gauche → **Paramètres du projet**.
6. En bas, section **Vos applications** → clique l'icône **</>** (Web) → donne un nom (ex. `xau-guardian-web`) → **Enregistrer l'application**.
7. Firebase affiche un bloc `firebaseConfig = {...}`. Copie ces valeurs.

## Étape 2 — Renseigner tes clés

Ouvre `firebase-config.js` et remplace chaque `REPLACE_...` par la valeur correspondante copiée à l'étape 1.

## Étape 3 — Autoriser ton domaine GitHub Pages

Une fois le site en ligne (étape 4), dans Firebase : **Authentication > Settings > Authorized domains** → **Add domain** → ajoute ton adresse GitHub Pages (ex. `tonpseudo.github.io`).

## Étape 4 — Mettre en ligne sur GitHub Pages

1. Crée un nouveau dépôt GitHub (ou utilise un dépôt existant), par exemple `xau-guardian-site`.
2. Mets les 4 fichiers de ce dossier à la racine du dépôt.
3. Dans le dépôt : **Settings > Pages** → Source : **Deploy from a branch** → Branch : `main` / dossier `/root` → **Save**.
4. Après une minute ou deux, le site est accessible à `https://tonpseudo.github.io/xau-guardian-site/`.

## Ce que fait vraiment ce site

- `index.html` : vitrine publique, aucune donnée sensible.
- `auth.html` : vraie création de compte et connexion via Firebase Authentication (les mots de passe ne transitent jamais par toi, Firebase les gère).
- `dashboard.html` : page protégée, redirige vers `auth.html` si personne n'est connecté. Pour l'instant elle affiche juste "aucune alerte" — c'est un point de départ pour brancher tes vraies alertes plus tard (par ex. en lisant les messages de ton canal Telegram XAU Guardian, ou une base de données Firestore).

## Pour aller plus loin

Si tu veux que les alertes envoyées sur ton bot Telegram apparaissent aussi dans le tableau de bord, il faudra une base de données (Firestore, gratuite aussi) qui stocke chaque signal envoyé par ton bot Python, et que `dashboard.html` vienne lire. Dis-le-moi quand tu en es là, je peux le brancher.
