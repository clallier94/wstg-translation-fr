# Injection codée

## Arrière plan

L'encodage des caractères est le processus de mappage des caractères, des nombres et d'autres symboles dans un format standard. En règle générale, cela est fait pour créer un message prêt à être transmis entre l'expéditeur et le destinataire. C'est, en termes simples, la conversion de caractères (appartenant à différentes langues comme l'anglais, le chinois, le grec ou toute autre langue connue) en octets. Un exemple de schéma de codage de caractères largement utilisé est le code standard américain pour l'échange d'informations (ASCII) qui utilisait initialement des codes à 7 bits. Des exemples plus récents de schémas de codage seraient les normes de l'industrie informatique Unicode "UTF-8" et "UTF-16".

Dans le domaine de la sécurité des applications et en raison de la pléthore de schémas de codage disponibles, le codage de caractères est souvent mal utilisé. Il est utilisé pour encoder des chaînes d'injection malveillantes de manière à les obscurcir. Cela peut conduire au contournement des filtres de validation d'entrée ou tirer parti de certaines manières dont les navigateurs restituent le texte codé.

## Codage d'entrée - Évasion du filtre

Les applications Web utilisent généralement différents types de mécanismes de filtrage des entrées pour limiter les entrées pouvant être soumises par l'utilisateur. Si ces filtres d'entrée ne sont pas suffisamment bien implémentés, il est possible de glisser un caractère ou deux à travers ces filtres. Par exemple, un `/` peut être représenté par `2F` (hex) en ASCII, tandis que le même caractère (`/`) est codé comme `C0` `AF` en Unicode (séquence de 2 octets). Par conséquent, il est important que la commande de filtrage d'entrée soit consciente du schéma de codage utilisé. S'il s'avère que le filtre ne détecte que des injections codées "UTF-8", un schéma de codage différent peut être utilisé pour contourner ce filtre.

## Encodage de sortie - Consensus du serveur et du navigateur

Les navigateurs Web doivent connaître le schéma de codage utilisé pour afficher de manière cohérente une page Web. Idéalement, ces informations devraient être fournies au navigateur dans le champ d'en-tête HTTP ("Content-Type"), comme indiqué ci-dessous :

```http
Content-Type: text/html; charset=UTF-8
```

ou via la balise HTML META (`META HTTP-EQUIV`), comme indiqué ci-dessous :

``` html
<META http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
```

C'est grâce à ces déclarations d'encodage de caractères que le navigateur comprend quel jeu de caractères utiliser lors de la conversion d'octets en caractères. Notez que le type de contenu mentionné dans l'en-tête HTTP a priorité sur la déclaration de balise META.

Le CERT le décrit ici comme suit :

De nombreuses pages Web laissent le codage des caractères (paramètre `charset` dans HTTP) indéfini. Dans les versions antérieures de HTML et HTTP, l'encodage des caractères était censé être par défaut "ISO-8859-1" s'il n'était pas défini. En fait, de nombreux navigateurs avaient une valeur par défaut différente, il n'était donc pas possible de se fier à la valeur par défaut `ISO-8859-1`. La version 4 de HTML le légitime - si l'encodage de caractères n'est pas spécifié, n'importe quel encodage de caractères peut être utilisé.

Si le serveur Web ne spécifie pas quel encodage de caractères est utilisé, il ne peut pas dire quels caractères sont spéciaux. Les pages Web avec un codage de caractères non spécifié fonctionnent la plupart du temps car la plupart des jeux de caractères attribuent les mêmes caractères aux valeurs d'octet inférieures à 128. Mais lesquelles des valeurs supérieures à 128 sont spéciales ? Certains schémas de `codage de caractères` 16 bits ont des représentations multi-octets supplémentaires pour les caractères spéciaux tels que `<`. Certains navigateurs reconnaissent cet encodage alternatif et agissent en conséquence. C'est un comportement "correct", mais il rend les attaques utilisant des scripts malveillants beaucoup plus difficiles à prévenir. Le serveur ne sait tout simplement pas quelles séquences d'octets représentent les caractères spéciaux.

Par conséquent, en cas de non-réception des informations d'encodage de caractères du serveur, le navigateur tente soit de deviner le schéma d'encodage, soit de revenir à un schéma par défaut. Dans certains cas, l'utilisateur définit explicitement le codage par défaut dans le navigateur sur un schéma différent. Une telle incompatibilité dans le schéma de codage utilisé par la page Web (serveur) et le navigateur peut amener le navigateur à interpréter la page d'une manière involontaire ou inattendue.

### Injections codées

Tous les scénarios donnés ci-dessous ne forment qu'un sous-ensemble des différentes façons dont l'obscurcissement peut être réalisé pour contourner les filtres d'entrée. De plus, le succès des injections codées dépend du navigateur utilisé. Par exemple, les injections encodées en "US-ASCII" ne réussissaient auparavant que dans le navigateur IE, mais pas dans Firefox. Par conséquent, on peut noter que les injections codées, dans une large mesure, dépendent du navigateur.

### Encodage de base

Considérez un filtre de validation d'entrée de base qui protège contre l'injection de guillemets simples. Dans ce cas, l'injection suivante contournerait facilement ce filtre :

``` html
<script>alert(String.fromCharCode(88,83,83))</script>
```

La fonction JavaScript `String.fromCharCode` prend les valeurs Unicode données et renvoie la chaîne correspondante. C'est l'une des formes les plus élémentaires d'injections codées. Un autre vecteur qui peut être utilisé pour contourner ce filtre est :

``` html
<IMG src="" onerror=javascript:alert(&quot;XSS&quot;)>
```

Ou en utilisant les [codes de caractères HTML](https://www.rapidtables.com/code/text/unicode-characters.html) respectifs :

``` html
<IMG src="" onerror="javascript:alert(&#34;XSS&#34;)">
```

Ce qui précède utilise des entités HTML pour construire la chaîne d'injection. L'encodage des entités HTML est utilisé pour afficher les caractères qui ont une signification particulière en HTML. Par exemple, `>` fonctionne comme un crochet fermant pour une balise HTML. Afin d'afficher réellement ce caractère sur la page Web, les entités de caractères HTML doivent être insérées dans la source de la page. Les injections mentionnées ci-dessus sont une manière d'encoder. Il existe de nombreuses autres manières d'encoder (obscurcir) une chaîne afin de contourner le filtre ci-dessus.

### Encodage hexadécimal

Hex, abréviation de Hexadecimal, est un système de numérotation en base 16, c'est-à-dire qu'il a 16 valeurs différentes de '0' à '9' et 'A' à 'F' pour représenter divers caractères. L'encodage hexadécimal est une autre forme d'obscurcissement qui est parfois utilisée pour contourner les filtres de validation d'entrée. Par exemple, la version encodée en hexadécimal de la chaîne `<IMG SRC=javascript:alert('XSS')>` est

``` html
<IMG SRC=%6A%61%76%61%73%63%72%69%70%74%3A%61%6C%65%72%74%28%27%58%53%53%27%29>
```

Une variante de la chaîne ci-dessus est donnée ci-dessous. Peut être utilisé dans le cas où '%' est filtré :

``` html
<IMG SRC=&#x6A&#x61&#x76&#x61&#x73&#x63&#x72&#x69&#x70&#x74&#x3A&#x61&#x6C&#x65&#x72&#x74&#x28&#x27&#x58&#x53&#x53&#x27&#x29>
```

Il existe d'autres schémas de codage, tels que Base64 et Octal, qui peuvent être utilisés pour l'obscurcissement. Bien que chaque schéma de codage puisse ne pas fonctionner à chaque fois, un peu d'essais et d'erreurs couplés à des manipulations intelligentes révéleraient certainement la faille dans un filtre de validation d'entrée faiblement construit.

### Encodage UTF-7

Encodage UTF-7 de

``` html
<SCRIPT>
    alert(‘XSS’);
</SCRIPT>
```

est comme ci-dessous

`+ADw-SCRIPT+AD4-alert('XSS');+ADw-/SCRIPT+AD4-`

Pour que le script ci-dessus fonctionne, le navigateur doit interpréter la page Web comme encodée en `UTF-7`.

### Encodage multi-octets

Le codage à largeur variable est un autre type de schéma de codage de caractères qui utilise des codes de longueurs variables pour coder les caractères. Le codage multi-octets est un type de codage à largeur variable qui utilise un nombre variable d'octets pour représenter un caractère. Le codage multi-octets est principalement utilisé pour coder des caractères appartenant à un grand jeu de caractères, par ex. chinois, japonais et coréen.

Le codage multioctet a été utilisé dans le passé pour contourner les fonctions de validation d'entrée standard et effectuer des scripts intersites et des attaques par injection SQL.

## Références

- [Encodage (Sémiotique)](https://en.wikipedia.org/wiki/Encoding_(sémiotique))
- [Entités HTML](https://www.w3schools.com/HTML/html_entities.asp)
- [Comment empêcher les attaques de validation d'entrée](https://searchsecurity.techtarget.com/answer/How-to-prevent-input-validation-attacks)
- [Unicode et jeux de caractères] (https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and- jeux-de-caractères-pas-d'excuses/)
