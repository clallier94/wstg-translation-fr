# Test du stockage du navigateur

|ID |
|------------|
|WSTG-CLNT-12|

## Sommaire

Les navigateurs fournissent les m�canismes de stockage c�t� client suivants permettant aux d�veloppeurs de stocker et de r�cup�rer des donn�es�:

- Stockage local
- Stockage des sessions
- IndexedDB
- Web SQL (obsol�te)
- Biscuits

Ces m�canismes de stockage peuvent �tre visualis�s et modifi�s � l'aide des outils de d�veloppement du navigateur, tels que [Google Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools/storage/localstorage) ou [Firefox's Storage Inspector] (https://developer.mozilla.org/en-US/docs/Tools/Storage_Inspector).

Remarque : bien que le cache soit �galement une forme de stockage, il est couvert dans une [section s�par�e](../04-Authentication_Testing/06-Testing_for_Browser_Cache_Weaknesses.md) couvrant ses propres particularit�s et pr�occupations.

## Objectifs des tests

- D�terminez si le site Web stocke des donn�es sensibles dans le stockage c�t� client.
- La gestion du code des objets de stockage doit �tre examin�e pour les possibilit�s d'attaques par injection, telles que l'utilisation d'entr�es non valid�es ou de biblioth�ques vuln�rables.

## Comment tester

### Stockage local

`window.localStorage` est une propri�t� globale qui impl�mente [Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API) et fournit une cl�-valeur **persistante** stockage dans le navigateur.

Les cl�s et les valeurs ne peuvent �tre que des cha�nes, donc toutes les valeurs non-cha�nes doivent d'abord �tre converties en cha�nes avant de les stocker, g�n�ralement via [JSON.stringify](https://developer.mozilla.org/en-US/docs /Web/JavaScript/Reference/Global_Objects/JSON/stringify).

Les entr�es dans `localStorage` persistent m�me lorsque la fen�tre du navigateur se ferme, � l'exception des fen�tres en mode priv�/incognito.

La capacit� de stockage maximale de `localStorage` varie selon les navigateurs.

#### Lister toutes les entr�es de valeur-cl�

```javascript
for (let i = 0; i < localStorage.length; i++) {
  const key = localStorage.key(i);
  const value = localStorage.getItem(key);
  console.log(`${key}: ${value}`);
}
```

###�Stockage des sessions

`window.sessionStorage` est une propri�t� globale qui impl�mente [Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API) et fournit une cl�-valeur **�ph�m�re** stockage dans le navigateur.

Les cl�s et les valeurs ne peuvent �tre que des cha�nes, donc toutes les valeurs non-cha�nes doivent d'abord �tre converties en cha�nes avant de les stocker, g�n�ralement via [JSON.stringify](https://developer.mozilla.org/en-US/docs /Web/JavaScript/Reference/Global_Objects/JSON/stringify).

Les entr�es de `sessionStorage` sont �ph�m�res car elles sont effac�es lorsque l'onglet/la fen�tre du navigateur est ferm�.

La capacit� de stockage maximale de `sessionStorage` varie selon les navigateurs.

#### Lister toutes les entr�es de valeur-cl�

```javascript
for (let i = 0; i < sessionStorage.length; i++) {
  const key = sessionStorage.key(i);
  const value = sessionStorage.getItem(key);
  console.log(`${key}: ${value}`);
}
```

### IndexedDB

IndexedDB est une base de donn�es transactionnelle orient�e objet destin�e aux donn�es structur�es. Une base de donn�es IndexedDB peut avoir plusieurs magasins d'objets et chaque magasin d'objets peut avoir plusieurs objets.

Contrairement au stockage local et au stockage de session, IndexedDB peut stocker plus que de simples cha�nes. Tous les objets pris en charge par [l'algorithme de clonage structur�](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm) peuvent �tre stock�s dans IndexedDB.

[CryptoKeys](https://developer.mozilla.org/en-US/docs/Web/API/CryptoKey) est un exemple d'objet JavaScript complexe pouvant �tre stock� dans IndexedDB, mais pas dans Local/Session Storage.

Recommandation du W3C sur [Web Crypto API](https://www.w3.org/TR/WebCryptoAPI/) [recommande](https://www.w3.org/TR/WebCryptoAPI/#concepts-key-storage) qui Cl�s de chiffrement qui doivent �tre conserv�es dans le navigateur, pour �tre stock�es dans IndexedDB. Lors du test d'une page Web, recherchez toutes les cl�s de chiffrement dans IndexedDB et v�rifiez si elles sont d�finies sur "extractable�: true" alors qu'elles auraient d� �tre d�finies sur "extractable�: false" (c'est-�-dire que la cl� priv�e sous-jacente n'est jamais expos�e pendant les op�rations cryptographiques .)

#### Imprimer tout le contenu de IndexedDB

```javascript
const dumpIndexedDB = dbName => {
  const DB_VERSION = 1;
  const req = indexedDB.open(dbName, DB_VERSION);
  req.onsuccess = function() {
    const db = req.result;
    const objectStoreNames = db.objectStoreNames || [];

    console.log(`[*] Database: ${dbName}`);

    Array.from(objectStoreNames).forEach(storeName => {
      const txn = db.transaction(storeName, 'readonly');
      const objectStore = txn.objectStore(storeName);

      console.log(`\t[+] ObjectStore: ${storeName}`);

      // Print all entries in objectStore with name `storeName`
      objectStore.getAll().onsuccess = event => {
        const items = event.target.result || [];
        items.forEach(item => console.log(`\t\t[-] `, item));
      };
    });
  };
};

indexedDB.databases().then(dbs => dbs.forEach(db => dumpIndexedDB(db.name)));
```

### SQL Web

Web SQL est obsol�te depuis le 18 novembre 2010 et il est recommand� aux d�veloppeurs Web de ne pas l'utiliser.

### Biscuits

Les cookies sont un m�canisme de stockage cl�-valeur qui est principalement utilis� pour la gestion de session, mais les d�veloppeurs Web peuvent toujours l'utiliser pour stocker des donn�es de cha�ne arbitraires.

Les cookies sont trait�s en d�tail dans le sc�nario [test des attributs des cookies](../06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md).

#### Lister tous les cookies

```javascript
console.log(window.document.cookie);
```

### Objet de fen�tre globale

Parfois, les d�veloppeurs Web initialisent et maintiennent un �tat global qui n'est disponible que pendant la dur�e d'ex�cution de la page en attribuant des attributs personnalis�s � l'objet global "window". Par exemple:

```javascript
window.MY_STATE = {
  counter: 0,
  flag: false,
};
```

Toutes les donn�es attach�es � l'objet `window` seront perdues lorsque la page est actualis�e ou ferm�e.

#### Lister toutes les entr�es de l'objet fen�tre

```javascript
(() => {
  // create an iframe and append to body to load a clean window object
  const iframe = document.createElement('iframe');
  iframe.style.display = 'none';
  document.body.appendChild(iframe);

  // get the current list of properties on window
  const currentWindow = Object.getOwnPropertyNames(window);

  // filter the list against the properties that exist in the clean window
  const results = currentWindow.filter(
    prop => !iframe.contentWindow.hasOwnProperty(prop)
  );

  // remove iframe
  document.body.removeChild(iframe);

  // log key-value entries that are different
  results.forEach(key => console.log(`${key}: ${window[key]}`));
})();
```

_(Version modifi�e de cet [extrait](https://stackoverflow.com/a/17246535/3099132))_

### Cha�ne d'attaque

Suite � l'identification de l'un des vecteurs d'attaque ci-dessus, une cha�ne d'attaque peut �tre form�e avec diff�rents types d'attaques c�t� client, telles que les attaques [XSS bas�es sur DOM](01-Testing_for_DOM-based_Cross_Site_Scripting.md).

## Correction

Les applications doivent stocker les donn�es sensibles c�t� serveur, et non c�t� client, de mani�re s�curis�e, conform�ment aux meilleures pratiques.

## R�f�rences

- [Stockage local](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [Stockage de session](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
- [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [API Web Crypto : Stockage de cl�s](https://www.w3.org/TR/WebCryptoAPI/#concepts-key-storage)
- [WebSQL](https://www.w3.org/TR/webdatabase/)
- [Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)

Pour plus de ressources OWASP sur l'API de stockage Web HTML5, consultez la [Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html#html5-web-storage-api).
