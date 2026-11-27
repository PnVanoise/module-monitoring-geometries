# Cahier des charges pour l'ajout de géométrie dans le module `gn_module_monitoring`

> [!CAUTION]
> En cours de rédaction

- [Cahier des charges pour l'ajout de géométrie dans le module `gn_module_monitoring`](#cahier-des-charges-pour-lajout-de-géométrie-dans-le-module-gn_module_monitoring)
  - [Contexte](#contexte)
    - [Origine du projet](#origine-du-projet)
    - [Groupe de travail](#groupe-de-travail)
  - [Expression du besoin](#expression-du-besoin)
    - [Modèle de données](#modèle-de-données)
    - [Configuration JSON](#configuration-json)
    - [Backend : API](#backend--api)
    - [Remontée dans la synthese](#remontée-dans-la-synthese)
    - [Frontend](#frontend)
      - [Affichage de la visite](#affichage-de-la-visite)
      - [Edition d'une visite](#edition-dune-visite)
      - [Affichage d'une observation](#affichage-dune-observation)
      - [Edition d'une observation](#edition-dune-observation)
  - [Modalité de réalisation](#modalité-de-réalisation)
    - [Version](#version)
    - [Dépôt](#dépôt)
    - [Github project](#github-project)
    - [Développments](#développments)
    - [Tests](#tests)
    - [Documentation](#documentation)
    - [Livraison de la commande](#livraison-de-la-commande)

## Contexte

### Origine du projet

Dans l'issue [#37](https://github.com/PnX-SI/gn_module_monitoring/issues/37) du module monitoring, un échangé a été mené sur plusieurs années pour ajouter des géométries au niveau de l'observation et de la visite.

Le Parc national de la Vanoise, dans le cadre du projet européen [Alcotra Interreg Cross-DBio](https://vanoise-parcnational.fr/nos-actions/contribuer-des-programmes-de-cooperation-transfrontalieres/cross-dbio-2025-2028) souhaite financer ces développements en faisant appel à une prestation de développement fullstack sur l'envirnnement GeoNature

### Groupe de travail

Un groupe de travail s'est tenu le 23/11/2026 pour convenir de l'orientation du cahier des charges.

Composition :

- Mainteneurs de GeoNature (Amandine Sahl, Camille Monchicourt)
- Contributeurs à GeoNature (Donovan Maillard, Maxime Toma, Alain Delvare, Anès Nobbigh, William Boffy)
- PN Vanoise coordinateur du projet (Claire Lagaye, Cynthia Borot)

## Expression du besoin

Dans le module monitoring de GeoNature, nous souhaiterions que des géométries dites 'secondaires' ou additionnelles' soient ajoutées aux visites et observations. Ces géométries répondraient aux besoins d'analyse de données (statistiques) propre à chaque sous-module et donc chaque protocole de monitoring.

### Modèle de données

Ajout d'un champ `additional_geom` de type `Geometry`et `NULLABLE` dans les tables `t_base_visits` et `t_observations`.

Ces champs devront comporter un index.

### Configuration JSON

Voici comment doit apparaître la configuration des géométries dans le fichier de configuration des visites par exemple (idem pour les observations) :

```json
{
    "display_properties": [...],
    "display_list": [...],
    "specific": {
        ...,
        "additional_geom": {
            "geometry_type": "Point",
            "required": false
        }
    }
}
```

`additional_geom` : La présence de ce champ indique la possibilité de saisir une géométrie.

`geometry_type` : Indique le type de géométries qui peuvent être saisies. La valeur de ce champ peut prendre :

- un seul type de géométrie parmis `Point`, `Multipoint`, `Polygon`, `Line`, `Multipolygon`, `Multilines` 
- ou une liste de type de géométries séparées par des virgules,
- ou `All` pour indiquer tout type de géométrie.

`required`: C'est un boolean `true/false` qui indique si la saisie de la géométrie est obligatoire ou non.

*`type_widget` non utilisé* : Ce champ n'est utilisé car d'une part il ne varierait jamais et d'autre part, il ne s'agit pas vraiment de l'ajout d'un widget mais de l'utilisation de l'outil cartographique déjà présent en édition.

### Backend : API

**Routes d'ajout et de modiciation d'une observation/d'une visite (POST/PUT)**
Il sera nécessaire de modifier les routes d'ajout et modification des visites et observations afin de pouvoir envoyer la géométrie saisie au backend, seulement si la fonctionnalité elle activée et renseignée.

**Routes de consultation d'une observation ou visite (GET)**
Il sera là aussi nécessaire de modifier les routes qui permettent de remonter les données utiles à l'initialisation du formualire lors d'un ajout afin de faire remonter le champ `additional_geom`.

### Remontée dans la synthese

Le standard d'échange SOT v3.0 précise bien que un relevé = une date + une personne + 1 lieu. 

A ce titre, Les géométries additionelles n'ont pas vocation à remonter dans la synthèse : le comportement par défaut et recommandé reste l'utilisation de la géométrie du site, non nullable, c'est-à-dire le fonctionnement actuel.

### Frontend

#### Affichage de la visite

Aujourd'hui, sur la page "Détail d'une visite" une carte est affichée et centrée sur la géométrie du site. 
Si une géométrie additionnelle est présente, il faudrait l'afficher sur cette même carte, en gris. L'affichage du site restera identique aisni que le centrage de la carte.

#### Edition d'une visite

Aujourd'hui seul un formulaire est présent sans carte.

Si la géométrie additionnelle est activée pour le protocole (sous-module) en cours, alors utiliser la même mécanique que pour l'édition d'une observation : faire apparaître la carte à gauche en mode éditable.

La carte doit comporter la géographie du site qui ne doit en aucun ca interférer avec l'édition d'une nouvelle géométrie.

Si l'utilisateur a les droits sur l'objet C U D `MONITORINGS_VISITES` dans le sous-module en question, alors il pourra C U D, sinon la carte sera seulement en mode visualisation.

Les outils d'édition de la carte devront être adaptés aux types de géométries possibles (cf fichier de configuration).

Si la géométrie est attendue, afficher dès le chargement du formulaire le même message que pour les sites : "Veuillez saisir une géométrie sur la carte". Le bouton de validation ne sera actif qu'une fois la géométrie validée.

#### Affichage d'une observation

Idem que la visite.

#### Edition d'une observation

Idem que la visite avec une nuance du côté des droits : Si l'utilisateur a le droit d'ajout/modification sur les visites du sous-module, il a tous les droits sur l'observation, donc la carte s'affichera systématiquement en mode édition.

## Modalité de réalisation

### Version

Les nouveau développements devront être à jour de la branche develop du dépot [https://github.com/PnX-SI/gn_module_monitoring](https://github.com/PnX-SI/gn_module_monitoring)

### Dépôt

Ils ont l'habitude de faire un fork sur https://github.com/naturalsolutions et n'ont pas vraiment le choix car pas les droits sur PnX-SI.

Le client réalisera un fork [https://github.com/PnX-SI/gn_module_monitoring](https://github.com/PnX-SI/gn_module_monitoring) et sera attentif à tenir la branche develop à jour de celle du dépôt d'origine.

Le nom du dépôt ainsi que la branche de fusion des développements devront être communiqués dès le début du projet au client.

### Github project

Afin que le client et l'équipe projet associée puisse suivre les développements, un Github project devra être créé. Le client devra faire partie du workflow de validation des principales étapes des développements.


### Développments

Les développements seront réalisés dans les languages imposés par la plateforme GeoNature. Il est attendu que le code soit clair :

- Noms de variables et de fonctions explicites et en anlais.
- Fonctions : Elles doivent être écrites dans le principe de responsabilité unique.
- Code simple : si des portions de codes semblent complexes et difficiles à comprendre, il est souhaitable de les réécrire de façon plus lisible et compréhensible. Ne pas hésiter à aussi ajouter un commentaire en anglais.
- Coder pour satisfaire les exigences du passage de GeoNature à Angular v15 tant que possible.
- Réutiliser au maximun les composants / les modules du coeur de GeoNature.
- Penser à documenter dans un document à part si besoin pour expliquer la démarche de certains développement complexe -> maintenabilité.
- Etre très attentifs à n'écrire aucun texte en "dur", mais passer systématiquement par les fonctions de traduction et les fichiers de langue.
- Attention à la généricité du code.

**Côté versionning**

Ne pas hésiter à publier des commits fonctionnellement cohérent et comportant peu de modifications :

  - Explicite
  - En anglais
  - Utilisant les en-têtes en vigeur (fix/feat/doc/chore ...)
  
### Tests

Des tests unitaires sur le backend devront être réalisés, de façon à passer le taux de recouvrement du code exigé par le dépôt [https://github.com/PnX-SI/gn_module_monitoring](https://github.com/PnX-SI/gn_module_monitoring).

Pour le frontend, des tests du parcours utilisateur devront être réalisés.

### Documentation

Il est attendu que la documentation des sous-modules du module monitoring ([doc/sous_module.md](https://github.com/PnX-SI/gn_module_monitoring/blob/main/docs/sous_module.md)) soit mise à jour en conséquence.

En collaboration avec le PNV, il est de même demandé qu'un sous module soit créé et configuré en utilisant ces nouvelles fonctionnalités (à titre d'exemple protocole IPS/IPA) et déposé sous [PnX-SI/protocoles_suivi](https://github.com/PnX-SI/protocoles_suivi).

### Livraison de la commande

Les délais seront à convenir avec les prestataires lors de la signature mais l'ensemble des devs devra être contenu dans une seule et unique branche nommée `feat/additionalGeom`.

Une PR, à jour de la branche `develop` devra être créée sur [https://github.com/PnX-SI/gn_module_monitoring](https://github.com/PnX-SI/gn_module_monitoring) afin que les mainteneurs puissent relire et valider les développements avant fusion à la branche `develop`.