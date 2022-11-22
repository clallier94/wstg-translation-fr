# Tester le stockage en nuage

|ID          |
|------------|
|WSTG-CONF-11|

## Sommaire

Les services de stockage en nuage facilitent l'application et les services Web pour stocker et acc�der aux objets dans le service de stockage. Cependant, une mauvaise configuration du contr�le d'acc�s peut entra�ner l'exposition d'informations sensibles, la falsification de donn�es ou un acc�s non autoris�.

Un exemple connu est celui o� un compartiment Amazon S3 est mal configur�, bien que les autres services de stockage en nuage puissent �galement �tre expos�s � des risques similaires. Par d�faut, tous les compartiments S3 sont priv�s et ne sont accessibles qu'aux utilisateurs auxquels l'acc�s est explicitement accord�. Les utilisateurs peuvent accorder un acc�s public au compartiment lui-m�me et aux objets individuels stock�s dans ce compartiment. Cela peut permettre � un utilisateur non autoris� de t�l�charger de nouveaux fichiers, de modifier ou de lire des fichiers stock�s.

## Objectifs des tests

- �valuer que la configuration du contr�le d'acc�s pour les services de stockage est correctement en place.

## Comment tester

Identifiez d'abord l'URL permettant d'acc�der aux donn�es dans le service de stockage, puis envisagez les tests suivants�:

- lire les donn�es non autoris�es
- t�l�charger un nouveau fichier arbitraire

Vous pouvez utiliser curl pour les tests avec les commandes suivantes et voir si des actions non autoris�es peuvent �tre effectu�es avec succ�s.

Pour tester la capacit� � lire un objet�:

```bash
curl -X GET https://<cloud-storage-service>/<object>
```

Pour tester la possibilit� de t�l�charger un fichier�:

```bash
curl -X PUT -d 'test' 'https://<cloud-storage-service>/test.txt'
```

### Test d'une mauvaise configuration du compartiment Amazon S3

Les URL de compartiment Amazon S3 suivent l'un des deux formats suivants, soit le style d'h�te virtuel, soit le style de chemin.

- Acc�s au style h�berg� virtuel

```text
https://bucket-name.s3.Region.amazonaws.com/key-name
```

Dans l'exemple suivant, "my-bucket" est le nom du bucket, "us-west-2" est la r�gion et "puppy.png" est le nom de la cl�:

```text
https://my-bucket.s3.us-west-2.amazonaws.com/puppy.png
```

- Acc�s au chemin

```text
https://s3.Region.amazonaws.com/bucket-name/key-name
```

Comme ci-dessus, dans l'exemple suivant, "my-bucket" est le nom du bucket, "us-west-2" est la r�gion et "puppy.png" est le nom de la cl�:

```text
https://s3.us-west-2.amazonaws.com/my-bucket/puppy.jpg
```

Pour certaines r�gions, le point de terminaison global h�rit� qui ne sp�cifie pas de point de terminaison sp�cifique � la r�gion peut �tre utilis�. Son format est �galement soit de style h�berg� virtuel, soit de style chemin.

- Acc�s au style h�berg� virtuel

```text
https://bucket-name.s3.amazonaws.com
```

- Acc�s au chemin

```text
https://s3.amazonaws.com/bucket-name
```

#### Identifier l'URL du compartiment

Pour les tests de bo�te noire, les URL S3 peuvent �tre trouv�es dans les messages HTTP. L'exemple suivant montre qu'une URL de compartiment est envoy�e dans la balise "img" d'une r�ponse HTTP.

```html
...
<img src="https://my-bucket.s3.us-west-2.amazonaws.com/puppy.png">
...
```

Pour les tests en bo�te grise, vous pouvez obtenir des URL de compartiment � partir de l'interface Web d'Amazon, des documents, du code source ou de toute autre source disponible.

#### Test avec AWS-CLI

En plus de tester avec curl, vous pouvez �galement tester avec l'outil de ligne de commande AWS. Dans ce cas, le protocole `s3://` est utilis�.

##### Liste

La commande suivante r�pertorie tous les objets du bucket lorsqu'il est configur� public.

```bash
aws s3 ls s3://<bucket-name>
```

##### T�l�charger

Voici la commande pour t�l�charger un fichier

```bash
aws s3 cp arbitrary-file s3://bucket-name/path-to-save
```

Cet exemple montre le r�sultat lorsque le t�l�chargement a r�ussi.

```bash
$ aws s3 cp test.txt s3://bucket-name/test.txt
upload: ./test.txt to s3://bucket-name/test.txt
```

Cet exemple montre le r�sultat lorsque le t�l�chargement a �chou�.

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
