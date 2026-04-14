.. _document_technique:

=======================
Documentation technique
=======================

Vue d'ensemble
==============

Le fichier ``Microsoft.PowerShell_profile.ps1`` est un profil PowerShell chargé automatiquement à chaque ouverture d'un terminal. Il constitue un environnement de développement complet et opinionné pour la création et la gestion de projets Django full-stack sous Windows.

**Stack cible**

- **Back-end** : Django, django-allauth, Django REST Framework, Celery, Channels
- **Front-end** : AlpineJS, HTMX, Tailwind CSS (via django-tailwind)
- **Base de données** : SQLite3 en développement local, PostgreSQL en production Docker
- **Gestionnaire de paquets** : ``uv`` (remplace pip/virtualenv)
- **Infrastructure** : Docker, Nginx, GitHub Actions, Synology DSM
- **Prompt** : Starship
- **Complétion** : PSReadLine

----

Installation et prérequis
==========================

Prérequis système
-----------------

Les outils suivants doivent être installés et disponibles dans le ``PATH`` avant de charger le profil :

.. list-table::
   :widths: 25 75
   :header-rows: 1

   * - Outil
     - Rôle
   * - ``uv``
     - Gestionnaire de paquets et d'environnements virtuels Python
   * - ``git``
     - Contrôle de version
   * - ``starship``
     - Prompt personnalisé (``starship init powershell``)
   * - ``code``
     - Visual Studio Code, appelé en fin de ``uvdjpro`` pour ouvrir le projet
   * - PowerShell 7+
     - Le profil utilise des fonctionnalités PS7 (``-replace``, ``Set-Content``)

Emplacement du profil
---------------------

Le fichier doit être placé au chemin retourné par la variable ``$PROFILE`` dans PowerShell :

.. code-block:: powershell

   $PROFILE
   # Exemple : C:\Users\<user>\Documents\PowerShell\Microsoft.PowerShell_profile.ps1

Pour recharger le profil sans redémarrer le terminal :

.. code-block:: powershell

   . $PROFILE

----

Section 1 — Préambule et helpers
=================================

Comportement global
-------------------

.. code-block:: powershell

   $ErrorActionPreference = "Stop"

Cette ligne configure PowerShell pour interrompre l'exécution dès qu'une erreur non gérée survient. Cela garantit qu'un échec intermédiaire dans ``uvdjpro`` (par exemple une installation de paquet) stoppe le script plutôt que de continuer dans un état incohérent.

Fonctions utilitaires internes
-------------------------------

Ces deux fonctions sont utilisées en interne par les générateurs. Elles ne sont pas destinées à être appelées directement.

``New-DirectorySafe``
~~~~~~~~~~~~~~~~~~~~~

.. code-block:: powershell

   function New-DirectorySafe {
       param([string]$Path)
       if (-not (Test-Path $Path)) {
           New-Item -ItemType Directory -Path $Path | Out-Null
       }
   }

Crée un répertoire uniquement s'il n'existe pas déjà. Équivalent de ``mkdir -p`` sous Unix. Utilisée massivement dans ``uvdjpro`` pour construire l'arborescence du projet.

``To-TitleCaseFromSnake``
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: powershell

   function To-TitleCaseFromSnake {
       param([string]$snake)
       ($snake.ToLower() -replace "_"," " -split " " |
           ForEach-Object { $_.Substring(0,1).ToUpper() + $_.Substring(1) }) -join " "
   }

Convertit un identifiant ``snake_case`` en ``Title Case`` lisible. Exemple : ``mon_projet`` → ``Mon Projet``. Utilisée dans ``uvdjpro`` pour générer le ``project_title`` affiché dans les templates HTML, le README et le manifest PWA.

----

Section 2 — Starship Prompt
============================

.. code-block:: powershell

   Invoke-Expression (&starship init powershell)

Initialise le prompt `Starship <https://starship.rs>`_ au démarrage du terminal. Starship affiche des informations contextuelles (branche Git, version Python, statut de l'environnement virtuel) dans l'invite de commande. Cette ligne est exécutée en dehors de toute fonction et s'applique donc à chaque session.

**Dépendance** : ``starship`` doit être installé et dans le ``PATH``. Si ce n'est pas le cas, l'ouverture du terminal produit une erreur. Pour désactiver temporairement, commenter la ligne.

----

Section 3 — PSReadLine
=======================

.. code-block:: powershell

   Import-Module PSReadLine
   Set-PSReadLineOption -PredictionSource History
   Set-PSReadLineOption -HistoryNoDuplicates
   Set-PSReadLineOption -HistorySaveStyle SaveIncrementally
   Set-PSReadLineKeyHandler -Key RightArrow -Function AcceptSuggestion

Configuration de l'auto-complétion et de l'historique :

.. list-table::
   :widths: 40 60
   :header-rows: 1

   * - Option
     - Effet
   * - ``PredictionSource History``
     - Suggère des commandes basées sur l'historique
   * - ``HistoryNoDuplicates``
     - Évite les doublons dans l'historique
   * - ``SaveIncrementally``
     - Sauvegarde l'historique après chaque commande (et non à la fermeture)
   * - ``RightArrow → AcceptSuggestion``
     - Valide la suggestion en cours avec la flèche droite

----

Section 4 — Alias Git
======================

Les alias Git sont implémentés comme des fonctions PowerShell (et non comme des ``Set-Alias``) pour permettre le passage d'arguments.

.. list-table::
   :widths: 20 35 45
   :header-rows: 1

   * - Fonction
     - Commande équivalente
     - Notes
   * - ``gst``
     - ``git status``
     -
   * - ``gaa``
     - ``git add .``
     - Ajoute tous les fichiers modifiés
   * - ``gco <msg>``
     - ``git commit -m <msg>``
     - Le message est un paramètre positionnel
   * - ``gpu``
     - ``git push``
     -
   * - ``gll``
     - ``git log --oneline --graph --decorate``
     - Vue condensée du graphe des commits
   * - ``gundo``
     - ``git reset --soft HEAD~1``
     - Annule le dernier commit sans supprimer les modifications

.. note::
   Les alias ``gco``, ``gpu``, ``gll`` étaient initialement définis avec ``Set-Alias``, ce qui provoquait un conflit avec les fonctions du même nom. La version actuelle utilise uniquement des fonctions.

----

Section 5 — Alias uv
=====================

.. list-table::
   :widths: 20 40 40
   :header-rows: 1

   * - Fonction
     - Commande équivalente
     - Notes
   * - ``uvsh``
     - ``.venv\Scripts\Activate.ps1``
     - Active l'environnement virtuel Windows
   * - ``uvadd <pkg>``
     - ``uv add <pkg>``
     - Installe un paquet et met à jour ``pyproject.toml``
   * - ``uvserve``
     - ``uv run python manage.py runserver``
     - Démarre le serveur Django via uv
   * - ``uvpy``
     - ``uv run python``
     - Lance l'interpréteur Python dans le venv courant

.. warning::
   ``uvsh`` utilise le chemin Windows ``\Scripts\Activate.ps1``. Cette commande ne fonctionne que si le dossier de travail courant est la racine du projet (là où se trouve ``.venv``).

----

Section 6 — Commandes Django
==============================

Ces fonctions sont des raccourcis pour les commandes ``manage.py`` les plus fréquentes. Elles utilisent toutes ``uv run`` pour s'assurer que Python est exécuté dans l'environnement virtuel correct sans activation préalable.

.. list-table::
   :widths: 20 40 40
   :header-rows: 1

   * - Fonction
     - Commande équivalente
     - Notes
   * - ``djrun``
     - ``uv run python manage.py runserver``
     -
   * - ``djmig``
     - ``uv run python manage.py migrate``
     -
   * - ``djmakemig``
     - ``uv run python manage.py makemigrations``
     -
   * - ``djsh``
     - ``uv run python manage.py shell``
     - Shell Django interactif
   * - ``djtest``
     - ``uv run python manage.py test``
     -
   * - ``djadmin``
     - ``uv run python manage.py createsuperuser``
     -
   * - ``djtw``
     - ``uv run python manage.py tailwind start``
     - Lance le watcher Tailwind CSS (django-tailwind)
   * - ``djclean``
     - Suppression récursive de ``__pycache__`` et ``*.pyc``
     - Utilise ``Get-ChildItem -Recurse``

----

Section 7 — ``djapp_pro``
==========================

Signature
---------

.. code-block:: powershell

   djapp_pro -name <string>

Description
-----------

Crée une application Django structurée dans le dossier ``apps/``. Appelle ``uv run python manage.py startapp <name> apps/<name>`` puis surcharge les fichiers générés avec une structure PRO.

Fichiers générés
----------------

.. code-block:: text

   apps/<name>/
   ├── (fichiers Django standard via startapp)
   ├── urls.py          ← url racine "" → views.index, app_name défini
   ├── views.py         ← vue index basique (fonction)
   ├── templates/
   │   └── <name>/
   │       └── index.html   ← extends layouts/main-layout.html
   └── static/
       └── <name>/

Comportement post-génération
------------------------------

Le script affiche deux rappels en jaune :

1. Ajouter ``'<name>'`` à ``LOCAL_APPS`` dans ``config/settings/base.py``
2. Ajouter ``path('<name>/', include('<name>.urls', namespace='<name>'))`` dans ``config/urls.py``

Ces deux étapes sont **manuelles** et volontairement laissées à la charge du développeur pour éviter des modifications automatiques de fichiers existants.

.. note::
   ``djapp_pro`` utilise ``manage.py startapp``, ce qui nécessite que Django soit correctement configuré (``config/settings/dev.py`` accessible et ``sys.path`` incluant ``apps/``). Elle ne doit donc être appelée qu'une fois le projet ``uvdjpro`` créé et fonctionnel.

----

Section 8 — ``djmodel``
========================

Signature
---------

.. code-block:: powershell

   djmodel -app <string> -model <string> [-fields <string>]

Description
-----------

Ajoute une classe de modèle Django dans ``apps/<app>/models.py`` en mode **append** (le fichier existant est conservé).

Paramètres
----------

.. list-table::
   :widths: 20 15 65
   :header-rows: 1

   * - Paramètre
     - Obligatoire
     - Description
   * - ``-app``
     - Oui
     - Nom de l'application cible (dossier dans ``apps/``)
   * - ``-model``
     - Oui
     - Nom du modèle en ``snake_case`` ou ``PascalCase`` (converti automatiquement en PascalCase)
   * - ``-fields``
     - Non
     - Chaîne de champs séparés par des virgules, insérés tels quels dans le corps du modèle

Transformation du nom
----------------------

Le premier caractère est mis en majuscule : ``article`` → ``Article``.

Format du paramètre ``-fields``
--------------------------------

Les champs sont séparés par des virgules et insérés avec une indentation de 4 espaces :

.. code-block:: powershell

   djmodel -app blog -model Article -fields "title = models.CharField(max_length=200),body = models.TextField()"

Produit :

.. code-block:: python

   class Article(models.Model):
       title = models.CharField(max_length=200)
       body = models.TextField()

       def __str__(self):
           return f"Article({self.pk})"

Si ``-fields`` est omis, le corps contient ``# fields here`` comme placeholder.

----

Section 9 — ``djview``
=======================

Signature
---------

.. code-block:: powershell

   djview -app <string> -view <string> [-urlPath <string>] [-templateName <string>]

Description
-----------

Génère simultanément une vue (fonction), une entrée URL et un template HTML pour une application existante. Tous les fichiers sont ouverts en mode **append** pour ``views.py`` et ``urls.py``.

Paramètres
----------

.. list-table::
   :widths: 20 15 65
   :header-rows: 1

   * - Paramètre
     - Obligatoire
     - Description
   * - ``-app``
     - Oui
     - Application cible
   * - ``-view``
     - Oui
     - Nom de la vue (converti : premier char en minuscule pour la fonction)
   * - ``-urlPath``
     - Non
     - Chemin URL. Par défaut : ``view.ToLower()``
   * - ``-templateName``
     - Non
     - Chemin du template. Par défaut : ``<app>/<view>.html``

Fichiers modifiés / créés
--------------------------

.. list-table::
   :widths: 35 65
   :header-rows: 1

   * - Fichier
     - Opération
   * - ``apps/<app>/views.py``
     - Append : fonction ``<viewFunc>(request)``
   * - ``apps/<app>/urls.py``
     - Append : ligne ``path(...)``
   * - ``apps/<app>/templates/<app>/<view>.html``
     - Création : template étendant ``layouts/main-layout.html``

----

Section 10 — ``djcrud``
========================

Signature
---------

.. code-block:: powershell

   djcrud -app <string> -model <string> [-fields <string>]

Description
-----------

Génère l'ensemble des fichiers nécessaires à un CRUD complet pour un modèle : modèle, formulaire, 5 vues (fonctions), 5 routes URL et 4 templates.

Fichiers générés
----------------

.. list-table::
   :widths: 35 65
   :header-rows: 1

   * - Fichier
     - Contenu
   * - ``apps/<app>/models.py``
     - Append : classe ``<Model>``
   * - ``apps/<app>/forms.py``
     - Création : ``<Model>Form(ModelForm)``
   * - ``apps/<app>/views.py``
     - Append : list, detail, create, update, delete
   * - ``apps/<app>/urls.py``
     - Append : 5 chemins URL
   * - ``apps/<app>/templates/<app>/<model>_list.html``
     - Liste avec boucle ``{% for %}``
   * - ``apps/<app>/templates/<app>/<model>_detail.html``
     - Détail avec boutons modifier / supprimer
   * - ``apps/<app>/templates/<app>/<model>_form.html``
     - Formulaire partagé create/update
   * - ``apps/<app>/templates/<app>/<model>_confirm_delete.html``
     - Confirmation de suppression

Routes générées
---------------

Les URLs utilisent le namespace de l'application. Exemple pour ``-model article`` dans ``-app blog`` :

.. code-block:: python

   path("article/",              views.article_list,   name="article_list"),
   path("article/<int:pk>/",     views.article_detail, name="article_detail"),
   path("article/create/",       views.article_create, name="article_create"),
   path("article/<int:pk>/update/", views.article_update, name="article_update"),
   path("article/<int:pk>/delete/", views.article_delete, name="article_delete"),

.. note::
   Les vues ``create`` et ``update`` partagent le même template ``<model>_form.html``. La vue ``update`` passe en plus l'objet ``object`` au contexte pour permettre la pré-remplissage.

----

Section 11 — ``djapi``
=======================

Signature
---------

.. code-block:: powershell

   djapi -name <string>

Description
-----------

Crée une application Django REST Framework. Installe ``djangorestframework`` via ``uv add``, crée l'application dans ``apps/<name>/`` et génère les fichiers DRF de base.

Fichiers générés
----------------

.. list-table::
   :widths: 35 65
   :header-rows: 1

   * - Fichier
     - Contenu
   * - ``apps/<name>/serializers.py``
     - ``ExampleSerializer(ModelSerializer)`` sur le modèle ``Example``
   * - ``apps/<name>/views.py``
     - ``ExampleViewSet(ModelViewSet)``
   * - ``apps/<name>/urls.py``
     - ``DefaultRouter`` enregistrant ``ExampleViewSet`` sur ``/examples/``

.. note::
   Les fichiers générés utilisent ``Example`` comme modèle placeholder. Ils sont conçus comme point de départ : le développeur remplace ``Example`` par le vrai modèle de l'application.

----

Section 12 — ``uvdjpro``
=========================

Signature
---------

.. code-block:: powershell

   uvdjpro -project <string>

Description
-----------

Générateur de projet complet. Crée un projet Django full-stack depuis zéro dans un nouveau dossier, installe toutes les dépendances, configure les settings, génère les applications ``users`` et ``maintenance``, crée tous les templates, exécute les migrations initiales et ouvre le projet dans VS Code.

**Durée estimée** : 2 à 5 minutes selon la vitesse de connexion (installation des paquets Python et npm).

Normalisation des noms
----------------------

À partir du paramètre ``-project``, trois variantes sont dérivées :

.. list-table::
   :widths: 25 30 45
   :header-rows: 1

   * - Variable
     - Exemple
     - Utilisation
   * - ``$project_snake``
     - ``mon_projet``
     - Nom du dossier, base de données, ``POSTGRES_DB``
   * - ``$project_kebab``
     - ``mon-projet``
     - Image Docker GHCR, domaine Nginx, ``docker-compose.prod.yml``
   * - ``$project_title``
     - ``Mon Projet``
     - ``<title>`` HTML, README, manifest PWA

Séquence d'exécution détaillée
--------------------------------

La génération suit un ordre précis conçu pour éviter les erreurs circulaires de chargement Django :

.. list-table::
   :widths: 5 30 65
   :header-rows: 1

   * - Étape
     - Action
     - Raison de l'ordre
   * - 1
     - ``uv init``
     - Initialise le projet uv
   * - 2
     - ``uv add django ...``
     - Installe toutes les dépendances
   * - 3
     - ``django-admin startproject config .``
     - Crée le squelette Django
   * - 4
     - Réécriture de ``manage.py``, ``wsgi.py``, ``asgi.py``
     - Injecte ``sys.path`` + ``config.settings.dev`` **avant** tout appel Django
   * - 5
     - Déplacement de ``config/settings.py``
     - Libère la place pour le package ``config/settings/``
   * - 6
     - Création de ``config/settings/base.py`` et ``dev.py``
     - Django peut maintenant démarrer correctement
   * - 7
     - Création manuelle de l'app ``theme``
     - Sans ``manage.py`` (évite le chargement circulaire)
   * - 8
     - Création manuelle des apps ``users`` et ``maintenance``
     - Idem — ``startapp`` est évité car ``INSTALLED_APPS`` les référence déjà
   * - 9
     - Génération de tous les fichiers (settings, templates, CSS, Docker…)
     - Ordre indépendant une fois Django opérationnel
   * - 10
     - ``uv run python manage.py makemigrations users``
     - Crée ``0001_initial.py`` pour le User custom
   * - 11
     - ``uv run python manage.py migrate``
     - Applique toutes les migrations (SQLite dev)

Dépendances installées
-----------------------

.. code-block:: text

   django
   gunicorn
   psycopg[binary]
   redis
   celery
   flower
   django-environ
   django-allauth[socialaccount]
   django-tailwind[reload]      ← inclut django_browser_reload
   whitenoise
   channels
   channels-redis
   djangorestframework
   pillow

Structure générée
-----------------

.. code-block:: text

   <project_snake>/
   ├── manage.py                        ← réécrit (sys.path + settings.dev)
   ├── .env                             ← SECRET_KEY et mots de passe générés aléatoirement
   ├── .gitignore
   ├── Makefile
   ├── README.md
   ├── Dockerfile
   ├── docker-compose.dev.yml
   ├── docker-compose.prod.yml
   ├── pyproject.toml                   ← géré par uv
   ├── apps/
   │   ├── users/                       ← User custom (AbstractUser)
   │   │   ├── models.py, forms.py, admin.py, adapters.py
   │   │   ├── views.py                 ← CBV : ProfileView, ProfileEditView
   │   │   ├── urls.py
   │   │   ├── migrations/0001_initial.py
   │   │   └── templates/users/profile.html, profile_edit.html
   │   └── maintenance/
   │       ├── views.py, urls.py
   │       └── apps.py
   ├── config/
   │   ├── urls.py
   │   ├── wsgi.py, asgi.py             ← réécrits
   │   └── settings/
   │       ├── __init__.py
   │       ├── base.py                  ← INSTALLED_APPS, allauth, Celery, Channels
   │       ├── dev.py                   ← SQLite3, consolemail
   │       ├── prod.py                  ← PostgreSQL, SMTP, HTTPS
   │       ├── test.py                  ← SQLite :memory:
   │       └── logging.py
   ├── templates/
   │   ├── base.html                    ← AlpineJS + HTMX + Tailwind + Google Fonts
   │   ├── layouts/
   │   │   ├── main-layout.html
   │   │   └── maintenance-layout.html  ← layout CSS autonome (.ml-*)
   │   ├── includes/
   │   │   ├── navbar.html, footer.html, messages.html
   │   ├── account/                     ← overrides allauth
   │   │   ├── base.html                ← charge auth-layout.css
   │   │   ├── login.html, signup.html, logout.html, password_reset.html
   │   │   └── email/
   │   └── maintenance/
   │       ├── maintenance.html
   │       └── errors/400.html, 403.html, 404.html, 500.html
   ├── static/
   │   ├── css/
   │   │   ├── styles.css
   │   │   └── layouts/
   │   │       ├── maintenance-layout.css   ← animations .ml-* autonomes
   │   │       └── auth-layout.css          ← styles .al-* autonomes
   │   ├── js/app.js
   │   ├── images/
   │   └── manifest.json
   ├── theme/                           ← app django-tailwind
   │   ├── apps.py
   │   └── static_src/
   │       ├── package.json
   │       ├── tailwind.config.js
   │       └── src/styles.css           ← @tailwind + composants .btn
   ├── nginx/default.conf
   └── .github/workflows/deploy.yml

Architecture des settings
--------------------------

.. code-block:: text

   config/settings/
   ├── base.py    ← commun à tous les environnements
   ├── dev.py     ← hérite de base, ajoute SQLite3 + debug email
   ├── prod.py    ← hérite de base, ajoute PostgreSQL + SMTP + HTTPS headers
   └── test.py    ← hérite de base, SQLite :memory: + locmem email

``manage.py`` pointe sur ``config.settings.dev``.
``wsgi.py`` et ``asgi.py`` pointent sur ``config.settings.prod``.

L'environnement peut être surchargé via la variable d'environnement ``DJANGO_SETTINGS_MODULE``.

Architecture CSS — stratégie sans Tailwind JIT
-----------------------------------------------

Deux fichiers CSS sont servis directement via ``{% static %}`` sans passer par la compilation Tailwind :

.. list-table::
   :widths: 40 60
   :header-rows: 1

   * - Fichier
     - Contexte d'utilisation
   * - ``static/css/layouts/maintenance-layout.css``
     - Pages de maintenance et d'erreurs (classes ``.ml-*``)
   * - ``static/css/layouts/auth-layout.css``
     - Pages d'authentification allauth (classes ``.al-*``)

Cette stratégie garantit que ces pages s'affichent correctement **même si ``make tailwind`` n'a jamais été lancé**, ce qui est le cas lors du premier ``djrun`` sur un projet fraîchement créé.

Les animations (``fadeUp``, ``spinCW/CCW``, ``barSlide``, ``fillBar``, ``blink``, ``pulse``) sont définies dans ces fichiers car elles ne peuvent pas être exprimées en classes Tailwind utilitaires.

Application ``users``
----------------------

L'application ``users`` remplace l'application Django native ``accounts`` pour éviter la collision d'URL avec django-allauth :

.. code-block:: python

   # config/urls.py
   path("users/",    include("users.urls", namespace="users")),   # profil, édition
   path("accounts/", include("allauth.urls")),                    # login, signup, OAuth

Le modèle ``User`` étend ``AbstractUser`` avec deux champs supplémentaires : ``avatar`` (``ImageField``) et ``bio`` (``TextField``).

Les vues sont basées sur des classes :

.. list-table::
   :widths: 30 30 40
   :header-rows: 1

   * - Classe
     - URL
     - Description
   * - ``ProfileView``
     - ``/users/profile/``
     - ``LoginRequiredMixin + TemplateView``
   * - ``ProfileEditView``
     - ``/users/profile/edit/``
     - ``LoginRequiredMixin + UpdateView``

Deux adapteurs allauth sont définis dans ``users/adapters.py`` :

- ``UserAdapter`` : sauvegarde les champs ``avatar`` et ``bio`` lors du signup
- ``SocialAccountAdapter`` : connecte automatiquement un compte social si l'email existe déjà en base

Fichier ``.env`` généré
------------------------

.. code-block:: ini

   SECRET_KEY=<64 chars aléatoires>
   DOMAIN=<project_kebab>.laurentjouron.dev

   POSTGRES_DB=<project_snake>
   POSTGRES_USER=<project_snake>
   POSTGRES_PASSWORD=<32 chars aléatoires>

   REDIS_URL=redis://redis:6379/0

   GOOGLE_CLIENT_ID=
   GOOGLE_CLIENT_SECRET=
   FACEBOOK_APP_ID=
   FACEBOOK_APP_SECRET=

   DEFAULT_FROM_EMAIL=noreply@<project_kebab>.laurentjouron.dev
   EMAIL_HOST=smtp.mailgun.org
   EMAIL_PORT=587
   EMAIL_HOST_USER=
   EMAIL_HOST_PASSWORD=

``SECRET_KEY`` et ``POSTGRES_PASSWORD`` sont générés aléatoirement à chaque exécution via :

.. code-block:: powershell

   -join ((48..57)+(65..90)+(97..122) | Get-Random -Count 64 | ForEach-Object {[char]$_})

Infrastructure Docker
----------------------

Deux fichiers ``docker-compose`` sont générés :

**docker-compose.dev.yml** — développement conteneurisé (sans PostgreSQL, SQLite utilisé) :

.. code-block:: yaml

   services:
     web:   # Django runserver
     redis: # Pour Celery

**docker-compose.prod.yml** — production complète :

.. code-block:: yaml

   services:
     nginx:    # Reverse proxy Cloudflare-ready
     web:      # Image GHCR gunicorn
     postgres: # PostgreSQL 16
     redis:    # Redis 7
     worker:   # Celery worker
     beat:     # Celery beat
     flower:   # Monitoring Celery

Le ``Dockerfile`` utilise un build multi-stage (``python:3.12-slim``) avec ``uv sync --frozen`` et ``collectstatic`` intégrés.

CI/CD GitHub Actions
---------------------

Le workflow ``.github/workflows/deploy.yml`` déclenché sur push ``main`` exécute deux jobs :

1. **build** : build + push de l'image Docker vers GHCR (``ghcr.io/laurentjouron/<repo>:latest``)
2. **deploy** : connexion SSH au NAS Synology et ``docker compose pull && up -d``

Secrets GitHub requis :

.. code-block:: text

   GHCR_TOKEN      ← Personal Access Token GitHub (read:packages, write:packages)
   NAS_HOST        ← IP ou hostname du NAS
   NAS_USER        ← Utilisateur SSH
   NAS_SSH_KEY     ← Clé privée SSH

----

Points techniques importants
==============================

Gestion du ``sys.path``
-------------------------

La résolution d'imports des applications dans ``apps/`` repose sur l'injection de ``apps/`` dans ``sys.path`` au démarrage de chaque point d'entrée Python :

.. code-block:: python

   # manage.py, wsgi.py, asgi.py
   apps_dir = os.path.join(BASE_DIR, "apps")
   if apps_dir not in sys.path:
       sys.path.insert(0, apps_dir)

Cela permet de déclarer ``"users"`` dans ``INSTALLED_APPS`` (et non ``"apps.users"``) et d'utiliser des imports relatifs normaux dans les applications.

Contrainte : ``apps/`` **ne doit pas** contenir de fichier ``__init__.py``. Un tel fichier transformerait ``apps`` en package Python, ce qui casserait les imports relatifs des applications.

Ordre de chargement et contrainte ``INSTALLED_APPS``
-----------------------------------------------------

Lors de l'exécution de ``manage.py startapp``, Django charge ``INSTALLED_APPS`` pour valider les dépendances. Si ``users`` ou ``maintenance`` sont listés dans ``INSTALLED_APPS`` mais que leurs dossiers n'existent pas encore, Django lève une ``ModuleNotFoundError``.

La solution retenue dans ``uvdjpro`` est de **créer manuellement** le contenu des apps ``users`` et ``maintenance`` (``__init__.py``, ``apps.py``, ``migrations/__init__.py``) sans utiliser ``manage.py startapp``. L'app ``theme`` (django-tailwind) suit la même logique.

Encodage des here-strings PowerShell
--------------------------------------

Tous les here-strings utilisent la syntaxe ``(@' ... '@)`` (single-quoted) avec le chaînage ``.Replace()`` immédiat sur la même expression entre parenthèses. La syntaxe ``@' ... '@\n.Replace(...)`` (point sur la ligne suivante) est invalide en PowerShell et constitue l'erreur la plus courante lors de la maintenance du fichier.

Les here-strings qui doivent interpoler des variables PowerShell (comme ``$project_title`` dans les templates HTML ou ``$project_kebab`` dans les fichiers Docker) utilisent la syntaxe ``@" ... "@`` (double-quoted).

Tous les fichiers sont écrits en ``-Encoding utf8`` pour garantir la compatibilité avec Python et les navigateurs.