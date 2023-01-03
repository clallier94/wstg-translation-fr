# Tester la fonctionnalité de paiement

|ID          |
|------------|
|WSTG-BUSL-10|

## Sommaire

De nombreuses applications implémentent des fonctionnalités de paiement, notamment des sites de commerce électronique, des abonnements, des organisations caritatives, des sites de dons et des bureaux de change. La sécurité de cette fonctionnalité est essentielle, car des vulnérabilités pourraient permettre à des attaquants de voler l'organisation, de faire des achats frauduleux ou même de voler les détails de la carte de paiement d'autres utilisateurs. Ces problèmes pourraient entraîner non seulement des dommages à la réputation de l'organisation, mais également des pertes financières importantes, à la fois des pertes directes et des amendes des régulateurs de l'industrie.

## Objectifs des tests

- Déterminer si la logique métier de la fonctionnalité de commerce électronique est robuste.
- Comprendre le fonctionnement de la fonctionnalité de paiement.
- Déterminer si la fonctionnalité de paiement est sécurisée.

## Comment tester

### Méthodes d'intégration de la passerelle de paiement

Les applications peuvent intégrer la fonctionnalité de paiement de différentes manières, et l'approche de test variera en fonction de celle qui est utilisée. Les méthodes les plus courantes sont :

- Rediriger l'utilisateur vers une passerelle de paiement tierce.
- Chargement d'une passerelle de paiement tiers dans un IFRAME sur l'application.
- Disposer d'un formulaire HTML qui effectue une requête POST inter-domaines vers une passerelle de paiement tierce.
- Accepter directement les détails de la carte, puis effectuer un POST depuis le backend de l'application vers l'API de la passerelle de paiement.

### PCI DSS

La norme de sécurité des données de l'industrie des cartes de paiement (PCI DSS) est une norme que les organisations sont tenues de suivre pour traiter les paiements par débit et par carte (bien qu'il soit important de noter qu'il ne s'agit pas d'une loi). Une discussion complète de cette norme sort du cadre de ce guide (et de la plupart des tests de pénétration) - mais il est utile pour les testeurs de comprendre quelques points clés.

L'idée fausse la plus courante concernant la norme PCI DSS est qu'elle ne s'applique qu'aux systèmes qui stockent les données des titulaires de cartes (c'est-à-dire les détails des cartes de débit ou de crédit). Ceci est incorrect : cela s'applique à tout système qui "stocke, traite ou transmet" ces informations. Les exigences exactes à respecter dépendent de la manière dont les méthodes d'intégration de la passerelle de paiement sont utilisées. Les [Conseils sur le traitement des paiements de commerce électronique par Visa](https://www.visa.co.uk/dam/VCOM/regional/ve/unitedkingdom/PDF/risk/processing-e-commerce-payments-guide-73-17337.pdf) fournit plus de détails à ce sujet, mais sous forme de bref résumé :

| Méthode d'intégration | Questionnaire d'auto-évaluation (SAQ) |
|--------------------|------------------------------------------|
| Redirect | [SAQ A](https://www.pcisecuritystandards.org/documents/PCI-DSS-v3_2_1-SAQ-A.pdf) |
| IFRAME | [SAQ A](https://www.pcisecuritystandards.org/documents/PCI-DSS-v3_2_1-SAQ-A.pdf) |
| Cross-domain POST | [SAQ A-EP](https://www.pcisecuritystandards.org/documents/PCI-DSS-v3_2-SAQ-A_EP-rev1_1.pdf) |
| Backend API | [SAQ D](https://www.pcisecuritystandards.org/documents/PCI-DSS-v3_2_1-SAQ-D_Merchant.pdf) |

Outre les différences dans la surface d'attaque et le profil de risque de chaque approche, il existe également une différence significative dans le nombre d'exigences entre SAQ A (22 exigences) et SAQ D (329 exigences) que l'organisation doit respecter. En tant que tel, il convient de mettre en évidence les applications qui n'utilisent pas de redirection ou d'IFRAME, car elles représentent des risques techniques et de conformité accrus.

### Modification de la quantité

La plupart des sites de commerce électronique permettent aux utilisateurs d'ajouter des articles à un panier avant de commencer le processus de paiement. Ce panier doit garder une trace des articles qui ont été ajoutés et de la quantité de chaque article. La quantité doit normalement être un nombre entier positif, mais si le site Web ne le valide pas correctement, il peut être possible de spécifier une quantité décimale d'un article (telle que `0.1`) ou une quantité négative (telle que `-1` ). Selon le traitement backend, l'ajout de quantités négatives d'un article peut entraîner une valeur négative, ce qui réduit le coût global du panier.

Il existe généralement plusieurs manières de modifier le contenu du panier qui doit être testé, telles que :

- Ajout d'une quantité négative d'un article.
- Suppression répétée d'articles jusqu'à ce que la quantité soit négative.
- Mise à jour de la quantité à une valeur négative.

Certains sites peuvent également fournir un menu déroulant de quantités valides (comme les articles qui doivent être achetés par pack de 10), et il peut être possible de falsifier ces demandes pour ajouter d'autres quantités d'articles.

Si les détails complets du panier sont transmis à la passerelle de paiement (plutôt que de simplement transmettre une valeur totale), il peut également être possible de falsifier les valeurs à ce stade.

Enfin, si l'application est vulnérable à la [pollution des paramètres HTTP](../07-Input_Validation_Testing/04-Testing_for_HTTP_Parameter_Pollution.md), il peut être possible de provoquer un comportement inattendu en passant plusieurs fois un paramètre, par exemple :

```http
POST /api/basket/add
Host: example.org

item_id=1&quantity=5&quantity=4
```

### Falsification des prix

#### Sur la demande

Lors de l'ajout d'un article au panier, l'application ne doit inclure que l'article et une quantité, comme dans l'exemple de demande ci-dessous :

```http
POST /api/basket/add HTTP/1.1
Host: example.org

item_id=1&quantity=5
```

Cependant, dans certains cas, l'application peut également inclure le prix, ce qui signifie qu'il peut être possible de le falsifier :

```http
POST /api/basket/add HTTP/1.1
Host: example.org

item_id=1&quantity=5&price=2.00
```

Différents types d'éléments peuvent avoir des règles de validation différentes, de sorte que chaque type doit être testé séparément. Certaines applications permettent également aux utilisateurs d'ajouter un don facultatif à une association caritative dans le cadre de leur achat, et ce don peut généralement être d'un montant arbitraire. Si ce montant n'est pas validé, il peut être possible d'ajouter un montant de don négatif, ce qui réduirait alors la valeur totale du panier.

#### Sur la passerelle de paiement

Si le processus de paiement est effectué sur une passerelle de paiement tierce, il peut être possible de falsifier les prix entre l'application et la passerelle.

Le transfert vers la passerelle peut être effectué à l'aide d'un POST interdomaine vers la passerelle, comme illustré dans l'exemple HTML ci-dessous.

> Remarque : Les détails de la carte ne sont pas inclus dans cette demande - l'utilisateur sera invité à les saisir sur la passerelle de paiement :

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

En modifiant le formulaire HTML ou en interceptant la requête POST, il peut être possible de modifier les prix des articles, et de les acheter effectivement moins cher. Notez que de nombreuses passerelles de paiement rejetteront une transaction avec une valeur de zéro, donc un total de 0,01 a plus de chances de réussir. Cependant, certaines passerelles de paiement peuvent accepter des valeurs négatives (utilisées pour traiter les remboursements). Lorsqu'il existe plusieurs valeurs (telles que les prix des articles, les frais d'expédition et le coût total du panier), toutes doivent être testées.

Si la passerelle de paiement utilise un IFRAME à la place, il peut être possible d'effectuer un type d'attaque similaire en modifiant l'URL IFRAME :

```html
<iframe src="https://example.org/payment_iframe?merchant_id=123&basket_total=22.00" />
```

> Remarque : Les passerelles de paiement sont généralement gérées par des tiers et, en tant que telles, peuvent ne pas être incluses dans la portée des tests. Cela signifie que si la falsification des prix peut être acceptable, d'autres types d'attaques (telles que l'injection SQL) ne doivent pas être effectuées sans approbation écrite explicite).

#### Détails de la transaction cryptée

Afin d'éviter que la transaction ne soit falsifiée, certaines passerelles de paiement crypteront les détails de la demande qui leur est faite. Par exemple, [Paypal](https://developer.paypal.com/api/nvp-soap/paypal-payments-standard/integration-guide/encryptedwebpayments/#link-usingewptoprotectmanuallycreatedpaymentbuttons) utilise la cryptographie à clé publique.

La première chose à essayer est de faire une demande non cryptée, car certaines passerelles de paiement autorisent les transactions non sécurisées à moins qu'elles n'aient été spécifiquement configurées pour les rejeter.

Si cela ne fonctionne pas, vous devez trouver la clé publique utilisée pour chiffrer les détails de la transaction, qui pourraient être exposés dans une sauvegarde de l'application, ou si vous pouvez trouver une vulnérabilité de traversée de répertoire.

Alternativement, il est possible que l'application réutilise la même paire de clés publique/privée pour la passerelle de paiement et son certificat numérique. Vous pouvez obtenir la clé publique du serveur avec la commande suivante :

```bash
echo -e '\0' | openssl s_client -connect example.org:443 2>/dev/null | openssl x509 -pubkey -noout
```

Une fois que vous avez cette clé, vous pouvez alors essayer de créer une demande cryptée (basée sur la documentation de la passerelle de paiement) et la soumettre à la passerelle pour voir si elle est acceptée.

#### Hachages sécurisés

D'autres passerelles de paiement utilisent un hachage sécurisé (ou un HMAC) des détails de la transaction pour empêcher toute falsification. Les détails exacts de la façon dont cela est fait varient selon les fournisseurs (par exemple, [Adyen](https://docs.adyen.com/online-payments/classic-integrations/hosted-payment-pages/hmac-signature-calculation) utilisez HMAC-SHA256), mais il inclura normalement les détails de la transaction et une valeur secrète. Par exemple, un hachage peut être calculé comme suit :

```php
$secure_hash = md5($merchant_id . $transaction_id . $items . $total_value . $secret)
```

Cette valeur est ensuite ajoutée à la requête POST envoyée à la passerelle de paiement et vérifiée pour s'assurer que la transaction n'a pas été falsifiée.

La première chose à essayer est de supprimer le hachage sécurisé, car certaines passerelles de paiement autorisent les transactions non sécurisées, sauf si une option de configuration spécifique a été définie.

La requête POST doit contenir toutes les valeurs requises pour calculer ce hachage, autres que la clé secrète. Cela signifie que si vous savez comment le hachage est calculé (ce qui devrait être inclus dans la documentation de la passerelle de paiement), vous pouvez tenter de forcer brutalement le secret. Alternativement, si le site Web exécute une application standard, il peut y avoir un secret par défaut dans les fichiers de configuration ou le code source. Enfin, si vous pouvez trouver une sauvegarde du site Web ou accéder aux fichiers de configuration, vous pourrez peut-être y trouver le secret.

Si vous pouvez obtenir ce secret, vous pouvez alors falsifier les détails de la transaction, puis générer votre propre hachage sécurisé qui sera accepté par la passerelle de paiement.

#### Falsification de devises

S'il n'est pas possible de modifier les prix réels, il peut être possible de modifier la devise utilisée, en particulier lorsque les applications prennent en charge plusieurs devises. Par exemple, l'application peut valider que le prix est de 10, mais si vous pouvez changer la devise pour payer 10 USD au lieu de 10 GBP, cela vous permettra d'acheter des articles moins cher.

#### Requêtes différées

Si la valeur des articles sur le site change au fil du temps (par exemple lors d'un échange de devises), il peut alors être possible d'acheter ou de vendre à un ancien prix en interceptant les demandes à l'aide d'un proxy local et en les retardant. Pour que cela soit exploitable, le prix devrait être soit inclus dans la demande, soit lié à quelque chose dans la demande (comme l'ID de session ou de transaction). L'exemple ci-dessous montre comment cela pourrait potentiellement être exploité sur une application permettant aux utilisateurs d'acheter et de vendre de l'or :

- Voir le prix actuel de l'or sur le site Web.
- Initier une demande d'achat pour 1 once d'or.
- Intercepter et geler la demande.
- Attendez une minute pour vérifier à nouveau le prix de l'or :
    - S'il augmente, laissez la transaction se terminer et achetez l'or à un prix inférieur à sa valeur actuelle.
    - S'il diminue, supprimez la requête request.

Si le site Web permet à l'utilisateur d'effectuer des paiements en utilisant des crypto-monnaies (qui sont généralement beaucoup plus volatiles), il peut être possible d'exploiter cela en obtenant un prix fixe dans cette crypto-monnaie, puis en attendant de voir si la valeur augmente ou diminue par rapport au devise principale utilisée par le site Web.

### Codes de réduction

Si l'application prend en charge les codes de réduction, différentes vérifications doivent être effectuées :

- Les codes sont-ils facilement devinables (TEST, TEST10, SORRY, SORRY10, nom de l'entreprise, etc.) ?
    - Si un code contient un nombre, peut-on trouver plus de codes en augmentant le nombre ?
- Existe-t-il une protection contre la force brute ?
- Plusieurs codes de réduction peuvent-ils être appliqués en même temps ?
- Les codes de réduction peuvent-ils être appliqués plusieurs fois ?
- Pouvez-vous [injecter des caractères génériques](../07-Input_Validation_Testing/05-Testing_for_SQL_Injection.md#sql-wildcard-injection) tels que `%` ou `*` ?
- Les codes de réduction sont-ils exposés dans la source HTML ou dans les champs `<input>` masqués n'importe où sur l'application ?

En plus de celles-ci, les vulnérabilités habituelles telles que l'injection SQL doivent être testées.

### Rupture des flux de paiement

Si le processus de paiement ou de paiement sur une application implique plusieurs étapes (telles que l'ajout d'articles à un panier, la saisie de codes de réduction, la saisie de détails d'expédition et la saisie d'informations de facturation), il peut être possible de provoquer un comportement imprévu en effectuant ces étapes en dehors de la séquence attendue. Par exemple, vous pouvez essayer :

- Modification de l'adresse de livraison après saisie des coordonnées de facturation pour réduire les frais de port.
- Suppression d'articles après avoir entré les détails d'expédition, pour éviter une valeur de panier minimum.
- Modification du contenu du panier après application d'un code de réduction.
- Modifier le contenu d'un panier après avoir terminé le processus de commande.

Il peut également être possible d'ignorer tout le processus de paiement pour la transaction. Par exemple, si l'application redirige vers une passerelle de paiement tierce, le flux de paiement peut être :

- L'utilisateur entre des détails sur l'application.
- L'utilisateur est redirigé vers la passerelle de paiement tiers.
- L'utilisateur saisit les détails de sa carte.
    - Si le paiement est réussi, ils sont redirigés vers `success.php` sur l'application.
    - Si le paiement échoue, ils sont redirigés vers `failure.php` sur l'application
- L'application met à jour sa base de données de commandes et traite la commande si elle a réussi.

Selon que l'application valide ou non le succès du paiement sur la passerelle, il peut être possible de forcer la navigation vers la page "success.php" (éventuellement en incluant un ID de transaction si nécessaire), ce qui entraînerait le site Web à traiter la commande comme si le paiement avait réussi. De plus, il peut être possible de faire des demandes répétées à la page `success.php` pour qu'une commande soit traitée plusieurs fois.

### Exploitation des frais de traitement des transactions

Les commerçants doivent normalement payer des frais pour chaque transaction traitée, qui se composent généralement d'une petite commission fixe et d'un pourcentage de la valeur totale. Cela signifie que recevoir de très petits paiements (tels que 0,01 $) peut entraîner une perte d'argent pour le commerçant, car les frais de traitement de la transaction sont supérieurs à la valeur totale de la transaction.

Ce problème est rarement exploitable sur les sites de commerce électronique, car le prix de l'article le moins cher est généralement suffisamment élevé pour l'éviter. Cependant, si le site Web permet aux clients d'effectuer des paiements avec des montants arbitraires (tels que des dons), vérifiez qu'il applique une valeur minimale raisonnable.

### Tester les cartes de paiement

La plupart des passerelles de paiement ont un ensemble de détails de carte de test définis, qui peuvent être utilisés par les développeurs lors des tests et du débogage. Celles-ci ne doivent être utilisables que sur les versions de développement ou bac à sable des passerelles, mais peuvent être acceptées sur les sites en ligne si elles ont été mal configurées.

Des exemples de ces détails de test pour diverses passerelles de paiement sont répertoriés ci-dessous :

- [Adyen - Numéros de carte de test](https://docs.adyen.com/development-resources/test-cards/test-card-numbers)
- [Globalpay - Cartes de test](https://developer.globalpay.com/resources/test-card-numbers)
- [Stripe - Numéros de carte de test de base](https://stripe.com/docs/testing#cards)
- [Worldpay - Numéros de carte de test](http://support.worldpay.com/support/kb/bg/testandgolive/tgl5103.html)

### Logistique des tests

Le test de la fonctionnalité de paiement sur les applications peut introduire une complexité supplémentaire, en particulier si un site en direct est en cours de test. Les domaines qui doivent être pris en compte incluent :

- Obtention des détails de paiement de la carte de test pour l'application.
    - Si celles-ci ne sont pas disponibles, il peut être possible d'obtenir une carte prépayée ou une alternative.
- Garder une trace de toutes les commandes passées afin qu'elles puissent être annulées et remboursées.
- Ne pas passer de commandes qui ne peuvent pas être annulées ou qui entraîneront d'autres actions (comme l'expédition immédiate de marchandises depuis un entrepôt).

## Cas de test associés

- [Test de la pollution des paramètres HTTP](../07-Input_Validation_Testing/04-Testing_for_HTTP_Parameter_Pollution.md)
- [Test pour l'injection SQL](../07-Input_Validation_Testing/05-Testing_for_SQL_Injection.md)
- [Test pour le contournement des flux de travail](06-Testing_for_the_Circumvention_of_Work_Flows.md)

## Correction

- Dans la mesure du possible, évitez de stocker, de transmettre ou de traiter les détails de la carte.
    - Utilisez une redirection ou IFRAME pour la passerelle de paiement.
- Examinez la documentation de la passerelle de paiement et utilisez toutes les fonctionnalités de sécurité disponibles (telles que le cryptage et les hachages sécurisés).
- Gérer toutes les informations relatives aux prix côté serveur :
    - Les seuls éléments inclus dans les demandes côté client doivent être les ID d'articles et les quantités.
- Mettre en oeuvre des contraintes de validation d'entrée et de logique métier appropriées (telles que la vérification des numéros ou des valeurs d'éléments négatifs).
- Assurez-vous que le flux de paiement de l'application est robuste et que les étapes ne peuvent pas être effectuées dans le désordre.

## Références

- [Norme de sécurité des données de l'industrie des cartes de paiement (PCI DSS)](https://www.pcisecuritystandards.org/documents/PCI_DSS_v3-2-1.pdf)
- [Conseils pour le traitement des paiements de commerce électronique par Visa](https://www.visa.co.uk/dam/VCOM/regional/ve/unitedkingdom/PDF/risk/processing-e-commerce-payments-guide-73-17337.pdf)
