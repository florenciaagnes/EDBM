# EDBM — Plateforme de régulation et de gestion des entreprises (CodeIgniter 4 + SQLite)

Thème **Executive Premium** : bleu nuit `#14263D`, ivoire `#F7F4ED`, or champagne `#C6A76A` (variables CSS dans `public/assets/css/app.css`).

## Prérequis
PHP **8.2+** avec extensions `intl`, `mbstring`, `pdo_sqlite`/`sqlite3`, `curl`, `xml`, `dom`, `fileinfo` ; Composer 2. L’extension `zip` est recommandée pour exploiter directement les modèles DOCX et reste facultative grâce au modèle texte de secours.

## Installation
```bash
composer install                       # installe CodeIgniter et PHPUnit (vendor/ n'est pas livré)
cp env .env                            # puis adapter app.baseURL (déjà fourni avec des valeurs de développement)
php spark migrate                      # crée la base SQLite (writable/database/edbm.db)
php spark db:seed RolesPermissionsSeeder   # rôles et permissions (indispensable)
php spark db:seed ReferenceSeeder          # secteurs, formes juridiques (règles de DÉMO), types de pièces, modèle de statuts
php spark db:seed DemoDataSeeder           # (développement uniquement) comptes et dossiers fictifs
php spark serve                        # http://localhost:8080
```
Raccourci développement : `php spark migrate && php spark db:seed DemoDataSeeder` (appelle aussi les deux seeders de référence).

### Comptes de démonstration (mot de passe : `Demo@2026-Edbm`, modifiable via `edbm.demoPassword`)
| Compte | Rôle |
|---|---|
| superadmin@edbm.test | Super administrateur |
| admin@edbm.test | Administrateur |
| instructeur@edbm.test | Agent instructeur |
| validateur@edbm.test | Agent chargé de la validation |
| documents@edbm.test | Responsable des documents |
| gerant@alpha.test | Client « Entreprise » (société Alpha Élevage SARL) |
| gerant@beta.test | Client « Entreprise » (société Beta Numérique SAS) |
| actionnaire@alpha.test | Client « Actionnaire » (60 % d'Alpha Élevage) |

## Tests
```bash
composer test            # = phpunit --no-coverage ; base SQLite en mémoire, migrations rejouées à chaque test
```
Couverture fonctionnelle : machine à états, règles par forme juridique, authentification (blocage, jetons), isolation entre sociétés, workflow actionnaire/entreprise, historique des mandats, recherche multi-critères, préférences et livraisons de notifications, documents (modèles par forme juridique, validation, génération, finalisation, immuabilité, intégrité, téléversement), collaboration modérée (profils vérifiés, opportunités, demandes et signalements), tableau de bord, audit, CSRF, rendu des pages.

### Collaboration entre entreprises
Les comptes entreprise propriétaires ou gestionnaires peuvent soumettre un profil de compétences à vérification, puis proposer des opportunités. Une annonce reste invisible tant qu’un rôle doté de `collaboration.moderate` ne l’a pas publiée. Les demandes de contact sont historisées et notifiées dans le centre interne ; le nom de l’entreprise demandeuse et ses coordonnées professionnelles ne sont révélés à l’entreprise hôte qu’après acceptation. Les annonces peuvent être désactivées ou signalées ; la modération peut classer un signalement ou suspendre l’annonce. Aucun dossier administratif ni contact personnel privé n’est partagé. Appliquer les migrations à jour et relancer `RolesPermissionsSeeder` après déploiement.

## Environnements
- **Développement** : `CI_ENVIRONMENT = development` (erreurs détaillées, lien de réinitialisation affiché à l'écran car aucun e-mail n'est envoyé).
- **Production** : voir `.env.production.example` et `docs/SECURITY_AND_SQLITE.md`.
- **E-mail** : en mode par défaut `edbm.emailMode = log`, aucun e-mail externe n’est envoyé et la livraison reste en attente. Pour activer SMTP, configurer dans `.env` `edbm.emailMode = smtp`, `edbm.smtpFrom`, puis les paramètres `email.protocol`, `email.SMTPHost`, `email.SMTPUser`, `email.SMTPPass`, `email.SMTPPort` et `email.SMTPCrypto`. Les noms d’environnement `EDBM_EMAIL_MODE`, `EDBM_SMTP_FROM`, `EMAIL_PROTOCOL`, `EMAIL_SMTP_HOST`, `EMAIL_SMTP_USER`, `EMAIL_SMTP_PASS`, `EMAIL_SMTP_PORT` et `EMAIL_SMTP_CRYPTO` sont également reconnus. Ne jamais inscrire un mot de passe SMTP dans le code ni dans un fichier versionné ; utilisez un mot de passe d’application dédié et révoquez tout secret exposé.
- **DOCX** : l’extension PHP `zip` permet d’extraire la mise en forme des modèles. Si elle est absente, la génération bascule sur le modèle texte de secours ; activer l’extension dans la configuration PHP utilisée par le serveur.

## Commandes utiles
| Commande | Rôle |
|---|---|
| `php spark migrate` / `migrate:rollback` / `migrate:status` | migrations (réversibles) |
| `php spark edbm:backup --dir D --keep N` | sauvegarde cohérente base + documents |
| `php spark edbm:restore fichier.db` | restauration contrôlée |
| `php spark notifications:retry [limit]` | relance les livraisons externes échouées arrivées à échéance (1–500, défaut 100) |
| `php spark routes` | liste des routes (voir `docs/ROUTES.md`) |

## Documentation
`docs/AUDIT.md` (état initial et décisions) · `docs/ARCHITECTURE.md` · `docs/LEGAL_RULES.md` (règles juridiques configurables — **à valider**) · `docs/SECURITY_AND_SQLITE.md` · `docs/ROUTES.md`.

## Dépendances externes (aucune n'est payante ni obligatoire)
- **Leaflet 1.9.4** chargé depuis `cdnjs.cloudflare.com` et **tuiles OpenStreetMap** (`tile.openstreetmap.org`) pour la carte du siège. Les tuiles OSM sont soumises à la *Tile Usage Policy* (usage modéré, attribution obligatoire, pas pour de fortes charges) : pour la production, prévoir un fournisseur de tuiles adapté (compte/clé à placer dans `.env`, jamais dans le code). **Si le service est injoignable**, la fiche reste utilisable : un message de repli est affiché et les coordonnées se saisissent à la main.
- Aucune police ni bibliothèque JavaScript externe ; PDF généré par une classe interne (`SimplePdf`, texte et tracé de signature manuscrite non certifiée).
