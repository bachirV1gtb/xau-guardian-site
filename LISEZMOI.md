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

## Générateur de niveaux (outil-signal.html)

Ce nouvel outil calcule une entrée, un stop et deux objectifs à partir de vraies données de prix (pas d'une image), avec une stratégie de rupture de canal calibrée sur la volatilité (ATR). Il a besoin d'une clé API TwelveData, gratuite :

1. Va sur https://twelvedata.com et crée un compte gratuit.
2. Sur ton tableau de bord TwelveData, ta clé API est affichée directement (section "API Key").
3. Copie-la dans `twelvedata-config.js`, à la place de `REPLACE_WITH_YOUR_TWELVEDATA_API_KEY`.
4. Ajoute ce fichier avec les autres sur GitHub (même méthode que pour les autres : `Add file > Create new file`, colle le contenu, `Commit changes`).

Le plan gratuit TwelveData limite le nombre de requêtes par minute et par jour — largement suffisant pour un usage personnel ou une petite communauté, mais à surveiller si tu ouvres l'outil à beaucoup de monde en même temps.

**Important : cet outil n'est pas un conseil financier.** Les niveaux affichés sont une lecture technique indicative basée sur une stratégie parmi d'autres (rupture de canal + volatilité), pas une prédiction garantie. Si tu comptes un jour le vendre à des membres, garde bien cet avertissement visible — le présenter comme un système infaillible pourrait causer de vraies pertes à tes membres et poser un problème légal (conseil en investissement non autorisé, selon les pays).

## Débloquer la formation pour tes Membres (Firestore)

La page `formation.html` vérifie, pour chaque personne connectée, si son e-mail existe dans une liste de membres stockée sur Firebase (Firestore). Voici comment la mettre en place, une seule fois :

1. Dans la console Firebase (console.firebase.google.com), ouvre ton projet **xau-guardian**.
2. Cherche **Firestore Database** dans la barre de recherche du menu.
3. Clique **Créer une base de données** (Create database).
4. Choisis **Mode production**, puis une région proche de toi, et valide.
5. Une fois la base créée, va dans l'onglet **Règles** (Rules) et remplace le contenu par :

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /members/{email} {
      allow read: if request.auth != null;
      allow write: if false;
    }
  }
}
```

6. Clique **Publier** (Publish).

### Ajouter un Membre après un paiement Stripe

Après chaque paiement reçu sur Stripe (visible dans ton tableau de bord Stripe, section Clients ou Paiements) :

1. Retourne dans **Firestore Database** sur Firebase, onglet **Données**.
2. Clique **Démarrer une collection**, nomme-la exactement `members`.
3. Comme **ID de document**, tape l'adresse e-mail exacte utilisée par la personne pour payer (et pour se connecter sur ton site — les deux doivent correspondre).
4. Ajoute un champ, par exemple `actif` (type booléen) à `true` — le contenu du champ importe peu, seule l'existence du document compte.
5. Enregistre.

La personne aura alors immédiatement accès à `formation.html` la prochaine fois qu'elle rechargera la page. Cette étape reste manuelle pour l'instant ; si le nombre de membres grandit, on pourra automatiser ça avec un webhook Stripe relié à Firebase.

## Analyse avancée (analyse-avancee.html)

Nouvel outil réservé aux Membres, qui combine 4 lectures indépendantes du marché au lieu d'une seule :
- **Tendance** — comparaison de deux moyennes mobiles exponentielles (EMA 20 et EMA 50).
- **Momentum** — RSI (14 périodes).
- **Élan** — MACD (12, 26, 9).
- **Structure de prix** — la même logique de rupture de canal que le générateur de niveaux gratuit.

L'outil compte combien de ces 4 lectures pointent dans la même direction ("confluence"), et ne propose une entrée/stop/objectifs que si au moins 3 des 4 sont alignées — sinon il indique honnêtement que les signaux sont mitigés plutôt que d'inventer un niveau. Il utilise la même clé API TwelveData que les autres outils et la même vérification d'accès Membre (Firestore) que la formation.

## Page "Mon compte" (mon-compte.html)

Nouvelle page où chaque personne connectée voit son e-mail, son statut d'abonnement, et peut demander un lien pour changer son mot de passe (envoyé par e-mail, sans que tu aies à intervenir).

Pour afficher une date de renouvellement sur cette page, ajoute un champ optionnel quand tu ajoutes un Membre dans Firestore :
- **Field** : `renouvellement`
- **Type** : `string`
- **Value** : la date au format que tu préfères, par exemple `12 décembre 2026`

Si tu ne renseignes pas ce champ, la page affiche simplement "⭐ Membre" sans date — ça reste fonctionnel sans cette étape.

## Alertes de prix personnalisées (alertes.html)

Nouvel outil réservé aux Membres. Chaque personne peut définir un prix cible sur un instrument ; tant que la page reste ouverte dans son navigateur, le site vérifie le prix toutes les minutes et déclenche une notification navigateur si la condition est atteinte. **Ce n'est pas une notification push envoyée sur le téléphone quand le site est fermé** — c'est une limitation assumée d'un site sans serveur, à rappeler à tes membres.

### Mettre à jour les règles Firestore (obligatoire)

Cette fonctionnalité a besoin d'une nouvelle collection `alerts`, avec des règles différentes de `members` (chaque personne doit pouvoir créer/lire/supprimer ses propres alertes, mais pas celles des autres). Remplace tes règles Firestore actuelles par :

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /members/{email} {
      allow read: if request.auth != null;
      allow write: if false;
    }
    match /alerts/{alertId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null && request.resource.data.email == request.auth.token.email;
      allow update: if request.auth != null && resource.data.email == request.auth.token.email;
      allow delete: if request.auth != null && resource.data.email == request.auth.token.email;
    }
  }
}
```

(Va dans Firestore Database > onglet Rules, remplace tout, Publish.)

## Pour aller plus loin

Le site a maintenant 11 fichiers : `index.html`, `auth.html`, `dashboard.html`, `outil-signal.html`, `analyse-marche.html`, `calculateur.html`, `sessions.html`, `tarifs.html`, `faq.html`, `assistant.html`, plus les 2 fichiers de configuration (`firebase-config.js`, `twelvedata-config.js`). Tout le site utilise le même thème sombre bleu-violet, avec le lien vers ton canal Telegram visible partout.

Nouveaux outils ajoutés :
- **analyse-marche.html** — décrit la tendance, la volatilité et les zones clés d'un instrument en langage clair (mêmes données réelles que le générateur de niveaux).
- **calculateur.html** — calculateur de taille de position (pur calcul, aucune donnée externe, aucun conseil).
- **sessions.html** — horloge en direct des sessions de marché (Sydney, Tokyo, Londres, New York), calculée localement dans le navigateur.

L'assistant (`assistant.html`) répond à partir d'une base de connaissances intégrée au site, sans clé d'API exposée publiquement — c'est volontaire : une clé d'IA payante ne doit jamais être placée dans le code d'un site hébergé sur GitHub Pages, car ce code est entièrement public et n'importe qui pourrait l'utiliser à tes frais. Si tu veux un jour un assistant connecté à un vrai modèle d'IA (réponses plus flexibles), il faudra passer par un petit serveur intermédiaire qui garde la clé secrète — une étape supplémentaire qu'on pourra construire ensemble le moment venu.

Si tu veux que les alertes envoyées sur ton bot Telegram apparaissent aussi dans le tableau de bord, il faudra une base de données (Firestore, gratuite aussi) qui stocke chaque signal envoyé par ton bot Python, et que `dashboard.html` vienne lire. Dis-le-moi quand tu en es là, je peux le brancher.
