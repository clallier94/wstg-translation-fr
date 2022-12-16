# Tester le stockage en nuage

|ID          |
|------------|
|WSTG-CONF-11|

## Sommaire

Les services de stockage en nuage facilitent l'application et les services Web pour stocker et accéder aux objets dans le service de stockage. Cependant, une mauvaise configuration du contrôle d'accès peut entraîner l'exposition d'informations sensibles, la falsification de données ou un accès non autorisé.

Un exemple connu est celui où un compartiment Amazon S3 est mal configuré, bien que les autres services de stockage en nuage puissent également être exposés à des risques similaires. Par défaut, tous les compartiments S3 sont privés et ne sont accessibles qu'aux utilisateurs auxquels l'accès est explicitement accordé. Les utilisateurs peuvent accorder un accès public au compartiment lui-même et aux objets individuels stockés dans ce compartiment. Cela peut permettre à un utilisateur non autorisé de télécharger de nouveaux fichiers, de modifier ou de lire des fichiers stockés.

## Objectifs des tests

- Évaluer que la configuration du contrôle d'accès pour les services de stockage est correctement en place.

## Comment tester

Identifiez d'abord l'URL permettant d'accéder aux données dans le service de stockage, puis envisagez les tests suivants :

- lire les données non autorisées
- télécharger un nouveau fichier arbitraire

Vous pouvez utiliser curl pour les tests avec les commandes suivantes et voir si des actions non autorisées peuvent être effectuées avec succès.

Pour tester la capacité à lire un objet :

```bash
curl -X GET https://<cloud-storage-service>/<object>
```

Pour tester la possibilité de télécharger un fichier :

```bash
curl -X PUT -d 'test' 'https://<cloud-storage-service>/test.txt'
```

### Test d'une mauvaise configuration du compartiment Amazon S3

Les URL de compartiment Amazon S3 suivent l'un des deux formats suivants, soit le style d'hôte virtuel, soit le style de chemin.

- Accès au style hébergé virtuel

```text
https://bucket-name.s3.Region.amazonaws.com/key-name
```

Dans l'exemple suivant, `my-bucket` est le nom du bucket, `us-west-2` est la région et `puppy.png` est le nom de la clé :

```text
https://my-bucket.s3.us-west-2.amazonaws.com/puppy.png
```

- Accès au chemin

```text
https://s3.Region.amazonaws.com/bucket-name/key-name
```

Comme ci-dessus, dans l'exemple suivant, `my-bucket` est le nom du bucket, `us-west-2` est la région et `puppy.png` est le nom de la clé :

```text
https://s3.us-west-2.amazonaws.com/my-bucket/puppy.jpg
```

Pour certaines régions, le point de terminaison global hérité qui ne spécifie pas de point de terminaison spécifique à la région peut être utilisé. Son format est également soit de style hébergé virtuel, soit de style chemin.

- Accès au style hébergé virtuel

```text
https://bucket-name.s3.amazonaws.com
```

- Accès au chemin

```text
https://s3.amazonaws.com/bucket-name
```

#### Identifier l'URL du compartiment

Pour les tests de boîte noire, les URL S3 peuvent être trouvées dans les messages HTTP. L'exemple suivant montre qu'une URL de compartiment est envoyée dans la balise `img` d'une réponse HTTP.

```html
...
<img src="https://my-bucket.s3.us-west-2.amazonaws.com/puppy.png">
...
```

Pour les tests en boîte grise, vous pouvez obtenir des URL de compartiment à partir de l'interface Web d'Amazon, des documents, du code source ou de toute autre source disponible.

#### Test avec AWS-CLI

En plus de tester avec curl, vous pouvez également tester avec l'outil de ligne de commande AWS. Dans ce cas, le protocole `s3://` est utilisé.

##### Liste

La commande suivante répertorie tous les objets du bucket lorsqu'il est configuré public.

```bash
aws s3 ls s3://<bucket-name>
```

##### Télécharger

Voici la commande pour télécharger un fichier

```bash
aws s3 cp arbitrary-file s3://bucket-name/path-to-save
```

Cet exemple montre le résultat lorsque le téléchargement a réussi.

```bash
$ aws s3 cp test.txt s3://bucket-name/test.txt
upload: ./test.txt to s3://bucket-name/test.txt
```

Cet exemple montre le résultat lorsque le téléchargement a échoué.

```bash
$ aws s3 cp test.txt s3://bucket-name/test.txt
upload failed: ./test2.txt to s3://bucket-name/test2.txt An error occurred (AccessDenied) when calling the PutObject operation: Access Denied
```

##### Retirer

Voici la commande pour supprimer un objet

```bash
aws s3 rm s3://bucket-name/object-to-remove
```

## Outils

- [AWS CLI](https://aws.amazon.com/cli/)

## References

- [Utilisation des Buckets Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingBucket.html)
- [flAWS 2](http://flaws2.cloud)
