# Schémas de nommage des vulnérabilités

Avec un nombre sans cesse croissant d'actifs informatiques à administrer, les professionnels de la sécurité ont besoin d'outils nouveaux et plus puissants pour effectuer des analyses automatisées à grande échelle. Grâce au logiciel, l'attention peut être concentrée sur les problèmes les plus créatifs et intellectuellement difficiles. Malheureusement, faire communiquer des outils d'évaluation des vulnérabilités, des logiciels antivirus et des systèmes de détection d'intrusion n'est pas une tâche facile. Cela a entraîné plusieurs complications techniques, nécessitant une méthode standardisée pour identifier chaque défaut logiciel, vulnérabilité ou problème de configuration identifié. L'absence de ces capacités d'interopérabilité peut entraîner des incohérences lors de l'évaluation de la sécurité, des rapports confus et des efforts de corrélation supplémentaires, entre autres problèmes, qui entraîneront une importante perte de ressources et de temps.

Un schéma de nommage est une méthodologie systématique utilisée pour identifier chacune de ces vulnérabilités afin de faciliter une identification claire et le partage d'informations. Cet objectif est atteint par la définition d'un nom unique, structuré et logiciel efficace pour chaque vulnérabilité. Il existe plusieurs schémas utilisés pour faciliter cet effort, les plus courants sont :

- Énumération de plate-forme commune (`CPE`)
- Étiquette d'identification du logiciel (`SWID`)
- URL du paquet (`PURL`)

## Étiquette d'identification du logiciel

L'étiquette d'identification du logiciel (`SWID`) est une norme de l'Organisation internationale de normalisation définie par la norme ISO/IEC 19770-2:2015. Les balises `SWID` sont utilisées pour identifier clairement chaque logiciel dans le cadre de cycles de vie complets de gestion des actifs logiciels. Il est recommandé que ce schéma d'informations soit utilisé par le National Institute of Standards and Technology (NIST) comme identification principale pour tout logiciel développé ou installé. À partir de `SWID`, il est possible de générer d'autres schémas tels que le `CPE` utilisé par la National Vulnerability Database (NVD) alors que le processus inverse n'est pas possible.

Chaque balise `SWID` est représentée sous la forme d'un format XML standardisé. Une balise `SWID` est composée de trois groupes d'éléments. Le premier bloc composé de 7 éléments prédéfinis doit être considéré comme une balise valide. Suivi d'un bloc facultatif qui fournit un ensemble de 30 éléments prédéfinis possibles que le créateur de balises peut utiliser pour fournir des informations fiables et détaillées. Enfin, le groupe d'éléments "Étendu" offre la possibilité au créateur de la balise de définir tous les éléments non prédéfinis nécessaires pour définir avec précision le logiciel décrit. Le haut niveau de granularité fourni par `SWID` permet non seulement de décrire un produit logiciel donné, mais également son statut spécifique sur le cycle de vie du logiciel.

### Exemples

- Correctif _ACME Roadrunner Service Pack 1_ créé par ACME Corporation pour le produit déjà installé identifié par le `@tagId` : _com.acme.rms-ce-v4-1-5-0_ :

```xml
<SoftwareIdentity
                  xmlns="http://standards.iso.org/iso/19770/-2/2015/schema.xsd"
                  name="ACME Roadrunner Service Pack 1"
                  tagId="com.acme.rms-ce-sp1-v1-0-0"
                  patch="true"
                  version="1.0.0">
  <Entity
          name="The ACME Corporation"
          regid="acme.com"
          role="tagCreator softwareCreator"/>
  <Link
        rel="patches"
        href="swid:com.acme.rms-ce-v4-1-5-0">
    ...
</SoftwareIdentity>
```

- Red Hat Enterprise Linux version 8 pour architecture x86-64 :

```xml
<SoftwareIdentity
                  xmlns="http://standards.iso.org/iso/19770/-2/2015/schema.xsd"
                  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                  xsi:schemaLocation="http://standards.iso.org/iso/19770/-2/2015/schema.xsd"
                  xml:lang="en-US"
                  name="Red Hat Enterprise Linux"
                  tagId="com.redhat.RHEL-8-x86_64"
                  tagVersion="1"
                  version="8"
                  versionScheme="multipartnumeric"
                  media="(OS:linux)">
```

## Énumération de plate-forme commune

Le schéma Common Platform Enumeration (`CPE`) est un schéma de nommage structuré pour les systèmes informatiques, les logiciels et les packages gérés par `NVD`. Couramment utilisé conjointement avec les codes d'identification des vulnérabilités et expositions communes (par exemple, `CVE-2017-0147`). Bien qu'il soit considéré comme un schéma obsolète remplacé par `SWID`, `CPE` est encore largement utilisé par plusieurs solutions de sécurité.

Défini comme un dictionnaire de valeurs enregistrées fourni par `NVD`. Chaque code `CPE` peut être défini comme un nom bien formaté ou comme une URL. Chaque valeur DOIT suivre cette structure :

- _cpe-name_ = "cpe:" component-list
- _component-list_ = part ":" vendor ":" product ":" version ":" update ":" edition ":" lang
- _component-list_ = part ":" vendor ":" product ":" version ":" update ":" edition
- _component-list_ = part ":" vendor ":" product ":" version ":" update
- _component-list_ = part ":" vendor ":" product ":" version
- _component-list_ = part ":" vendor ":" product
- _component-list_ = part ":" vendor
- _component-list_ = part
- _component-list_ = empty
- _part_ = "h" / "o" / "a" = string
- _vendor_ = string
- _product_ = string
- _version_ = string
- _update_ = string
- _edition_ = string
- _lang_ LANGTAG / empty
- _string_ = *( unreserved / pct-encoded )
- _empty_ = ""
- _unreserved_ = ALPHA / DIGIT / "-" / "." / "_" / " ̃"
- _pct-encoded_ = "%" HEXDIG HEXDIG
- _ALPHA_ = %x41-5a / %x61-7a ; A-Z or a-z
- _DIGIT_ = %x30-39 ; 0-9
- _HEXDIG_ = DIGIT / "a" / "b" / "c" / "d" / "e" / "f"
- _LANGTAG_ = cf. [RFC5646]

### Exemples

- Microsoft Internet Explorer 8.0.6001 Beta (toute édition) : `wfn:[part="a",vendor="microsoft",product="internet_explorer", version="8\.0\.6001",update="beta",edition=ANY]`, qui se lie à l'URL suivante : `cpe:/a:microsoft:internet_explorer:8.0.6001:beta`.
- Édition spéciale Foo\Bar Big$Money Manager 2010 pour iPod Touch 80 Go : `wfn:[part="a",vendor="foo\\bar",product="big\$money_manager_2010", sw_edition="special",target_sw="ipod_touch",target_hw="80gb"]`, qui se lie à l'URL suivante : `cpe:/a:foo%5cbar:big%24money_manager_2010:::~~special~ipod_touch~80gb~`.

## URL du paquet

L'URL du package normalise la représentation des métadonnées du package logiciel afin que les packages puissent être localisés de manière universelle, quel que soit le fournisseur, le projet ou l'écosystème auquel les packages appartiennent.

Une PURL est une URL définie par une chaîne ASCII `RFC3986` valide composée de sept éléments. Chacun d'eux est séparé par un caractère défini afin de le rendre facilement manipulable par logiciel.

`scheme:type/namespace/name@version?qualifiers#subpath`

La définition de chaque composant est la suivante :

- _scheme_ : valeur constante conforme au schéma d'URL de "pkg". (**Obligatoire**).
- _type_ : type de package ou protocole de package tel que maven, npm, nuget, gem, pypi, etc. (**obligatoire**).
- _namespace_ : valeur spécifique au type d'un préfixe de package, tel que son nom de propriétaire, son identifiant de groupe, etc. (facultatif).
- _name_ : nom du paquet. (**Obligatoire**).
- _version_ : version du paquet. (Optionnel).
- _qualifiers_ : données de qualification supplémentaires pour un package tel qu'un système d'exploitation, une architecture, une distribution, etc. (facultatif).
- _subpath_ : sous-chemin supplémentaire dans un package, relatif à la racine du package. (Optionnel).

### Exemples

- Logiciel Curl, empaqueté sous la forme d'un paquet `.deb` pour Debian Jessie destiné à une architecture i386 : `pkg:deb/debian/curl@7.50.3-1?arch=i386&distro=jessie`
- Image Docker d'Apache Casandra signée avec le hachage SHA256 244fd47e07d1004f0aed9c : `pkg:docker/cassandra@sha256:244fd47e07d1004f0aed9c`

## Utilisations recommandées

| UTILISER | RECOMMANDATION |
|---|---|
| Application client ou serveur | CPE ou SWID |
| Conteneur | PURL ou SWID |
| Micrologiciel | CPE ou SWID* |
| Bibliothèque ou Framework (package) | PURL |
| Bibliothèque ou Framework (hors package) | SWID |
| Système d'exploitation | CPE ou SWID |
| Paquet de système d'exploitation | PURL ou SWID |

> Remarque : En raison du statut obsolète de `CPE`, l'industrie recommande que les nouveaux projets implémentent `SWID` lorsqu'ils doivent choisir entre les deux méthodes. Même si `CPE` est connu pour être un schéma de nommage largement utilisé dans les projets et solutions actifs actuels.

## Références

- [NISTIR 8060 - Directives pour la création d'étiquettes d'identification de logiciel interopérable (SWID) (pdf)](https://nvlpubs.nist.gov/nistpubs/ir/2016/NIST.IR.8060.pdf)
- [NISTIR 8085 - Forming Common Platform Enumeration (CPE) Names from Software Identification (SWID) Tags](https://csrc.nist.gov/CSRC/media/Publications/nistir/8085/draft/documents/nistir_8085_draft.pdf)
- [ISO/IEC 19770-2:2015 - Technologies de l'information—Gestion des actifs logiciels—Partie 2 : Étiquette d'identification du logiciel](https://www.iso.org/standard/65666.html)
- [Dictionnaire officiel de l'énumération de la plate-forme commune (CPE)](https://nvd.nist.gov/products/cpe)
- [Common Platform Enumeration : Dictionary Specification Version 2.3](https://csrc.nist.gov/publications/detail/nistir/7697/final)
- [Spécification PURL](https://github.com/package-url/purl-spec)

### Implémentations connues

- [packageurl-go](https://github.com/package-url/packageurl-go)
- [packageurl-dotnet](https://github.com/package-url/packageurl-dotnet)
- [packageurl-java](https://github.com/package-url/packageurl-java), [package-url-java](https://github.com/sonatype/package-url-java)
- [packageurl-python](https://github.com/package-url/packageurl-python)
- [packageurl-rust](https://github.com/package-url/packageurl.rs)
- [packageurl-js](https://github.com/package-url/packageurl-js)
