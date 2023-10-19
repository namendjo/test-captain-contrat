## 1. Regex

#### Ne surligner que SAS dans le text fourni
- toutes les occurrences
```pseudo
/(?:SAS)/g
```
- uniquement la première occurrence
```pseudo
/(?:SAS)
```
#### Ne sélectionner que les formes juridiques anonymes
```pseudo
/^SAS|SASU/g
```
#### Ecrire une regex pour détecter l’Alsace-Moselle dans une adresse
Considérant ce texte:

```
67000 Strasbourg
59000 Lille
68000 Colmar
69000 Lyon
57000 Metz
```

```pseudo
/(?:67|68|57)[ ,.-]+((?:[0-9]{5}|[0-9]{6})[ ,.-]+)(?:[A-Za-z])+/gm
```

## 2. Configuration

L'objectif est de rendre l'offre détaillée dans l'énoncée modulable et compétitive en proposant une composante optionnelle facturée 40 euros au client.

Le prix total de l'offre de création de SAS actuel semble ne pas correspondre à la somme du prix de l'offre et des prix de ses composants.
De deux choses l'une:
- soit il y'a une erreur sur l'un ou plusieurs des prix proposés
- soit c'est une stratégie de pricing assumée de la boîte de marger en négatif pour rester compétitif

<!-- Une offre possède plusieurs composants. Son prix total correspond au prix de l’offre plus le prix des composants.
Les composants sont de type regular, admin_fee ou option. Les options sont choisies par le client, le reste est forcément présent dans l’offre. Ils servent notamment à avoir un détail des frais et de ce qui constitue une offre.
Les composants contiennent des todos : il s’agit des actions que devront effectuer le client ou les juristes afin de réaliser l’offre.


Voici notre offre création SAS :
Prix : 169€

Composants
1. offre basique
prix : 0
type : regular
todos : vérification du régime fiscal, rédaction des statuts, upload du certificat de dépôt des fonds, relecture des statuts, rédaction de l’annonce légale, publication de l’annonce légale

2. publication annonce légale
prix: 150 
type: admin_fee
todos: publication de l’annonce légale

3. immatriculation au greffe
prix: 70
type: admin_fee

4. dépôt de la marque
prix: 80
type: option
todos : choix du nom de la marque, choix des classes, dépôt de la marque
-->

La solution que l'on propose est de créer un nouveau composant optionnel *vérification du régime fiscal*, facturé 40 euros au client. On retirera l'intitulé de ce composant comme todos de l'*offre basique*. L'offre finale ressemblera à:

#### Composants
**1. offre basique**
- prix : 0
- type : regular
- todos : rédaction des statuts, upload du certificat de dépôt des fonds, relecture des statuts, rédaction de l’annonce légale, publication de l’annonce légale

**2. publication annonce légale**
- prix: 150 
- type: admin_fee
- todos: publication de l’annonce légale

**3. immatriculation au greffe**
- prix: 70
- type: admin_fee

**4. dépôt de la marque**
- prix: 80
- type: option
- todos : choix du nom de la marque, choix des classes, dépôt de la marque

**_5. Vérification régime fiscal_**
- _prix: 40_
- _type: option_
- _todos: vérification du régime fiscal_

Nouveau prix de l'offre création SAS : **129 €**

## 3. Automatisation
<!-- balise : conteneur; élément des jeux de données sur sur le client
    Chaque balise débute {{ et se termine par }}
        
    Affichage conditionnel basé sur ces balises

        {% if balise == 'reponse' %}
            commande_1
        {% else %}
            commande_2
        {% endif %}

    Boucle basée sur ces balises

        {% for user in partners %}
            {{user.email.TX}}
        {% endfor %}
 -->

L'objectif est d'afficher un texte en préambule de contrat en fonction du fait que l'utilisateur identifié soit une personne physique ou morale.

Si c'est une personne physique, il s'agira d'afficher les infos suivantes:
- son genre (au choix parmi 2 possibilités)
- son prénom
- son nom
- sa nationalité
- sa date de naissance
- et son adresse de domicile

S'il s'agit d'une personne morale, il s'agira d'afficher les infos suivantes:
- sa dénomination sociale
- sa forme juridique (au choix parmi 5 possibilités)
- son capital social
- l'adresse de son siège social
- sa ville d'immatriculation
- son numéro SIREN
- le nom de son représentant légal
- les nom, prénom et capital correspondant à chacun de ses associés

> __On suppose pour les besoins du cas que le prestataire a rentré toutes les infos nécessaires dans les champs adéquats et qu'aucun champ n'a été laissé vide.__


```pseudo
{% if type_prestataire_MC == 'PP' %}

    {% if sexe_prestataire_MC == 'masculin' %}
        Monsieur
    {% else %}
        Madame
    {% endif %}

    {{prenom_prestataire_TX}}
    {{nom_prestataire_TX}},
    de nationalité
    {{nationalite_TX}},
    né(e) le
    {{date_naissance_prestataire_DT}},
    demeurant
    {{adresse_domicile_prestataire_TX}}

{% else %}

    La société
    {{denomination_sociale_prestataire_TX}},

    {% if forme_juridique_prestataire_MC == 'SAS' %}
        Société par actions simplifiée 
    {% else if forme_juridique_prestataire_MC == 'SARL' %}
        Société à responsabilité limitée 
    {% else if forme_juridique_prestataire_MC == 'SASU' %}
        Société par actions simplifiée unipersonnelle 
    {% else if forme_juridique_prestataire_MC == 'SCI' %}
        Société civile immobilière 
    {% else if forme_juridique_prestataire_MC == 'EURL' %}
        Entreprise unipersonnelle à responsabilité limitée 
    {% else %}
        {{autre_forme_juridique_prestataire_TX}} 
    {% endif %} 

    au capital de 
    {{capital_social_prestataire_NB}}
    euros, dont le siège social est fixé au 
    {{adresse_siege_social_prestataire_TX}}, 
    immatriculée au Registre du Commerce et des Sociétés de 
    {{ville_RCS_prestataire_TX}} 
    sous le numéro 
    {{numero_RCS_prestataire_TX}}, 
    représentée par 
    {{representant_legal_prestataire_TX}}, 
    dirigeant de la société.

    Sont associés à la création :
    
    {% for associe in stakeholders %}
        {{associe.nom_stakeholder_TX}}, 
        {{associe.prenom_stakeholder_TX}} 
        à hauteur d’un capital de 
        {{associe.capital_stakeholder_NB}} €
    {% endfor %}
    
{% endif %}
```