# Test de rembourrage Oracle

|ID          |
|------------|
|WSTG-CRYP-02|

## Sommaire

Un oracle de remplissage est une fonction d'une application qui décrypte les données cryptées fournies par le client, par ex. l'état de session interne stocké sur le client, et divulgue l'état de la validité du bourrage après déchiffrement. L'existence d'un oracle de remplissage permet à un attaquant de déchiffrer des données chiffrées et de chiffrer des données arbitraires sans connaître la clé utilisée pour ces opérations cryptographiques. Cela peut conduire à une fuite de données sensibles ou à des vulnérabilités d'escalade de privilèges, si l'intégrité des données chiffrées est assumée par l'application.

Les chiffrements par blocs chiffrent les données uniquement dans des blocs de certaines tailles. Les tailles de bloc utilisées par les chiffrements courants sont de 8 et 16 octets. Les données dont la taille ne correspond pas à un multiple de la taille de bloc du chiffrement utilisé doivent être rembourrées d'une manière spécifique afin que le décrypteur puisse supprimer le remplissage. Un schéma de remplissage couramment utilisé est PKCS#7. Il remplit les octets restants avec la valeur de la longueur de remplissage.

### Exemple 1

Si le remplissage a une longueur de 5 octets, la valeur d'octet « 0x05 » est répétée cinq fois après le texte brut.

Une condition d'erreur est présente si le remplissage ne correspond pas à la syntaxe du schéma de remplissage utilisé. Un oracle de remplissage est présent si une application divulgue cette condition d'erreur de remplissage spécifique pour les données chiffrées fournies par le client. Cela peut se produire en exposant directement des exceptions (par exemple, `BadPaddingException` en Java), par des différences subtiles dans les réponses envoyées au client ou par un autre canal latéral comme le comportement de synchronisation.

Certains modes de fonctionnement de la cryptographie permettent des attaques par retournement de bits, où le retournement d'un bit dans le texte chiffré fait que le bit est également retourné dans le texte en clair. Le basculement d'un bit dans le n-ième bloc de données cryptées CBC fait que le même bit dans le (n+1)-ième bloc est basculé dans les données décryptées. Le n-ième bloc du texte chiffré déchiffré est effacé par cette manipulation.

L'attaque oracle de remplissage permet à un attaquant de déchiffrer des données chiffrées sans connaître la clé de chiffrement et le chiffrement utilisé en envoyant des textes chiffrés habilement manipulés à l'oracle de remplissage et en observant les résultats renvoyés par celui-ci. Cela entraîne une perte de confidentialité des données cryptées. Par exemple. dans le cas de données de session stockées côté client, l'attaquant peut obtenir des informations sur l'état interne et la structure de l'application.

Une attaque oracle de remplissage permet également à un attaquant de chiffrer des textes bruts arbitraires sans connaître la clé et le chiffrement utilisés. Si l'application suppose que l'intégrité et l'authenticité des données déchiffrées sont données, un attaquant pourrait être en mesure de manipuler l'état de la session interne et éventuellement d'obtenir des privilèges plus élevés.

## Objectifs des tests

- Identifiez les messages chiffrés qui reposent sur le remplissage.
- Essayez de casser le rembourrage des messages chiffrés et analysez les messages d'erreur renvoyés pour une analyse plus approfondie.

## Comment tester

### Test en boîte noire

Tout d'abord, les points d'entrée possibles pour les oracles de remplissage doivent être identifiés. En règle générale, les conditions suivantes doivent être remplies :

1. Les données sont cryptées. Les bons candidats sont des valeurs qui semblent aléatoires.
2. Un chiffrement par blocs est utilisé. La longueur du texte chiffré décodé (Base64 est souvent utilisé) est un multiple des tailles de bloc de chiffrement courantes, comme 8 ou 16 octets. Différents textes chiffrés (par exemple, rassemblés par différentes sessions ou manipulation de l'état de la session) partagent un diviseur commun dans la longueur.

#### Exemple 2

`Dg6W8OiWMIdVokIDH15T/A==` donne après décodage Base64 `0e ??0e 96 f0 e8 96 30 87 55 a2 42 03 1f 5e 53 fc`. Cela semble être aléatoire et long de 16 octets.

Si un tel candidat de valeur d'entrée est identifié, le comportement de l'application à la falsification bit à bit de la valeur cryptée doit être vérifié. Normalement, cette valeur encodée en Base64 inclura le vecteur d'initialisation (IV) ajouté au texte chiffré. Étant donné un texte en clair *`p`* et un chiffrement avec une taille de bloc *`n`*, le nombre de blocs sera *`b = ceil( length(b) / n)`*. La longueur de la chaîne chiffrée sera *`y=(b+1)*n`* en raison du vecteur d'initialisation. Pour vérifier la présence de l'oracle, décodez la chaîne, retournez le dernier bit de l'avant-dernier bloc *`b-1`* (le bit le moins significatif de l'octet à *`y-n-1`*), re -encoder et envoyer. Ensuite, décodez la chaîne d'origine, retournez le dernier bit du bloc *`b-2`* (le bit le moins significatif de l'octet à *`y-2*n-1`*), réencodez et envoyez.

Si l'on sait que la chaîne cryptée est un bloc unique (l'IV est stocké sur le serveur ou l'application utilise une mauvaise pratique d'IV codée en dur), plusieurs retournements de bits doivent être effectués à tour de rôle. Une autre approche pourrait consister à ajouter un bloc aléatoire au début et à inverser les bits afin que le dernier octet du bloc ajouté prenne toutes les valeurs possibles (0 à 255).

Les tests et la valeur de base doivent au moins provoquer trois états différents pendant et après le déchiffrement :

- Le texte chiffré est déchiffré, les données résultantes sont correctes.
- Le texte chiffré est déchiffré, les données résultantes sont brouillées et provoquent une gestion des exceptions ou des erreurs dans la logique de l'application.
- Le déchiffrement du texte chiffré échoue en raison d'erreurs de remplissage.

Comparez soigneusement les réponses. Recherchez en particulier les exceptions et les messages qui indiquent que quelque chose ne va pas avec le rembourrage. Si de tels messages apparaissent, l'application contient un oracle de remplissage. Si les trois états différents décrits ci-dessus sont observables implicitement (différents messages d'erreur, canaux latéraux de synchronisation), il y a une forte probabilité qu'un oracle de remplissage soit présent à ce stade. Essayez d'effectuer l'attaque oracle de rembourrage pour vous en assurer.

##### Exemple 3

- ASP.NET lève `System.Security.Cryptography.CryptographicException: Padding is invalid and cannot be removed.` si le rembourrage d'un texte chiffré déchiffré est rompu.
- En Java, une `javax.crypto.BadPaddingException` est lancée dans ce cas.
- Des erreurs de déchiffrement ou similaires peuvent être des oracles de remplissage possibles.

> Une implémentation sécurisée vérifiera l'intégrité et ne provoquera que deux réponses : `ok` et `failed`. Il n'y a pas de canaux latéraux qui peuvent être utilisés pour déterminer les états d'erreur internes.

### Test en boîte grise

Vérifiez que tous les endroits où les données chiffrées du client, qui ne doivent être connues que du serveur, sont déchiffrées. Les conditions suivantes doivent être remplies par un tel code :

1. L'intégrité du texte chiffré doit être vérifiée par un mécanisme sécurisé, comme HMAC ou des modes de fonctionnement chiffrés authentifiés comme GCM ou CCM.
2. Tous les états d'erreur lors du déchiffrement et du traitement ultérieur sont traités de manière uniforme.

### Exemple 4

[Visualisation du processus de décryptage](https://erlend.oftedal.no/blog/poet/)

## Outils

- [Bletchley](https://code.blindspotsecurity.com/trac/bletchley)
- [PadBuster](https://github.com/GDSSecurity/PadBuster)
- [Outil d'exploitation d'Oracle de rembourrage (POET)](http://netifera.com/research/)
- [Poracle](https://github.com/iagox86/Poracle)
- [python-paddingoracle](https://github.com/mwielgoszewski/python-paddingoracle)

## Références

- [Wikipédia - Padding Oracle Attack](https://en.wikipedia.org/wiki/Padding_oracle_attack)
- [Juliano Rizzo, Thai Duong, "Practical Padding Oracle Attacks"](https://www.usenix.org/event/woot10/tech/full_papers/Rizzo.pdf)
