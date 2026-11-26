# CR workshop du 23/11/2016

## Présents

- Amandine Sahl
- Camille Monchicourt
- Donovan Maillard
- Maxime Toma
- Alain Delvare
- William Boffy
- Anès Nobbigh
- Claire Lagaye
- Cynthia Borot

## Contexte

Pour rappeler le besoin [#37](https://github.com/PnX-SI/gn_module_monitoring/issues/37), nous aurions besoin d'ajouter une géométrie à l'observation afin de pouvoir faire du distance sampling entre l'observateur (géométrie du site ou sous site) et l'objet de l'observation (pour des protocoles de type IPA ou IPS).

Dans le même ticket, il s'avère qu'il y aurait de même besoin d'une géométries pour les visites.

Nous (PN Vanoise) souhaitons financer les devs pour les observations, et si la même solution technique peut-être appliquée aux visites, nous envisageons de financer l'ensemble.

Il nous faudrait assez rapidement établir un cahier des charges avec des précisions techniques tranchées.

- Stockage de la géométrie : JSONB ou Geometry
- Geométrie sur l'observation et sur la visite ?
- Quelle géométrie à remonter dans la synthese ? Préconisations

## CR

## Stockage de la géométrie : JSONB ou Geometry

Visites et observations :

- Champ Geom nullable avec un index
- Remontée dans la synthese : la géométrie la plus précise, la plus cohérente et adaptée à chaque protocol

Visites :

- Cas d'un transect -> Est-ce plus un effort de prospection ? (La propection est un sujet à proprement parlé)
- Dans la plupart des cas le sous-site répond au besoin

Vue pour la synthese :

Pb avec le standard pour l'observation : Un relevé = une date + une personne + 1 lieu

La géométrie de référence : celle du site (ou éventuellement de la visite)
Les autres géométries n'ont pas vocation à monter dans la synthese, elles ne servent qu'à l'nalyse des données du module, d'où l'idée de les appeler "géométries additionelles".

## Front

Le front détecte déjà l'ajout de géométries au niveau de la visite et du site en affichant le composant carte.
De nombreuses briques sont déjà existantes.

Saisie : S'appuyer sur la saisie carte déjà existante

Affichage des géométries :

Affichage de la géométrie du niveau en cours et du site.
Travail sur la symbologie à faire pour dissocier la géomtrie du niveau en cours et celle du site.

## Architecture

Backend et BDD : au + proche de la définition
Le restant côté front : dans monitoring