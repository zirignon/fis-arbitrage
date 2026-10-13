# Inscriptions FIS : contexte du projet

Site d'inscriptions en ligne de la **Fédération Ivoirienne de Scrabble (FIS)** : un seul site pour tous les tournois et formations, avec un lien par tournoi. Publié sur **https://inscriptions-fis.ci** (GitHub Pages, fichier `CNAME`). Interface en **français**, public majoritairement sur mobile.

Compétitions gérées : Formation nationale des arbitres, Championnat National de Duplicate (CND), Master de Bouaké, Championnat National de Classique (CNC), Interclub de Duplicate, Interclub Classique. Elles sont créées verrouillées, puis ouvertes une par une quand leurs dates sont fixées.

## Stack

- **Un seul fichier `index.html`** (~860 lignes) : HTML, CSS et JavaScript vanilla, sans build ni npm. Pas d'autre fichier JS ou CSS.
- **Supabase** via CDN (`@supabase/supabase-js@2`). Projet **dédié** `lfmhytxqglujotzpvlku`, à ne jamais confondre avec celui d'Ivoire Électroménagers.
- Le client s'appelle **`supabaseClient`**. La clé dans le code est la clé **publishable** (publique) : c'est normal, la sécurité repose sur la RLS.
- **SheetJS** (`xlsx@0.18.5`, CDN) pour l'export Excel des inscriptions.
- Polices Google Fonts : Fraunces (titres) et Inter (texte). Couleurs en variables CSS dans `:root` (navy, gold, cream…).
- Logo FIS **embarqué en base64** dans le HTML : ne pas l'afficher ni le recopier en entier quand tu lis ou modifies le fichier.
- `nojekyll` est présent pour GitHub Pages.

## Fonctionnement

Tout le JS est dans une IIFE en bas de `index.html`.

- **Sans paramètre** : page d'accueil listant les tournois où `active = true`.
- **`?t=<slug>`** : formulaire du tournoi. Il est construit dynamiquement à partir de la ligne `fis_tournaments` :
  - `title`, `dates_label`, `category_label`, `category_options` (cases à cocher) ;
  - `followup_question` et `followup_detail_label` (question complémentaire facultative) ;
  - `engagement_text` ;
  - champs affichés selon les drapeaux `ask_birthdate`, `ask_ville`, `ask_license`, `ask_ligue` ;
  - fenêtre `registration_open` / `registration_close` (« pas encore ouvertes » ou « closes »).
- **Envoi** : `insert` dans `fis_registrations` (`tournament_slug`, `nom`, `prenom`, `ville`, `telephone`, `email`, `club`, `date_naissance`, `numero_licence`, `ligue`, `categories`, `followup_answer`, `followup_detail`). Une erreur `42501` signifie que la RLS a refusé l'insertion parce que le tournoi est hors période.
- **Administration** : bouton « Voir les inscriptions enregistrées » → connexion Supabase Auth (e-mail + mot de passe) → tableau filtrable par tournoi et export `.xlsx`. Ont accès : Ahissé, le DTN et le chargé de communication de la FIS.

### Règles du formulaire (décidées, à conserver)

- Obligatoires : nom, prénom, téléphone, club, au moins une catégorie, case d'engagement ; ville et ligue si le tournoi les demande.
- **Facultatifs** : numéro de licence et e-mail.
- Les tournois demandent licence et ligue **à la place de** la date de naissance et de la ville ; c'est réglé tournoi par tournoi avec les drapeaux `ask_*`.
- La ligue est une liste déroulante à deux choix : **Abidjan** et **Bouaké**.

## Règles pour modifier le site

1. **Ajouter ou ouvrir un tournoi = des données, pas du code.** On insère ou modifie une ligne de `fis_tournaments` (SQL fourni à Ahissé, à exécuter dans l'éditeur SQL du projet `lfmhytxqglujotzpvlku`). Ne pas coder un tournoi en dur dans le HTML.
2. **Sécurité côté serveur** : la période d'inscription et la lecture des inscriptions sont protégées par la **RLS** Supabase. Masquer un bouton ou vérifier une date en JavaScript ne protège rien. Toute nouvelle table ou colonne sensible doit avoir sa politique RLS. Seuls les comptes admin doivent pouvoir lire `fis_registrations` (téléphones, e-mails…).
3. **XSS** : tout texte venant de la base ou d'un utilisateur passe par `escapeHtml()` avant d'aller dans `innerHTML`.
4. **Ne jamais réécrire `index.html` de zéro** ni repartir d'une ancienne copie : modifier par petites retouches ciblées, puis vérifier que l'accueil, le formulaire, la fenêtre d'inscription, l'admin et l'export Excel sont toujours là.
5. Garder le site **léger et mobile d'abord** (pas de framework, pas de nouvelle bibliothèque sans raison).
6. Ne pas ajouter de nouveau champ obligatoire sans l'accord d'Ahissé : cela change ce que les joueurs doivent fournir.

## Vérifications avant de livrer

- Extraire le script et vérifier la syntaxe (`node --check` sur le JS extrait).
- Tester mentalement ou dans le navigateur : accueil sans `?t=`, un tournoi ouvert, un tournoi pas encore ouvert, un tournoi clos, un slug inconnu, la connexion admin, le filtre par tournoi, l'export Excel.

## Environnement d'Ahissé

- Windows 10, **cmd.exe** (pas PowerShell) : les commandes qui lui sont données doivent être compatibles cmd.
- Mise en ligne par **glisser-déposer sur github.com** (compte `zirignon`, dépôt `fis-arbitrage`) ; GitHub Pages publie automatiquement. Livrer le fichier `index.html` complet final quand il doit être déposé.
