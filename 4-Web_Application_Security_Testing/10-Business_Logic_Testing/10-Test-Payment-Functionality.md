# Tester la fonctionnalit� de paiement

|ID          |
|------------|
|WSTG-BUSL-10|

## Sommaire

De nombreuses applications impl�mentent des fonctionnalit�s de paiement, notamment des sites de commerce �lectronique, des abonnements, des organisations caritatives, des sites de dons et des bureaux de change. La s�curit� de cette fonctionnalit� est essentielle, car des vuln�rabilit�s pourraient permettre � des attaquants de voler l'organisation, de faire des achats frauduleux ou m�me de voler les d�tails de la carte de paiement d'autres utilisateurs. Ces probl�mes pourraient entra�ner non seulement des dommages � la r�putation de l'organisation, mais �galement des pertes financi�res importantes, � la fois des pertes directes et des amendes des r�gulateurs de l'industrie.

## Objectifs des tests

- D�terminer si la logique m�tier de la fonctionnalit� de commerce �lectronique est robuste.
- Comprendre le fonctionnement de la fonctionnalit� de paiement.
- D�terminer si la fonctionnalit� de paiement est s�curis�e.

## Comment tester

### M�thodes d'int�gration de la passerelle de paiement

Les applications peuvent int�grer la fonctionnalit� de paiement de diff�rentes mani�res, et l'approche de test variera en fonction de celle qui est utilis�e. Les m�thodes les plus courantes sont :

- Rediriger l'utilisateur vers une passerelle de paiement tierce.
- Chargement d'une passerelle de paiement tiers dans un IFRAME sur l'application.
- Disposer d'un formulaire HTML qui effectue une requ�te POST inter-domaines vers une passerelle de paiement tierce.
- Accepter directement les d�tails de la carte, puis effectuer un POST depuis le backend de l'application vers l'API de la passerelle de paiement.

###�PCI�DSS

La norme de s�curit� des donn�es de l'industrie des cartes de paiement (PCI DSS) est une norme que les organisations sont tenues de suivre pour traiter les paiements par d�bit et par carte (bien qu'il soit important de noter qu'il ne s'agit pas d'une loi). Une discussion compl�te de cette norme sort du cadre de ce guide (et de la plupart des tests de p�n�tration) - mais il est utile pour les testeurs de comprendre quelques points cl�s.

L'id�e fausse la plus courante concernant la norme PCI DSS est qu'elle ne s'applique qu'aux syst�mes qui stockent les donn�es des titulaires de cartes (c'est-�-dire les d�tails des cartes de d�bit ou de cr�dit). Ceci est incorrect�: cela s'applique � tout syst�me qui "stocke, traite ou transmet" ces informations. Les exigences exactes � respecter d�pendent de la mani�re dont les m�thodes d'int�gration de la passerelle de paiement sont utilis�es. Les [Conseils sur le traitement des paiements de commerce �lectronique par Visa](https://www.visa.co.uk/dam/VCOM/regional/ve/unitedkingdom/PDF/risk/processing-e-commerce-payments-guide-73-17337 .pdf) fournit plus de d�tails � ce sujet, mais sous forme de bref r�sum�:

| M�thode d'int�gration | Questionnaire d'auto-�valuation (SAQ) |
|--------------------|------------------------------------------|
| Redirect | [SAQ A](https://www.pcisecuritystandards.org/documents/PCI-DSS-v3_2_1-SAQ-A.pdf) |
| IFRAME | [SAQ A](https://www.pcisecuritystandards.org/documents/PCI-DSS-v3_2_1-SAQ-A.pdf) |
| Cross-domain POST | [SAQ A-EP](https://www.pcisecuritystandards.org/documents/PCI-DSS-v3_2-SAQ-A_EP-rev1_1.pdf) |
| Backend API | [SAQ D](https://www.pcisecuritystandards.org/documents/PCI-DSS-v3_2_1-SAQ-D_Merchant.pdf) |

Outre les diff�rences dans la surface d'attaque et le profil de risque de chaque approche, il existe �galement une diff�rence significative dans le nombre d'exigences entre SAQ A (22 exigences) et SAQ D (329 exigences) que l'organisation doit respecter. En tant que tel, il convient de mettre en �vidence les applications qui n'utilisent pas de redirection ou d'IFRAME, car elles repr�sentent des risques techniques et de conformit� accrus.

### Modification de la quantit�

La plupart des sites de commerce �lectronique permettent aux utilisateurs d'ajouter des articles � un panier avant de commencer le processus de paiement. Ce panier doit garder une trace des articles qui ont �t� ajout�s et de la quantit� de chaque article. La quantit� doit normalement �tre un nombre entier positif, mais si le site Web ne le valide pas correctement, il peut �tre possible de sp�cifier une quantit� d�cimale d'un article (telle que "0,1") ou une quantit� n�gative (telle que "-1" ). Selon le traitement backend, l'ajout de quantit�s n�gatives d'un article peut entra�ner une valeur n�gative, ce qui r�duit le co�t global du panier.

Il existe g�n�ralement plusieurs mani�res de modifier le contenu du panier qui doit �tre test�, telles que :

- Ajout d'une quantit� n�gative d'un article.
- Suppression r�p�t�e d'articles jusqu'� ce que la quantit� soit n�gative.
- Mise � jour de la quantit� � une valeur n�gative.

Certains sites peuvent �galement fournir un menu d�roulant de quantit�s valides (comme les articles qui doivent �tre achet�s par pack de 10), et il peut �tre possible de falsifier ces demandes pour ajouter d'autres quantit�s d'articles.

Si les d�tails complets du panier sont transmis � la passerelle de paiement (plut�t que de simplement transmettre une valeur totale), il peut �galement �tre possible de falsifier les valeurs � ce stade.

Enfin, si l'application est vuln�rable � la [pollution des param�tres HTTP](../07-Input_Validation_Testing/04-Testing_for_HTTP_Parameter_Pollution.md), il peut �tre possible de provoquer un comportement inattendu en passant plusieurs fois un param�tre, par exemple�:

```http
POST /api/basket/add
Host: example.org

item_id=1&quantity=5&quantity=4
```

### Falsification des prix

#### Sur la demande

Lors de l'ajout d'un article au panier, l'application ne doit inclure que l'article et une quantit�, comme dans l'exemple de demande ci-dessous�:

```http
POST /api/basket/add HTTP/1.1
Host: example.org

item_id=1&quantity=5
```

Cependant, dans certains cas, l'application peut �galement inclure le prix, ce qui signifie qu'il peut �tre possible de le falsifier�:

```http
POST /api/basket/add HTTP/1.1
Host: example.org

item_id=1&quantity=5&price=2.00
```

Diff�rents types d'�l�ments peuvent avoir des r�gles de validation diff�rentes, de sorte que chaque type doit �tre test� s�par�ment. Certaines applications permettent �galement aux utilisateurs d'ajouter un don facultatif � une association caritative dans le cadre de leur achat, et ce don peut g�n�ralement �tre d'un montant arbitraire. Si ce montant n'est pas valid�, il peut �tre possible d'ajouter un montant de don n�gatif, ce qui r�duirait alors la valeur totale du panier.

#### Sur la passerelle de paiement

Si le processus de paiement est effectu� sur une passerelle de paiement tierce, il peut �tre possible de falsifier les prix entre l'application et la passerelle.

Le transfert vers la passerelle peut �tre effectu� � l'aide d'un POST interdomaine vers la passerelle, comme illustr� dans l'exemple HTML ci-dessous.

> Remarque : Les d�tails de la carte ne sont pas inclus dans cette demande - l'utilisateur sera invit� � les saisir sur la passerelle de paiement�:

```html
<form action="https://example.org/process_payment" method="POST">
    <input type="hidden" id="merchant_id" value="123" />
    <input type="hidden" id="basket_id" value="456" />
    <input type="hidden" id="item_id" value="1" />
    <input type="hidden" id="item_quantity" value="5" />
    <input type="hidden" id="item_total" value="20.00" />
    <input type="hidden" id="shipping_total" value="2.00" />
    <input type="hidden" id="basket_total" value="22.00" />
    <input type="hidden" id="currency" value="GBP" />
    <input type="submit" id="submit" value="submit" />
</form>
```

En modifiant le formulaire HTML ou en interceptant la requ�te POST, il peut �tre possible de modifier les prix des articles, et de les acheter effectivement moins cher. Notez que de nombreuses passerelles de paiement rejetteront une transaction avec une valeur de z�ro, donc un total de 0,01 a plus de chances de r�ussir. Cependant, certaines passerelles de paiement peuvent accepter des valeurs n�gatives (utilis�es pour traiter les remboursements). Lorsqu'il existe plusieurs valeurs (telles que les prix des articles, les frais d'exp�dition et le co�t total du panier), toutes doivent �tre test�es.

Si la passerelle de paiement utilise un IFRAME � la place, il peut �tre possible d'effectuer un type d'attaque similaire en modifiant l'URL IFRAME�:

```html
<iframe src="https://example.org/payment_iframe?merchant_id=123&basket_total=22.00" />
```

> Remarque�: Les passerelles de paiement sont g�n�ralement g�r�es par des tiers et, en tant que telles, peuvent ne pas �tre incluses dans la port�e des tests. Cela signifie que si la falsification des prix peut �tre acceptable, d'autres types d'attaques (telles que l'injection SQL) ne doivent pas �tre effectu�es sans approbation �crite explicite).

#### D�tails de la transaction crypt�e

Afin d'�viter que la transaction ne soit falsifi�e, certaines passerelles de paiement crypteront les d�tails de la demande qui leur est faite. Par exemple, [Paypal](https://developer.paypal.com/api/nvp-soap/paypal-payments-standard/integration-guide/encryptedwebpayments/#link-usingewptoprotectmanuallycreatedpaymentbuttons) utilise la cryptographie � cl� publique.

La premi�re chose � essayer est de faire une demande non crypt�e, car certaines passerelles de paiement autorisent les transactions non s�curis�es � moins qu'elles n'aient �t� sp�cifiquement configur�es pour les rejeter.

Si cela ne fonctionne pas, vous devez trouver la cl� publique utilis�e pour chiffrer les d�tails de la transaction, qui pourraient �tre expos�s dans une sauvegarde de l'application, ou si vous pouvez trouver une vuln�rabilit� de travers�e de r�pertoire.

Alternativement, il est possible que l'application r�utilise la m�me paire de cl�s publique/priv�e pour la passerelle de paiement et son certificat num�rique. Vous pouvez obtenir la cl� publique du serveur avec la commande suivante�:

```bash
echo -e '\0' | openssl s_client -connect example.org:443 2>/dev/null | openssl x509 -pubkey -noout
```

Une fois que vous avez cette cl�, vous pouvez alors essayer de cr�er une demande crypt�e (bas�e sur la documentation de la passerelle de paiement) et la soumettre � la passerelle pour voir si elle est accept�e.

#### Hachages s�curis�s

D'autres passerelles de paiement utilisent un hachage s�curis� (ou un HMAC) des d�tails de la transaction pour emp�cher toute falsification. Les d�tails exacts de la fa�on dont cela est fait varient selon les fournisseurs (par exemple, [Adyen](https://docs.adyen.com/online-payments/classic-integrations/hosted-payment-pages/hmac-signature-calculation) utilisez HMAC-SHA256), mais il inclura normalement les d�tails de la transaction et une valeur secr�te. Par exemple, un hachage peut �tre calcul� comme suit�:

```php
$secure_hash = md5($merchant_id . $transaction_id . $items . $total_value . $secret)
```

Cette valeur est ensuite ajout�e � la requ�te POST envoy�e � la passerelle de paiement et v�rifi�e pour s'assurer que la transaction n'a pas �t� falsifi�e.

La premi�re chose � essayer est de supprimer le hachage s�curis�, car certaines passerelles de paiement autorisent les transactions non s�curis�es, sauf si une option de configuration sp�cifique a �t� d�finie.

La requ�te POST doit contenir toutes les valeurs requises pour calculer ce hachage, autres que la cl� secr�te. Cela signifie que si vous savez comment le hachage est calcul� (ce qui devrait �tre inclus dans la documentation de la passerelle de paiement), vous pouvez tenter de forcer brutalement le secret. Alternativement, si le site Web ex�cute une application standard, il peut y avoir un secret par d�faut dans les fichiers de configuration ou le code source. Enfin, si vous pouvez trouver une sauvegarde du site Web ou acc�der aux fichiers de configuration, vous pourrez peut-�tre y trouver le secret.

Si vous pouvez obtenir ce secret, vous pouvez alors falsifier les d�tails de la transaction, puis g�n�rer votre propre hachage s�curis� qui sera accept� par la passerelle de paiement.

#### Falsification de devises

S'il n'est pas possible de modifier les prix r�els, il peut �tre possible de modifier la devise utilis�e, en particulier lorsque les applications prennent en charge plusieurs devises. Par exemple, l'application peut valider que le prix est de 10, mais si vous pouvez changer la devise pour payer 10 USD au lieu de 10 GBP, cela vous permettra d'acheter des articles moins cher.

#### Requ�tes diff�r�es

Si la valeur des articles sur le site change au fil du temps (par exemple lors d'un �change de devises), il peut alors �tre possible d'acheter ou de vendre � un ancien prix en interceptant les demandes � l'aide d'un proxy local et en les retardant. Pour que cela soit exploitable, le prix devrait �tre soit inclus dans la demande, soit li� � quelque chose dans la demande (comme l'ID de session ou de transaction). L'exemple ci-dessous montre comment cela pourrait potentiellement �tre exploit� sur une application permettant aux utilisateurs d'acheter et de vendre de l'or�:

- Voir le prix actuel de l'or sur le site Web.
- Initier une demande d'achat pour 1 once d'or.
- Intercepter et geler la demande.
- Attendez une minute pour v�rifier � nouveau le prix de l'or�:
    - S'il augmente, laissez la transaction se terminer et achetez l'or � un prix inf�rieur � sa valeur actuelle.
    - S'il diminue, supprimez la requ�te request.

Si le site Web permet � l'utilisateur d'effectuer des paiements en utilisant des crypto-monnaies (qui sont g�n�ralement beaucoup plus volatiles), il peut �tre possible d'exploiter cela en obtenant un prix fixe dans cette crypto-monnaie, puis en attendant de voir si la valeur augmente ou diminue par rapport au devise principale utilis�e par le site Web.

### Codes de r�duction

Si l'application prend en charge les codes de r�duction, diff�rentes v�rifications doivent �tre effectu�es�:

- Les codes sont-ils facilement devinables (TEST, TEST10, SORRY, SORRY10, nom de l'entreprise, etc.)�?
    - Si un code contient un nombre, peut-on trouver plus de codes en augmentant le nombre�?
- Existe-t-il une protection contre la force brute�?
- Plusieurs codes de r�duction peuvent-ils �tre appliqu�s en m�me temps�?
- Les codes de r�duction peuvent-ils �tre appliqu�s plusieurs fois ?
- Pouvez-vous [injecter des caract�res g�n�riques](../07-Input_Validation_Testing/05-Testing_for_SQL_Injection.md#sql-wildcard-injection) tels que `%` ou `*`�?
- Les codes de r�duction sont-ils expos�s dans la source HTML ou dans les champs `<input>` masqu�s n'importe o� sur l'application�?

En plus de celles-ci, les vuln�rabilit�s habituelles telles que l'injection SQL doivent �tre test�es.

### Rupture des flux de paiement

Si le processus de paiement ou de paiement sur une application implique plusieurs �tapes (telles que l'ajout d'articles � un panier, la saisie de codes de r�duction, la saisie de d�tails d'exp�dition et la saisie d'informations de facturation), il peut �tre possible de provoquer un comportement impr�vu en effectuant ces �tapes en dehors de la s�quence attendue. Par exemple, vous pouvez essayer�:

- Modification de l'adresse de livraison apr�s saisie des coordonn�es de facturation pour r�duire les frais de port.
- Suppression d'articles apr�s avoir entr� les d�tails d'exp�dition, pour �viter une valeur de panier minimum.
- Modification du contenu du panier apr�s application d'un code de r�duction.
- Modifier le contenu d'un panier apr�s avoir termin� le processus de commande.

Il peut �galement �tre possible d'ignorer tout le processus de paiement pour la transaction. Par exemple, si l'application redirige vers une passerelle de paiement tierce, le flux de paiement peut �tre�:

- L'utilisateur entre des d�tails sur l'application.
- L'utilisateur est redirig� vers la passerelle de paiement tiers.
- L'utilisateur saisit les d�tails de sa carte.
    - Si le paiement est r�ussi, ils sont redirig�s vers `success.php` sur l'application.
    - Si le paiement �choue, ils sont redirig�s vers `failure.php` sur l'application
- L'application met � jour sa base de donn�es de commandes et traite la commande si elle a r�ussi.

Selon que l'application valide ou non le succ�s du paiement sur la passerelle, il peut �tre possible de forcer la navigation vers la page "success.php" (�ventuellement en incluant un ID de transaction si n�cessaire), ce qui entra�nerait le site Web � traiter la commande comme si le paiement avait r�ussi. De plus, il peut �tre possible de faire des demandes r�p�t�es � la page `success.php` pour qu'une commande soit trait�e plusieurs fois.

### Exploitation des frais de traitement des transactions

Les commer�ants doivent normalement payer des frais pour chaque transaction trait�e, qui se composent g�n�ralement d'une petite commission fixe et d'un pourcentage de la valeur totale. Cela signifie que recevoir de tr�s petits paiements (tels que 0,01 $) peut entra�ner une perte d'argent pour le commer�ant, car les frais de traitement de la transaction sont sup�rieurs � la valeur totale de la transaction.

Ce probl�me est rarement exploitable sur les sites de commerce �lectronique, car le prix de l'article le moins cher est g�n�ralement suffisamment �lev� pour l'�viter. Cependant, si le site Web permet aux clients d'effectuer des paiements avec des montants arbitraires (tels que des dons), v�rifiez qu'il applique une valeur minimale raisonnable.

### Tester les cartes de paiement

La plupart des passerelles de paiement ont un ensemble de d�tails de carte de test d�finis, qui peuvent �tre utilis�s par les d�veloppeurs lors des tests et du d�bogage. Celles-ci ne doivent �tre utilisables que sur les versions de d�veloppement ou bac � sable des passerelles, mais peuvent �tre accept�es sur les sites en ligne si elles ont �t� mal configur�es.

Des exemples de ces d�tails de test pour diverses passerelles de paiement sont r�pertori�s ci-dessous�:

- [Adyen - Num�ros de carte de test](https://docs.adyen.com/development-resources/test-cards/test-card-numbers)
- [Globalpay - Cartes de test](https://developer.globalpay.com/resources/test-card-numbers)
- [Stripe - Num�ros de carte de test de base] (https://stripe.com/docs/testing#cards)
- [Worldpay - Num�ros de carte de test](http://support.worldpay.com/support/kb/bg/testandgolive/tgl5103.html)

### Logistique des tests

Le test de la fonctionnalit� de paiement sur les applications peut introduire une complexit� suppl�mentaire, en particulier si un site en direct est en cours de test. Les domaines qui doivent �tre pris en compte incluent :

- Obtention des d�tails de paiement de la carte de test pour l'application.
    - Si celles-ci ne sont pas disponibles, il peut �tre possible d'obtenir une carte pr�pay�e ou une alternative.
- Garder une trace de toutes les commandes pass�es afin qu'elles puissent �tre annul�es et rembours�es.
- Ne pas passer de commandes qui ne peuvent pas �tre annul�es ou qui entra�neront d'autres actions (comme l'exp�dition imm�diate de marchandises depuis un entrep�t).

## Cas de test associ�s

- [Test de la pollution des param�tres HTTP](../07-Input_Validation_Testing/04-Testing_for_HTTP_Parameter_Pollution.md)
- [Test pour l'injection SQL](../07-Input_Validation_Testing/05-Testing_for_SQL_Injection.md)
- [Test pour le contournement des flux de travail](06-Testing_for_the_Circumvention_of_Work_Flows.md)

## Correction

- Dans la mesure du possible, �vitez de stocker, de transmettre ou de traiter les d�tails de la carte.
    - Utilisez une redirection ou IFRAME pour la passerelle de paiement.
- Examinez la documentation de la passerelle de paiement et utilisez toutes les fonctionnalit�s de s�curit� disponibles (telles que le cryptage et les hachages s�curis�s).
- G�rer toutes les informations relatives aux prix c�t� serveur�:
    - Les seuls �l�ments inclus dans les demandes c�t� client doivent �tre les ID d'articles et les quantit�s.
- Mettre en �uvre des contraintes de validation d'entr�e et de logique m�tier appropri�es (telles que la v�rification des num�ros ou des valeurs d'�l�ments n�gatifs).
- Assurez-vous que le flux de paiement de l'application est robuste et que les �tapes ne peuvent pas �tre effectu�es dans le d�sordre.

## R�f�rences

- [Norme de s�curit� des donn�es de l'industrie des cartes de paiement (PCI DSS)](https://www.pcisecuritystandards.org/documents/PCI_DSS_v3-2-1.pdf)
- [Conseils pour le traitement des paiements de commerce �lectronique par Visa](https://www.visa.co.uk/dam/VCOM/regional/ve/unitedkingdom/PDF/risk/processing-e-commerce-payments-guide-73-17337.pdf)
