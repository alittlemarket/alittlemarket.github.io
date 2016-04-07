---
layout: post
title:  "Use https to securise our websites"
date:   2016-04-07
categories: security
tags: [security, https, http, migration]
authors: [antoine]
---

Phases :
- définition des points bloquants
	- listing des noms de domaines pas compatibles (sous-domaines multiples et sous-domaines contenant des underscores)
		- static.commun.alittlemarket.com, static_commun.alittlemarket.com
		- img.commun.alittlemarket.com, img_commun.alittlemarket.com
		- ...
	- listing des dépendances
		- blogs : www.alittlemag.com, blog.alittlemarket.com
		- assets : static.alittlemarket.com, assets-orig.alittlemarket.com, images.alittlemarket.com, img-commun.alittlemarket.com
		- images : galerie.alittlemarket.com
		- ...
	- centaines d'URLs "en dur" à mettre à jour
	- définition avec notre consultant SEO de la marche à suivre
- exécution
	- assets
		- remplacement de toutes les URLs en dur des assets par des constantes (une par domaine) avec déploiement progressif
		- activation du https
	- images
		- remplacement de toutes les URLs en dur des assets par des constantes (une par domaine) avec déploiement progressif
		- activation du https
	- remplacement de toutes les URLs en dur du site par des constantes avec déploiement progressif
	- activation du https sur les dépendances
		- blogs : www.alittlemag.com, blog.alittlemarket.com
	- SEO
		- création d'un nouveau domaine dans Gooogle Search Console
- monitoring
	- mise en place de métrique Statsd pour suivre le crawl Google http/https
	- tracking dans GA des sessions http/https
- activation progressive
	- activation sur princess
	- activation en prod juste pour les admins
	- activation en prod pour tous (site par site)
	- redirection du http vers https
- problèmes relatifs aux https
	- bug avec Mondial Relay
- retours x semaines après
	- ...