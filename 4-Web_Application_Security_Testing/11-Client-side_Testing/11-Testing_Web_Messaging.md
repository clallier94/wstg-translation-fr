# Tester la messagerie Web

|ID          |
|------------|
|WSTG-CLNT-11|

## Sommaire

La messagerie Web (�galement connue sous le nom de [messagerie crois�e](https://html.spec.whatwg.org/multipage/web-messaging.html#web-messaging)) permet aux applications ex�cut�es sur diff�rents domaines de communiquer de mani�re s�curis�e. Avant l'introduction de la messagerie Web, la communication d'origines diff�rentes (entre les iframes, les onglets et les fen�tres) �tait restreinte par la m�me politique d'origine et impos�e par le navigateur. Les d�veloppeurs ont utilis� plusieurs hacks pour accomplir ces t�ches, et la plupart d'entre eux �taient principalement non s�curis�s.

Cette restriction dans le navigateur est en place pour emp�cher un site Web malveillant de lire des donn�es confidentielles � partir d'autres iframes, onglets, etc.�; cependant, il existe des cas l�gitimes o� deux sites Web de confiance doivent �changer des donn�es entre eux. Pour r�pondre � ce besoin, la messagerie interdocuments a �t� introduite dans le projet de sp�cification [WHATWG HTML5](https://html.spec.whatwg.org/multipage/) et a �t� impl�ment�e dans tous les principaux navigateurs. Il permet des communications s�curis�es entre plusieurs origines � travers des iframes, des onglets et des fen�tres.

L'API de messagerie a introduit la [m�thode `postMessage()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage), avec laquelle des messages en texte brut peuvent �tre envoy�s origine. Il se compose de deux param�tres�: message et domaine.

Il existe des probl�mes de s�curit� lors de l'utilisation de `*` comme domaine dont nous parlerons ci-dessous. Pour recevoir des messages, le site Web destinataire doit ajouter un nouveau gestionnaire d'�v�nements, qui poss�de les attributs suivants�:

- Donn�es, le contenu du message entrant ;
- Origine du document exp�diteur ; et
- Source, la fen�tre source.

Voici un exemple de l'API de messagerie utilis�e. Pour envoyer un message�:

```js
iframe1.contentWindow.postMessage("Hello world","http://www.exemple.com");
```

Pour recevoir un message�:

```js
window.addEventListener("message", handler, true);
function handler(event) {
    if(event.origin === 'chat.exemple.com') {
        /* process message (event.data) */
    } else {
        /* ignore messages from untrusted domains */
    }
}
```

### S�curit� d'origine

L'origine est compos�e d'un sch�ma, d'un nom d'h�te et d'un port. Il identifie de mani�re unique le domaine qui envoie ou re�oit le message et n'inclut pas le chemin ou la partie fragment de l'URL. Par exemple, `https://exemple.com` sera consid�r� comme diff�rent de `http://exemple.com` car le sch�ma du premier est `https`, tandis que le second est `http`. Cela s'applique �galement aux serveurs Web ex�cut�s dans le m�me domaine mais sur des ports diff�rents.

## Objectifs des tests

- �valuer la s�curit� de l'origine du message.
- Validez qu'il utilise des m�thodes s�res et validez son entr�e.

## Comment tester

### Examiner la s�curit� de l'origine

Les testeurs doivent v�rifier si le code de l'application filtre et traite les messages provenant de domaines de confiance. Dans le domaine d'envoi, assurez-vous �galement que le domaine de r�ception est explicitement indiqu� et que `*` n'est pas utilis� comme deuxi�me argument de `postMessage()`. Cette pratique pourrait introduire des probl�mes de s�curit� et pourrait conduire, dans le cas d'une redirection ou si l'origine change par d'autres moyens, � ce que le site Web envoie des donn�es � des h�tes inconnus, et donc � des fuites de donn�es confidentielles vers des serveurs malveillants.

Si le site Web ne parvient pas � ajouter des contr�les de s�curit� pour restreindre les domaines ou les origines autoris�s � envoyer des messages � un site Web, il est susceptible d'introduire un risque de s�curit�. Les testeurs doivent examiner le code des �couteurs d'�v�nements de message et obtenir la fonction de rappel de la m�thode "addEventListener" pour une analyse plus approfondie. Les domaines doivent toujours �tre v�rifi�s avant la manipulation des donn�es.

### Examiner la validation des entr�es

Bien que le site Web n'accepte th�oriquement que les messages provenant de domaines de confiance, les donn�es doivent toujours �tre trait�es comme des donn�es non fiables de source externe et trait�es avec les contr�les de s�curit� appropri�s. Les testeurs doivent analyser le code et rechercher des m�thodes non s�curis�es, en particulier lorsque les donn�es sont �valu�es via `eval()` ou ins�r�es dans le DOM via la propri�t� `innerHTML`, ce qui peut cr�er des vuln�rabilit�s XSS bas�es sur DOM.

### Analyse de code statique

Le code JavaScript doit �tre analys� pour d�terminer comment la messagerie Web est impl�ment�e. En particulier, les testeurs doivent s'int�resser � la mani�re dont le site Web limite les messages provenant de domaines non approuv�s et � la mani�re dont les donn�es sont g�r�es m�me pour les domaines de confiance.

Dans cet exemple, un acc�s est n�cessaire pour chaque sous-domaine (www, chat, forums, ...) du domaine owasp.org. Le code essaie d'accepter n'importe quel domaine avec `.owasp.org`�:

```js
window.addEventListener("message", callback, true);

function callback(e) {
    if(e.origin.indexOf(".owasp.org")!=-1) {
        /* process message (e.data) */
    }
}
```

L'intention est d'autoriser des sous-domaines tels que�:

- `www.owasp.org`
- `chat.owasp.org`
- `forums.owasp.org`

Malheureusement, cela introduit des vuln�rabilit�s. Un attaquant peut facilement contourner le filtre car un domaine tel que `www.owasp.org.attacker.com` correspondra.

Voici un exemple de code qui n'a pas de v�rification d'origine. Ceci est tr�s peu s�r, car il acceptera les entr�es de n'importe quel domaine�:

```js
window.addEventListener("message", callback, true);

function callback(e) {
        /* process message (e.data) */
}
```

Voici un exemple avec des vuln�rabilit�s de validation d'entr�e pouvant conduire � une attaque XSS�:

```js
window.addEventListener("message", callback, true);

function callback(e) {
        if(e.origin === "trusted.domain.com") {
            element.innerHTML= e.data;
        }
}
```

Une approche plus s�re consisterait � utiliser la propri�t� `innerText` au lieu de `innerHTML`.

Pour plus de ressources OWASP concernant la messagerie Web, voir [OWASP HTML5 Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html)
