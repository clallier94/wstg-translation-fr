# Test de rembourrage Oracle

|ID          |
|------------|
|WSTG-CRYP-02|

## Sommaire

Un oracle de remplissage est une fonction d'une application qui d�crypte les donn�es crypt�es fournies par le client, par ex. l'�tat de session interne stock� sur le client, et divulgue l'�tat de la validit� du bourrage apr�s d�chiffrement. L'existence d'un oracle de remplissage permet � un attaquant de d�chiffrer des donn�es chiffr�es et de chiffrer des donn�es arbitraires sans conna�tre la cl� utilis�e pour ces op�rations cryptographiques. Cela peut conduire � une fuite de donn�es sensibles ou � des vuln�rabilit�s d'escalade de privil�ges, si l'int�grit� des donn�es chiffr�es est assum�e par l'application.

Les chiffrements par blocs chiffrent les donn�es uniquement dans des blocs de certaines tailles. Les tailles de bloc utilis�es par les chiffrements courants sont de 8 et 16 octets. Les donn�es dont la taille ne correspond pas � un multiple de la taille de bloc du chiffrement utilis� doivent �tre rembourr�es d'une mani�re sp�cifique afin que le d�crypteur puisse supprimer le remplissage. Un sch�ma de remplissage couramment utilis� est PKCS#7. Il remplit les octets restants avec la valeur de la longueur de remplissage.

### Exemple 1

Si le remplissage a une longueur de 5 octets, la valeur d'octet � 0x05 � est r�p�t�e cinq fois apr�s le texte brut.

Une condition d'erreur est pr�sente si le remplissage ne correspond pas � la syntaxe du sch�ma de remplissage utilis�. Un oracle de remplissage est pr�sent si une application divulgue cette condition d'erreur de remplissage sp�cifique pour les donn�es chiffr�es fournies par le client. Cela peut se produire en exposant directement des exceptions (par exemple, `BadPaddingException` en Java), par des diff�rences subtiles dans les r�ponses envoy�es au client ou par un autre canal lat�ral comme le comportement de synchronisation.

Certains modes de fonctionnement de la cryptographie permettent des attaques par retournement de bits, o� le retournement d'un bit dans le texte chiffr� fait que le bit est �galement retourn� dans le texte en clair. Le basculement d'un bit dans le n-i�me bloc de donn�es crypt�es CBC fait que le m�me bit dans le (n+1)-i�me bloc est bascul� dans les donn�es d�crypt�es. Le n-i�me bloc du texte chiffr� d�chiffr� est effac� par cette manipulation.

L'attaque oracle de remplissage permet � un attaquant de d�chiffrer des donn�es chiffr�es sans conna�tre la cl� de chiffrement et le chiffrement utilis� en envoyant des textes chiffr�s habilement manipul�s � l'oracle de remplissage et en observant les r�sultats renvoy�s par celui-ci. Cela entra�ne une perte de confidentialit� des donn�es crypt�es. Par exemple. dans le cas de donn�es de session stock�es c�t� client, l'attaquant peut obtenir des informations sur l'�tat interne et la structure de l'application.

Une attaque oracle de remplissage permet �galement � un attaquant de chiffrer des textes bruts arbitraires sans conna�tre la cl� et le chiffrement utilis�s. Si l'application suppose que l'int�grit� et l'authenticit� des donn�es d�chiffr�es sont donn�es, un attaquant pourrait �tre en mesure de manipuler l'�tat de la session interne et �ventuellement d'obtenir des privil�ges plus �lev�s.

## Objectifs des tests

- Identifiez les messages chiffr�s qui reposent sur le remplissage.
- Essayez de casser le rembourrage des messages chiffr�s et analysez les messages d'erreur renvoy�s pour une analyse plus approfondie.

## Comment tester

### Test de la bo�te noire

Tout d'abord, les points d'entr�e possibles pour les oracles de remplissage doivent �tre identifi�s. En r�gle g�n�rale, les conditions suivantes doivent �tre remplies�:

1. Les donn�es sont crypt�es. Les bons candidats sont des valeurs qui semblent al�atoires.
2. Un chiffrement par blocs est utilis�. La longueur du texte chiffr� d�cod� (Base64 est souvent utilis�) est un multiple des tailles de bloc de chiffrement courantes, comme 8 ou 16 octets. Diff�rents textes chiffr�s (par exemple, rassembl�s par diff�rentes sessions ou manipulation de l'�tat de la session) partagent un diviseur commun dans la longueur.

#### Exemple 2

`Dg6W8OiWMIdVokIDH15T/A==` donne apr�s d�codage Base64 `0e ??0e 96 f0 e8 96 30 87 55 a2 42 03 1f 5e 53 fc`. Cela semble �tre al�atoire et long de 16 octets.

Si un tel candidat de valeur d'entr�e est identifi�, le comportement de l'application � la falsification bit � bit de la valeur crypt�e doit �tre v�rifi�. Normalement, cette valeur encod�e en Base64 inclura le vecteur d'initialisation (IV) ajout� au texte chiffr�. �tant donn� un texte en clair *`p`* et un chiffrement avec une taille de bloc *`n`*, le nombre de blocs sera *`b = ceil( length(b) / n)`*. La longueur de la cha�ne chiffr�e sera *`y=(b+1)*n`* en raison du vecteur d'initialisation. Pour v�rifier la pr�sence de l'oracle, d�codez la cha�ne, retournez le dernier bit de l'avant-dernier bloc *`b-1`* (le bit le moins significatif de l'octet � *`y-n-1`*), re -encoder et envoyer. Ensuite, d�codez la cha�ne d'origine, retournez le dernier bit du bloc *`b-2`* (le bit le moins significatif de l'octet � *`y-2*n-1`*), r�encodez et envoyez.

Si l'on sait que la cha�ne crypt�e est un bloc unique (l'IV est stock� sur le serveur ou l'application utilise une mauvaise pratique d'IV cod�e en dur), plusieurs retournements de bits doivent �tre effectu�s � tour de r�le. Une autre approche pourrait consister � ajouter un bloc al�atoire au d�but et � inverser les bits afin que le dernier octet du bloc ajout� prenne toutes les valeurs possibles (0 � 255).

Les tests et la valeur de base doivent au moins provoquer trois �tats diff�rents pendant et apr�s le d�chiffrement�:

- Le texte chiffr� est d�chiffr�, les donn�es r�sultantes sont correctes.
- Le texte chiffr� est d�chiffr�, les donn�es r�sultantes sont brouill�es et provoquent une gestion des exceptions ou des erreurs dans la logique de l'application.
- Le d�chiffrement du texte chiffr� �choue en raison d'erreurs de remplissage.

Comparez soigneusement les r�ponses. Recherchez en particulier les exceptions et les messages qui indiquent que quelque chose ne va pas avec le rembourrage. Si de tels messages apparaissent, l'application contient un oracle de remplissage. Si les trois �tats diff�rents d�crits ci-dessus sont observables implicitement (diff�rents messages d'erreur, canaux lat�raux de synchronisation), il y a une forte probabilit� qu'un oracle de remplissage soit pr�sent � ce stade. Essayez d'effectuer l'attaque oracle de rembourrage pour vous en assurer.

##### Exemple 3

- ASP.NET l�ve `System.Security.Cryptography.CryptographicException: Padding is invalid and cannot be removed.` si le rembourrage d'un texte chiffr� d�chiffr� est rompu.
- En Java, une `javax.crypto.BadPaddingException` est lanc�e dans ce cas.
- Des erreurs de d�chiffrement ou similaires peuvent �tre des oracles de remplissage possibles.

> Une impl�mentation s�curis�e v�rifiera l'int�grit� et ne provoquera que deux r�ponses : `ok` et `failed`. Il n'y a pas de canaux lat�raux qui peuvent �tre utilis�s pour d�terminer les �tats d'erreur internes.

###�Test de la bo�te grise

V�rifiez que tous les endroits o� les donn�es chiffr�es du client, qui ne doivent �tre connues que du serveur, sont d�chiffr�es. Les conditions suivantes doivent �tre remplies par un tel code�:

1. L'int�grit� du texte chiffr� doit �tre v�rifi�e par un m�canisme s�curis�, comme HMAC ou des modes de fonctionnement chiffr�s authentifi�s comme GCM ou CCM.
2. Tous les �tats d'erreur lors du d�chiffrement et du traitement ult�rieur sont trait�s de mani�re uniforme.

### Exemple 4

[Visualisation du processus de d�cryptage](https://erlend.oftedal.no/blog/poet/)

## Outils

- [Bletchley](https://code.blindspotsecurity.com/trac/bletchley)
- [PadBuster](https://github.com/GDSSecurity/PadBuster)
- [Outil d'exploitation d'Oracle de rembourrage (POET)] (http://netifera.com/research/)
- [Poracle](https://github.com/iagox86/Poracle)
- [python-paddingoracle](https://github.com/mwielgoszewski/python-paddingoracle)

## R�f�rences

- [Wikip�dia - Padding Oracle Attack](https://en.wikipedia.org/wiki/Padding_oracle_attack)
- [Juliano Rizzo, Thai Duong, "Practical Padding Oracle Attacks"](https://www.usenix.org/event/woot10/tech/full_papers/Rizzo.pdf)
