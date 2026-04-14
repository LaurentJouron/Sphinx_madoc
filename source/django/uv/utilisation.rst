.. _Utilisation:

===========
Utilisation
===========

Démarrage rapide
================

Le profil PowerShell se charge automatiquement à chaque ouverture d'un terminal. Aucune action n'est nécessaire pour l'activer.

Pour créer un nouveau projet Django complet en une seule commande :

.. code-block:: powershell

   uvdjpro mon_projet

Le projet est créé dans ``.\mon_projet\``, les dépendances sont installées, les migrations sont appliquées et VS Code s'ouvre automatiquement.

----

Créer un nouveau projet — ``uvdjpro``
======================================

.. code-block:: powershell

   uvdjpro -project <nom_du_projet>

``<nom_du_projet>`` doit être en ``snake_case``. Les espaces ne sont pas acceptés.

**Exemples :**

.. code-block:: powershell

   uvdjpro mon_blog
   uvdjpro defilepsie
   uvdjpro gestion_stock

Ce que fait la commande
-----------------------

En quelques minutes, ``uvdjpro`` crée un projet Django complet prêt à être développé :

- Dossier du projet, environnement virtuel Python (via ``uv``)
- Toutes les dépendances installées (Django, allauth, Tailwind, HTMX, Celery, DRF…)
- Settings Django organisés par environnement (``dev.py``, ``prod.py``, ``test.py``)
- Application ``users`` avec un modèle ``User`` personnalisé (avatar, bio)
- Application ``maintenance`` avec les pages d'erreur 400, 403, 404, 500
- Authentification complète (email, Google, Facebook) via django-allauth
- Templates de login, signup, déconnexion et réinitialisation de mot de passe
- Pages de maintenance et d'erreur stylisées avec animations
- Configuration Tailwind CSS, AlpineJS et HTMX
- Fichier ``.env`` avec des clés secrètes générées aléatoirement
- Dockerfile et ``docker-compose.dev.yml`` / ``docker-compose.prod.yml``
- Nginx configuré pour Cloudflare
- Pipeline CI/CD GitHub Actions vers Synology
- Makefile avec les commandes courantes
- README complet

Une fois terminé, le terminal affiche les prochaines étapes :

.. code-block:: text

   1. uv run python manage.py createsuperuser
   2. make tailwind   ← terminal dédié
   3. make dev        ← autre terminal
   4. Admin Django → Sites → ajouter 'mon-projet.laurentjouron.dev'
   5. Renseigner .env : Google/Facebook OAuth + SMTP

.. tip::
   Les migrations sont exécutées automatiquement par ``uvdjpro``. La base SQLite est prête à l'emploi dès la fin de la génération.

Personnalisations à faire après génération
-------------------------------------------

.. list-table::
   :widths: 40 60
   :header-rows: 1

   * - Fichier / Action
     - Ce à quoi il faut penser
   * - ``.env``
     - Renseigner ``GOOGLE_CLIENT_ID``, ``GOOGLE_CLIENT_SECRET``, ``FACEBOOK_APP_ID``, ``FACEBOOK_APP_SECRET``, ``EMAIL_HOST_USER``, ``EMAIL_HOST_PASSWORD``
   * - ``static/images/logo_full.png``
     - Remplacer le placeholder par le vrai logo du projet (affiché dans les pages maintenance)
   * - ``templates/maintenance/maintenance.html``
     - Remplacer ``contact@example.com`` par l'adresse réelle
   * - ``templates/maintenance/errors/403.html``
     - Idem pour le bouton de contact
   * - Admin Django → Sites
     - Mettre à jour le domaine (requis par allauth pour les emails de confirmation)

----

Lancer le serveur de développement
====================================

.. code-block:: powershell

   djrun

Equivalent à ``uv run python manage.py runserver``.

Pour le développement avec rechargement automatique de Tailwind, ouvrir **deux terminaux** :

.. code-block:: powershell

   # Terminal 1 — compilation CSS en temps réel
   make tailwind
   # ou directement :
   djtw

   # Terminal 2 — serveur Django
   djrun

.. note::
   Sans ``make tailwind``, les classes Tailwind dans les templates ne seront pas compilées et la mise en page de l'interface principale n'apparaîtra pas. Les pages de maintenance et d'authentification fonctionnent toutefois correctement car elles utilisent un CSS autonome.

----

Commandes Git
=============

Ces raccourcis remplacent les commandes Git les plus fréquentes.

.. list-table::
   :widths: 25 75
   :header-rows: 1

   * - Commande
     - Description
   * - ``gst``
     - Affiche le statut des fichiers (``git status``)
   * - ``gaa``
     - Ajoute tous les fichiers modifiés au prochain commit (``git add .``)
   * - ``gco "mon message"``
     - Crée un commit avec le message donné
   * - ``gpu``
     - Pousse les commits vers le dépôt distant (``git push``)
   * - ``gll``
     - Affiche l'historique des commits sous forme de graphe condensé
   * - ``gundo``
     - Annule le dernier commit **sans** supprimer les modifications (les fichiers restent modifiés)

**Exemple de workflow Git typique :**

.. code-block:: powershell

   gst                      # Voir ce qui a changé
   gaa                      # Tout stager
   gco "Ajout de la vue liste"   # Commiter
   gpu                      # Pousser

----

Commandes uv
============

.. list-table::
   :widths: 25 75
   :header-rows: 1

   * - Commande
     - Description
   * - ``uvsh``
     - Active l'environnement virtuel (équivalent à ``source .venv/bin/activate``)
   * - ``uvadd <paquet>``
     - Installe un paquet et met à jour ``pyproject.toml``
   * - ``uvserve``
     - Lance le serveur Django
   * - ``uvpy``
     - Ouvre l'interpréteur Python dans l'environnement virtuel

**Exemple :**

.. code-block:: powershell

   uvadd django-import-export
   uvadd "celery[redis]"

----

Commandes Django
================

.. list-table::
   :widths: 25 75
   :header-rows: 1

   * - Commande
     - Description
   * - ``djrun``
     - Lance le serveur de développement Django
   * - ``djmig``
     - Applique les migrations en attente (``migrate``)
   * - ``djmakemig``
     - Génère les fichiers de migration pour les modèles modifiés (``makemigrations``)
   * - ``djsh``
     - Ouvre le shell Django interactif
   * - ``djtest``
     - Lance la suite de tests
   * - ``djadmin``
     - Crée un superutilisateur pour l'admin Django
   * - ``djtw``
     - Lance la compilation Tailwind CSS en mode watch (à garder dans un terminal dédié)
   * - ``djclean``
     - Supprime tous les fichiers ``__pycache__`` et ``*.pyc`` du projet

**Cycle de développement classique après modification d'un modèle :**

.. code-block:: powershell

   djmakemig     # Génère la migration
   djmig         # Applique la migration
   djrun         # Relance le serveur

----

Ajouter une nouvelle application — ``djapp_pro``
=================================================

.. code-block:: powershell

   djapp_pro -name <nom_app>

Crée une application Django dans ``apps/<nom_app>/`` avec une structure complète (vues, URLs, template de base).

**Exemple :**

.. code-block:: powershell

   djapp_pro -name blog

Génère :

.. code-block:: text

   apps/blog/
   ├── urls.py          ← route racine "/" → vue index
   ├── views.py         ← vue index
   ├── templates/
   │   └── blog/
   │       └── index.html
   └── static/
       └── blog/

Après génération, deux étapes manuelles sont nécessaires :

**1. Ajouter l'app à ``config/settings/base.py`` :**

.. code-block:: python

   LOCAL_APPS = [
       "users",
       "maintenance",
       "blog",   # ← ajouter ici
   ]

**2. Ajouter le chemin dans ``config/urls.py`` :**

.. code-block:: python

   path("blog/", include("blog.urls", namespace="blog")),

----

Ajouter un modèle — ``djmodel``
================================

.. code-block:: powershell

   djmodel -app <nom_app> -model <nom_modele> [-fields <champs>]

Ajoute une classe de modèle Django dans le fichier ``models.py`` existant de l'application.

**Exemple sans champs :**

.. code-block:: powershell

   djmodel -app blog -model Article

Génère dans ``apps/blog/models.py`` :

.. code-block:: python

   class Article(models.Model):
       # fields here

       def __str__(self):
           return f"Article({self.pk})"

**Exemple avec champs :**

.. code-block:: powershell

   djmodel -app blog -model Article -fields "title = models.CharField(max_length=200),body = models.TextField(),published = models.BooleanField(default=False)"

Génère :

.. code-block:: python

   class Article(models.Model):
       title = models.CharField(max_length=200)
       body = models.TextField()
       published = models.BooleanField(default=False)

       def __str__(self):
           return f"Article({self.pk})"

Après avoir ajouté un modèle, penser à migrer :

.. code-block:: powershell

   djmakemig
   djmig

----

Ajouter une vue — ``djview``
=============================

.. code-block:: powershell

   djview -app <nom_app> -view <nom_vue> [-urlPath <chemin>] [-templateName <template>]

Ajoute une vue (fonction), une entrée URL et un template HTML dans une application existante.

**Exemple minimal :**

.. code-block:: powershell

   djview -app blog -view ArticleList

Crée :

- une fonction ``articleList`` dans ``apps/blog/views.py``
- un path ``articlelist/`` dans ``apps/blog/urls.py``
- un template ``apps/blog/templates/blog/ArticleList.html``

**Exemple avec options :**

.. code-block:: powershell

   djview -app blog -view ArticleList -urlPath "articles" -templateName "blog/article_list.html"

Crée :

- un path ``articles/`` dans ``urls.py``
- un template au chemin ``blog/article_list.html``

.. note::
   ``djview`` génère des vues sous forme de fonctions (``def``). Pour des vues basées sur des classes, utiliser ``djview`` comme point de départ puis réécrire manuellement la vue générée.

----

Générer un CRUD complet — ``djcrud``
=====================================

.. code-block:: powershell

   djcrud -app <nom_app> -model <nom_modele> [-fields <champs>]

Génère l'intégralité des fichiers pour gérer un modèle : liste, détail, création, modification, suppression.

**Exemple :**

.. code-block:: powershell

   djcrud -app blog -model Article -fields "title = models.CharField(max_length=200),body = models.TextField()"

Génère en une seule commande :

- Le modèle ``Article`` dans ``models.py``
- Le formulaire ``ArticleForm`` dans ``forms.py``
- 5 vues dans ``views.py`` : liste, détail, création, modification, suppression
- 5 routes dans ``urls.py``
- 4 templates : liste, détail, formulaire, confirmation de suppression

Les URLs créées sont accessibles avec le namespace de l'app :

.. code-block:: text

   /blog/article/                    → liste de tous les articles
   /blog/article/<pk>/               → détail d'un article
   /blog/article/create/             → créer un article
   /blog/article/<pk>/update/        → modifier un article
   /blog/article/<pk>/delete/        → supprimer un article

Après génération, penser à migrer si des champs ont été ajoutés :

.. code-block:: powershell

   djmakemig
   djmig

----

Créer une API REST — ``djapi``
================================

.. code-block:: powershell

   djapi -name <nom_app>

Crée une application Django REST Framework avec la structure de base (serializer, viewset, router).

**Exemple :**

.. code-block:: powershell

   djapi -name api_v1

Génère dans ``apps/api_v1/`` :

- ``serializers.py`` — ``ExampleSerializer``
- ``views.py`` — ``ExampleViewSet``
- ``urls.py`` — ``DefaultRouter`` avec l'endpoint ``/examples/``

Les fichiers utilisent ``Example`` comme modèle placeholder. Après génération :

1. Remplacer ``Example`` par le vrai modèle
2. Ajouter l'app à ``LOCAL_APPS``
3. Déclarer le chemin dans ``config/urls.py``

**Exemple d'utilisation typique :**

.. code-block:: powershell

   djapi -name api_v1
   djmodel -app api_v1 -model Article -fields "title = models.CharField(max_length=200)"
   # Puis éditer api_v1/serializers.py et api_v1/views.py pour remplacer Example par Article
   djmakemig
   djmig

----

Supprimer les fichiers compilés — ``djclean``
=============================================

.. code-block:: powershell

   djclean

Supprime récursivement tous les dossiers ``__pycache__`` et fichiers ``.pyc`` dans le projet courant. Utile avant un commit, un déploiement ou pour résoudre des problèmes d'import.

----

Commandes Makefile
==================

Le ``Makefile`` généré par ``uvdjpro`` regroupe les opérations courantes :

.. list-table::
   :widths: 30 70
   :header-rows: 1

   * - Commande
     - Description
   * - ``make dev``
     - Lance le serveur Django (identique à ``djrun``)
   * - ``make tailwind``
     - Lance le watcher Tailwind CSS (identique à ``djtw``)
   * - ``make migrate``
     - Applique les migrations
   * - ``make makemig``
     - Génère les migrations
   * - ``make superuser``
     - Crée un superutilisateur
   * - ``make collect``
     - Collecte les fichiers statiques (pour la prod)
   * - ``make test``
     - Lance les tests
   * - ``make shell``
     - Ouvre le shell Django
   * - ``make docker-dev``
     - Lance Docker Compose en mode développement
   * - ``make docker-prod``
     - Lance Docker Compose en mode production

----

Déploiement
===========

Déploiement en production via Docker
-------------------------------------

Le déploiement se fait via GitHub Actions au push sur la branche ``main``. Le pipeline :

1. Build l'image Docker
2. Push vers GitHub Container Registry (``ghcr.io/laurentjouron/<projet>:latest``)
3. Se connecte en SSH au NAS Synology
4. Exécute ``docker compose pull && docker compose up -d``

Pour le premier déploiement manuel :

.. code-block:: bash

   # Sur le NAS Synology
   mkdir -p /volume1/docker/<projet_kebab>
   cd /volume1/docker/<projet_kebab>
   # Copier docker-compose.prod.yml et .env
   docker compose up -d

Variables d'environnement pour la production
---------------------------------------------

Compléter le fichier ``.env`` avant le premier déploiement :

.. code-block:: ini

   # Authentification sociale
   GOOGLE_CLIENT_ID=<depuis console.cloud.google.com>
   GOOGLE_CLIENT_SECRET=<depuis console.cloud.google.com>
   FACEBOOK_APP_ID=<depuis developers.facebook.com>
   FACEBOOK_APP_SECRET=<depuis developers.facebook.com>

   # Email (exemple Mailgun)
   EMAIL_HOST_USER=postmaster@mg.mondomaine.com
   EMAIL_HOST_PASSWORD=<clé API Mailgun>

Configurer l'authentification sociale
--------------------------------------

**Google OAuth** :

1. Aller sur `console.cloud.google.com <https://console.cloud.google.com>`_
2. Créer un projet → APIs & Services → Identifiants → Créer des identifiants OAuth 2.0
3. URI de redirection autorisée : ``https://mondomaine.com/accounts/google/login/callback/``
4. Copier ``client_id`` et ``secret`` dans ``.env``

**Facebook Login** :

1. Aller sur `developers.facebook.com <https://developers.facebook.com>`_
2. Créer une app → Facebook Login → Paramètres
3. URI de redirection OAuth valide : ``https://mondomaine.com/accounts/facebook/login/callback/``
4. Copier ``App ID`` et ``App Secret`` dans ``.env``

**Dans l'admin Django** (une fois le projet déployé) :

1. ``/admin/sites/site/`` → mettre à jour le domaine
2. ``/admin/socialaccount/socialapp/`` → créer une entrée pour Google et une pour Facebook en renseignant les clés depuis ``.env``

----

Workflow complet — exemple de projet
======================================

Voici un exemple de création d'un blog de bout en bout :

.. code-block:: powershell

   # 1. Créer le projet
   uvdjpro mon_blog

   # 2. Ouvrir un terminal Tailwind (garder ouvert)
   djtw

   # 3. Créer l'app blog
   djapp_pro -name blog

   # 4. Ajouter 'blog' à LOCAL_APPS dans config/settings/base.py
   # Ajouter path("blog/", include("blog.urls", namespace="blog")) dans config/urls.py

   # 5. Générer le CRUD Article
   djcrud -app blog -model Article -fields "title = models.CharField(max_length=200),body = models.TextField(),author = models.ForeignKey('users.User' on_delete=models.CASCADE),created_at = models.DateTimeField(auto_now_add=True)"

   # 6. Migrer
   djmakemig
   djmig

   # 7. Créer le superuser
   djadmin

   # 8. Lancer le serveur
   djrun

   # → http://127.0.0.1:8000/blog/article/ est accessible

----

Résolution des problèmes courants
==================================

.. list-table::
   :widths: 40 60
   :header-rows: 1

   * - Problème
     - Solution
   * - ``No module named 'config.settings.dev'``
     - Vérifier que ``config/settings/`` est un dossier (pas un fichier) et qu'il contient ``__init__.py`` et ``dev.py``
   * - ``No module named 'users'``
     - Vérifier que ``apps/`` **ne contient pas** de fichier ``__init__.py``
   * - Les styles Tailwind n'apparaissent pas
     - Lancer ``make tailwind`` dans un terminal dédié. Les pages maintenance et auth fonctionnent sans cela.
   * - ``KeyError: 'tailwind'``
     - Ne pas appeler ``manage.py tailwind init`` pendant la génération du projet — l'app ``theme`` est créée manuellement
   * - ``ValueError: Dependency on app with no migrations: users``
     - Lancer ``djmakemig`` puis ``djmig`` pour créer les migrations du User custom
   * - ``uvsh`` ne fonctionne pas
     - Se placer à la racine du projet (là où se trouve ``.venv\``) avant d'appeler ``uvsh``
   * - Les pages allauth n'ont pas de style
     - Vérifier que ``static/css/layouts/auth-layout.css`` existe et que ``DEBUG=True`` ou ``whitenoise`` est configuré pour servir les fichiers statiques en prod.