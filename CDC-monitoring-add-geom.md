# Cahier des charges pour l'ajout de géométrie dans le module `gn_module_monitoring`

> [!CAUTION]
> En cours de rédaction

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

## Modèle de données

Ajout d'un champ `additional_geom` de type `Geometry`et `NULLABLE` dans les tables `t_base_visits` et `t_observations`.

Ces champs devront comporter un index.

## Configuration JSON

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

## Backend : API

### Routes d'ajout et de modiciation d'une observation/d'une visite (POST/PUT)
Il sera nécessaire de modifier les routes d'ajout et modification des visites et observations afin de pouvoir envoyer la géométrie saisie au backend, seulement si la fonctionnalité elle activée et renseignée.

### Routes de consultation d'une observation ou visite (GET)
Il sera là aussi nécessaire de modifier les routes qui permettent de remonter les données utiles à l'initialisation du formualire lors d'un ajout afin de faire remonter le champ `additional_geom`.

## Remontée dans la synthese

Le standard d'échange SOT v3.0 précise bien que un relevé = une date + une personne + 1 lieu. A ce titre seule la géométrie du site ne doit remonter dans la synthese.

Les géométries additionnelles des visites et observations ne doivent donc en aucun cas remonter dans la synthese. Elles resteront au niveau du module monitoring et ne seront utlisées que lors d'analyse de données propres à chaque protocole (sous-module).

## Frontend

### Affichage de la visite

Aujourd'hui, sur la page "Détail d'une visite" une carte est affichée et centrée sur la géométrie du site. 
Si une géométrie additionnelle est présente, il faudrait l'afficher sur cette même carte, en gris. L'affichage du site restera identique aisni que le centrage de la carte.

### Edition d'une visite

Aujourd'hui seul un formulaire est présent sans carte.

Si la géométrie additionnelle est activée pour le protocole (sous-module) en cours, alors utiliser la même mécanique que pour l'édition d'une observation : faire apparaître la carte à gauche en mode éditable.

La carte doit comporter la géographie du site qui ne doit en aucun ca interférer avec l'édition d'une nouvelle géométrie.

Si l'utilisateur a les droits sur l'objet C U D `MONITORINGS_VISITES` dans le sous-module en question, alors il pourra C U D, sinon la carte sera seulement en mode visualisation.

Les outils d'édition de la carte devront être adaptés aux types de géométries possibles (cf fichier de configuration).

Si la géométrie est attendue, afficher dès le chargement du formulaire le même message que pour les sites : "Veuillez saisir une géométrie sur la carte". Le bouton de validation ne sera actif qu'une fois la géométrie validée.

### Affichage d'une observation

Idem que pour la visite.

### Edition d'une observation

Même principe que pour la visite. 

Petite nuance du côté des droits : Si l'utilisateur a le droit d'ajout/modification sur les visites du sous-module, il a tous les droits sur l'observation, donc la carte s'affichera systématiquement en mode édition.