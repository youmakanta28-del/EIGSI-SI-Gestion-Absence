# Gestion des absences étudiantes

Application de suivi des absences, de dépôt et de décision sur les justificatifs, ainsi que de suivi semestriel et d’alertes de seuil.

Le navigateur communique avec le serveur Express ; il n’accède jamais directement à la base SQLite ni aux fichiers privés.

## Prérequis

- Node.js LTS (version 20 ou supérieure recommandée)
- npm / npm.cmd
- PowerShell sur Windows, car l’environnement peut bloquer les scripts npm par défaut

## Installation depuis zéro

Depuis la racine du projet :

```powershell
Set-Location "c:\chemin\vers\Mon Projet"

Set-Location server
npm.cmd install

Set-Location ..\client
npm.cmd install
```

## Lancement en développement

Ouvrez deux terminaux PowerShell distincts.

### Terminal 1 — API Express + SQLite

```powershell
Set-Location "c:\chemin\vers\Mon Projet\server"
npm.cmd run dev
```

Le serveur démarre sur :
- http://127.0.0.1:3000

### Terminal 2 — interface React

```powershell
Set-Location "c:\chemin\vers\Mon Projet\client"
npm.cmd run dev
```

L’interface est disponible sur :
- http://127.0.0.1:5173

## Comptes de démonstration

Les comptes localement activés sont :

- Admin : nadia.admin@example.test / Demo-2026!
- Professeur : samir.professeur@example.test / Demo-2026!
- Étudiant : amina.etudiante@example.test / Demo-2026!

## Vérification du build front

Depuis le dossier client :

```powershell
npm.cmd run build
```

## Préparer le déploiement Render (application pour l’école)

Le fichier `render.yaml` décrit un serveur web Node, un disque persistant monté dans `/var/data` et le build React. Il sert de configuration ; il ne publie pas le projet tout seul. La création du service Render demande un dépôt Git distant et un compte Render.

### Avant de publier

- Vérifier le tarif Render affiché pour le service et le disque persistant. La formule `starter` et le disque peuvent être payants ; ne pas confirmer la création si le prix n’est pas accepté.
- Ne pas committer la base SQLite, les justificatifs, les exports étudiants, les mots de passe, les clés ou les valeurs secrètes.
- Ne pas importer de vraies données scolaires avant d’avoir vérifié les règles d’accès, les personnes autorisées et les obligations de l’école.
- L’application désactive automatiquement les comptes `@example.test` en production. L’import de démonstration ne doit donc pas servir de comptes réels.

### Mise en service

1. Envoyer le code au dépôt Git privé choisi par l’école, après vérification de `.gitignore`.
2. Dans Render, créer un service à partir du dépôt et sélectionner le blueprint `render.yaml`. Examiner le prix avant de confirmer.
3. Après création, récupérer l’adresse HTTPS du service. Dans les variables du service, renseigner `PUBLIC_URL` avec cette adresse exacte, par exemple `https://nom-du-service.onrender.com`.
4. Le blueprint génère `ADMIN_SETUP_SECRET`. Le copier depuis les variables privées Render et le transmettre uniquement au premier administrateur par un canal sûr. Ne pas le mettre dans un document, le dépôt ou une capture d’écran.
5. Ouvrir l’adresse HTTPS : le premier écran demande le secret, le nom, l’adresse e-mail et un mot de passe administrateur d’au moins 12 caractères. La route initiale cesse de fonctionner dès qu’un administrateur a été créé.
6. Sur le serveur, laisser `ENABLE_NOTIFICATION_SCHEDULER=false` jusqu’à la configuration et au test SMTP. Pour activer les alertes, définir séparément les variables SMTP (`SMTP_HOST`, `SMTP_PORT`, `SMTP_FROM` et, si nécessaire, `SMTP_USER`, `SMTP_PASS`, `SMTP_SECURE`) puis activer le planificateur.
7. Se connecter comme administrateur et importer les CSV préparés par l’école depuis **Paramètres → Importer les données de l’école**. Commencer par les professeurs, étudiants et cours/séances. Contrôler le résultat et les liens d’activation avant de les transmettre individuellement.

Les étudiants et professeurs invités définissent eux-mêmes leur mot de passe via un lien unique valable 7 jours. Le serveur affiche ces liens une seule fois à l’administrateur après l’import ; les liens sont des données sensibles et ne doivent pas être partagés entre plusieurs personnes.

### Formats des fichiers CSV

Des gabarits sans données personnelles sont fournis dans `server/import-templates/`.

- `teachers.csv` : `email,first_name,last_name`
- `students.csv` : `email,first_name,last_name,student_number,program,level,group,school_year,semester_number,starts_on,ends_on`
- `courses.csv` : `code,title,program,level,group,school_year,semester_number,starts_on,ends_on,planned_hours,teacher_email,session_starts_at,session_ends_at`

Le fichier `courses.csv` contient une ligne par séance planifiée. Les dates de semestre sont `AAAA-MM-JJ` et les horaires de séance `AAAA-MM-JJ HH:MM:SS`. Les échecs de validation annulent l’import complet afin d’éviter un chargement partiel.

### Données conservées

- Base SQLite : `/var/data/absences.sqlite`
- Justificatifs privés : `/var/data/private_uploads/`

Ces emplacements doivent rester sur le disque persistant monté par Render et ne sont jamais servis comme fichiers statiques. Conserver aussi une procédure de sauvegarde et de restauration hors du serveur.

### Vérifications après déploiement

- L’adresse HTTPS ouvre l’interface sur téléphone et ordinateur.
- L’API `/api/hello` répond pour le contrôle de santé.
- Un administrateur peut se connecter et importer les fichiers d’essai fictifs.
- Les étudiants ne voient que leur dossier ; les professeurs ne voient que leurs groupes ; seul l’administrateur décide des justificatifs.
- Un justificatif envoyé n’est pas accessible sans session autorisée.
- Les données existent encore après redémarrage du service.
- Le SMTP est en mode aperçu tant qu’il n’est pas configuré et testé.

Le déploiement public, la création du service Render, la publication du dépôt et tout coût associé nécessitent une approbation séparée. Le fichier `render.yaml` ne déclenche aucune de ces actions.

Vérification attendue : build Vite réussi sans erreur.

## Données privées

- Base SQLite locale : server/data/absences.sqlite
- Documents justificatifs : server/private_uploads/

Ces dossiers sont privés et ne doivent pas être exposés publiquement. Ils sont exclus du dépôt Git.

## Bilan du MVP

### Fonctionnalités terminées

- authentification locale avec cookie de session
- rôles et accès restreints : étudiant, professeur, administration
- enregistrement des absences par séance pour le professeur
- impossibilité d’enregistrer deux absences identiques pour la même séance
- dépôt de justificatif par l’étudiant avec fichier privé
- validation / refus par l’administration
- historique des décisions et suivi d’état
- tableau de bord administratif
- rapports semestriels et export CSV
- liste des notifications / alertes
- paramétrage des seuils et fréquence d’alerte
- démarrage système sans crash sur les données existantes

### Fonctionnalités restantes ou à clarifier

- validation complète du parcours navigateur pour chaque rôle
- config SMTP réelle pour l’envoi de courriels de production
- choix définitif du mécanisme de relance automatique et du calendrier exact des alertes
- mise en place d’un vrai environnement de production et de données non démonstration

### Bugs connus

- Les courriels ne sont pas réellement envoyés sans configuration SMTP ; le système reste en mode aperçu local.
- La validation utilisateur complète est encore à faire dans un navigateur réel pour vérifier les clics et le parcours complet.
- Les données de démonstration sont destinées au développement local, pas encore à une utilisation de production.

## Scénario de démonstration — 3 minutes

### 1. Connexion et rôle

1. Ouvrir http://127.0.0.1:5173
2. Se connecter avec l’admin : nadia.admin@example.test / Demo-2026!
3. Vérifier le tableau de bord et les statistiques du semestre

### 2. Parcours professeur

1. Se déconnecter
2. Se connecter avec samir.professeur@example.test / Demo-2026!
3. Cliquer sur Mes cours
4. Sélectionner une séance
5. Cocher un ou plusieurs étudiants absents
6. Enregistrer l’appel
7. Vérifier que l’absence apparaît dans le fichier historique

### 3. Parcours étudiant

1. Se déconnecter
2. Se connecter avec amina.etudiante@example.test / Demo-2026!
3. Ouvrir Mes absences
4. Choisir une absence non justifiée
5. Ajouter un justificatif PDF/JPG/PNG
6. Valider le dépôt
7. Vérifier l’état du document dans Mes justificatifs

### 4. Décision administration

1. Se déconnecter
2. Se connecter avec l’admin
3. Ouvrir Justificatifs
4. Valider ou refuser un justificatif
5. Ajouter un commentaire si nécessaire
6. Vérifier que le statut de l’étudiant est mis à jour

### 5. Vérification finale

- Ouvrir l’onglet Notifications côté étudiant
- Vérifier la notification de décision
- Ouvrir le menu Rapports pour contrôler le bilan semestriel
- Vérifier que le front et l’API fonctionnent sans erreur sur le terminal

## Commandes utiles

```powershell
# Démarrer le backend
Set-Location "c:\chemin\vers\Mon Projet\server"
npm.cmd run dev

# Démarrer le frontend
Set-Location "c:\chemin\vers\Mon Projet\client"
npm.cmd run dev

# Construire le frontend en production
Set-Location "c:\chemin\vers\Mon Projet\client"
npm.cmd run build
```