---
title: Cross-origin resource sharing (CORS)
slug: Web/HTTP/CORS
tags:
  - AJAX
  - CORS
  - HTTP
  - Same-origin policy
  - XMLHttpRequest
  - cross-site
translation_of: Web/HTTP/CORS
---
<div>{{HTTPSidebar}}</div>

<p>Le «  <em>Cross-origin resource sharing</em> » (CORS) ou « partage des ressources entre origines multiples » (en français, moins usité) est un mécanisme qui consiste à ajouter des en-têtes HTTP afin de permettre à un agent utilisateur d'accéder à des ressources d'un serveur situé sur une autre origine que le site courant. Un agent utilisateur réalise une requête HTTP <strong>multi-origine (<em>cross-origin</em>)</strong> lorsqu'il demande une ressource provenant d'un domaine, d'un protocole ou d'un port différent de ceux utilisés pour la page courante.</p>

<p>Prenons un exemple de requête multi-origine : une page HTML est servie depuis <code>http://domaine-a.com</code> contient un élément <code><a href="/fr/docs/Web/HTML/Element/Img#attr-src">&lt;img&gt; src</a></code> ciblant <code>http://domaine-b.com/image.jpg</code>. Aujourd'hui, de nombreuses pages web chargent leurs ressources (feuilles CSS, images, scripts) à partir de domaines séparés (par exemple des CDN (<em>Content Delivery Network</em> en anglais ou « Réseau de diffusion de contenu »).</p>

<p>Pour des raisons de sécurité, les requêtes HTTP multi-origine émises depuis les scripts sont restreintes. Ainsi, {{domxref("XMLHttpRequest")}} et l'<a href="/en-US/docs/Web/API/Fetch_API">API Fetch</a> respectent la règle <a href="/en-US/docs/Web/Security/Same-origin_policy">d'origine unique</a>. Cela signifie qu'une application web qui utilise ces API peut uniquement émettre des requêtes vers la même origine que celle à partir de laquelle l'application a été chargée, sauf si des en-têtes CORS sont utilisés.</p>

<p><img alt="" src="cors_principle.png"></p>

<p>Le CORS permet de prendre en charge des requêtes multi-origines sécurisées et des transferts de données entre des navigateurs et des serveurs web. Les navigateurs récents utilisent le CORS dans une API contenante comme {{domxref("XMLHttpRequest")}} ou <code><a href="/fr/docs/Web/API/Fetch_API">Fetch</a></code> pour aider à réduire les risques de requêtes HTTP multi-origines.</p>

<h2 id="À_qui_est_destiné_cet_article">À qui est destiné cet article ?</h2>

<p>Cet article est destiné à toutes et à tous.</p>

<p>Il pourra notamment servir aux administrateurs web, aux développeurs côté serveur ainsi qu'aux développeurs côté client. Les navigateurs récents permettent de gérer les règles de partage multi-origine côté client grâce à certaines règles et en-têtes mais cela implique également que des serveurs puissent gérer ces requêtes et réponses. Aussi, pour compléter le spectre concerné, nous vous invitons à lire d'autres articles complétant le point de vue « serveur » (par exemple <a href="/fr/docs/Web/HTTP/Server-Side_Access_Control">cet article utilisant des fragments de code PHP</a>).</p>

<h2 id="Quelles_requêtes_utilisent_le_CORS">Quelles requêtes utilisent le CORS ?</h2>

<p>Le <a class="external" href="https://fetch.spec.whatwg.org/#http-cors-protocol">standard CORS</a> est utilisé afin de permettre les requêtes multi-origines pour :</p>

<ul>
 <li>L'utilisation des API {{domxref("XMLHttpRequest")}} ou <a href="/fr/docs/Web/API/Fetch_API">Fetch</a></li>
 <li>Les polices web (pour récupérer des polices provenant d'autres origines lorsqu'on utilise {{cssxref("@font-face")}} en CSS), <a class="external" href="https://www.w3.org/TR/css-fonts-3/#font-fetching-requirements">afin que les serveurs puissent déployer des polices TrueType uniquement chargées en <em>cross-site</em> et utilisées par les sites web qui l'autorisent</a></li>
 <li><a href="/fr/docs/Web/API/WebGL_API/Tutorial/Utiliser_les_textures_avec_WebGL">Les textures WebGL</a></li>
 <li>Les <em>frames</em> (images ou vidéo) dessinées sur un canevas avec <code><a href="/fr/docs/Web/API/CanvasRenderingContext2D/drawImage">drawImage</a></code></li>
 <li>Les feuilles de style (pour les accès <a href="/fr/docs/Web/CSS/CSSOM_View">CSSOM</a>)</li>
 <li>Les scripts (pour les exceptions non silencieuses (<em>unmuted exceptions</em>)).</li>
</ul>

<p>Cet article propose un aperçu général de <em>Cross-Origin Resource Sharing</em> ainsi qu'un aperçu des en-têtes HTTP nécessaires.</p>

<h2 id="Aperçu_fonctionnel">Aperçu fonctionnel</h2>

<p>Le standard CORS fonctionne grâce à l'ajout de nouveaux <a href="/fr/docs/Web/HTTP/Headers">en-têtes HTTP</a> qui permettent aux serveurs de décrire un ensemble d'origines autorisées pour lire l'information depuis un navigateur web. De plus, pour les méthodes de requêtes HTTP qui entraînent des effets de bord sur les données côté serveur (notamment pour les méthodes en dehors de {{HTTPMethod("GET")}} ou pour les méthodes {{HTTPMethod("POST")}} utilisées avec certains <a href="/fr/docs/Web/HTTP/Basics_of_HTTP/MIME_types">types MIME</a>), la spécification indique que les navigateurs doivent effectuer une requête préliminaire (« <em>preflight request</em> ») et demander au serveur les méthodes prises en charges via une requête utilisant la méthode {{HTTPMethod("OPTIONS")}} puis, après approbation du serveur, envoyer la vraie requête. Les serveurs peuvent également indiquer aux clients s'il est nécessaire de fournir des informations d'authentification (que ce soit des <a href="/fr/docs/Web/HTTP/Cookies">cookies</a> ou des données d'authentification HTTP) avec les requêtes.</p>

<p>Les sections qui suivent évoquent les différents scénarios relatifs au CORS ainsi qu'un aperçu des en-têtes HTTP utilisés.</p>

<h2 id="Exemples_de_scénarios_pour_le_contrôle_daccès">Exemples de scénarios pour le contrôle d'accès</h2>

<p>Voyons ici trois scénarios qui illustrent le fonctionnement du CORS. Tous ces exemples utilisent l'objet {{domxref("XMLHttpRequest")}} qui peut être utilisé afin de faire des requêtes entre différents sites (dans les navigateurs qui prennent en charge cette fonctionnalité).</p>

<p>Les fragments de code JavaScript (ainsi que les instances serveurs qui gèrent ces requêtes) se trouvent sur <a class="external" href="http://arunranga.com/examples/access-control/">http://arunranga.com/examples/access-control/</a> et fonctionnent pour les navigateurs qui prennent en charge {{domxref("XMLHttpRequest")}} dans un contexte multi-site.</p>

<p>Un aperçu « côté serveur » des fonctionnalités CORS se trouve dans l'article <a href="/fr/docs/Web/HTTP/Server-Side_Access_Control">Contrôle d'accès côté serveur</a>.</p>

<h3 id="Requêtes_simples">Requêtes simples</h3>

<p>Certaines requêtes ne nécessitent pas de <a href="#preflight">requête CORS préliminaire</a>. Dans le reste de cet article, ce sont ce que nous appellerons des requêtes « simples » (bien que la spécification {{SpecName('Fetch')}} (qui définit le CORS) n'utilise pas ce terme). Une requête simple est une requête qui respecte les conditions suivantes :</p>

<ul>
 <li>Les seules méthodes autorisées sont :
  <ul>
   <li>{{HTTPMethod("GET")}}</li>
   <li>{{HTTPMethod("HEAD")}}</li>
   <li>{{HTTPMethod("POST")}}</li>
  </ul>
 </li>
 <li>En dehors des en-têtes paramétrés automatiquement par l'agent utilisateur (tels que {{HTTPHeader("Connection")}}, {{HTTPHeader("User-Agent")}} ou <a href="https://fetch.spec.whatwg.org/#forbidden-header-name">tout autre en-tête dont le nom fait partie de la spécification Fetch comme « nom d'en-tête interdit »</a>), les seuls en-têtes qui peuvent être paramétrés manuellement sont, selon <a href="https://fetch.spec.whatwg.org/#cors-safelisted-request-header">la spécification</a> :
  <ul>
   <li>{{HTTPHeader("Accept")}}</li>
   <li>{{HTTPHeader("Accept-Language")}}</li>
   <li>{{HTTPHeader("Content-Language")}}</li>
   <li>{{HTTPHeader("Content-Type")}} (cf. les contraintes supplémentaires ci-après)</li>
  </ul>
 </li>
 <li>Les seules valeurs autorisées pour l'en-tête {{HTTPHeader("Content-Type")}} sont :
  <ul>
   <li><code>application/x-www-form-urlencoded</code></li>
   <li><code>multipart/form-data</code></li>
   <li><code>text/plain</code></li>
  </ul>
 </li>
 <li>Aucun gestionnaire d'évènement n'est enregistré sur aucun des objets {{domxref("XMLHttpRequestUpload")}} utilisés pour la requête, on y accède via la propriété {{domxref("XMLHttpRequest.upload")}}.</li>
 <li>Aucun objet {{domxref("ReadableStream")}} n'est utilisé dans la requête.</li>
</ul>

<div class="note">
  <p><strong>Note :</strong> Cela correspond aux classes de requêtes généralement produites par du contenu web. Aucune donnée de réponse n'est envoyée au client qui a lancé la requête sauf si le serveur envoie un en-tête approprié. Aussi, les sites qui empêchent les requêtes étrangères falsifiées ne craignent rien de nouveau.</p>
</div>

<div class="note">
  <p><strong>Note :</strong> WebKit Nightly et Safari Technology Preview ajoutent des restrictions supplémentaires pour les valeurs autorisées des en-têtes {{HTTPHeader("Accept")}}, {{HTTPHeader("Accept-Language")}} et {{HTTPHeader("Content-Language")}}. Si l'un de ces en-têtes a une valeur non-standard, WebKit/Safari considère que la requête ne correspond pas à une requête simple. Les valeurs considérées comme non-standard par WebKit/Safari ne sont pas documentées en dehors de ces bugs WebKit : <em><a href="https://bugs.webkit.org/show_bug.cgi?id=165178" rel="nofollow noreferrer">Require preflight for non-standard CORS-safelisted request headers Accept, Accept-Language, and Content-Language</a></em>, <em><a href="https://bugs.webkit.org/show_bug.cgi?id=165566" rel="nofollow noreferrer">Allow commas in Accept, Accept-Language, and Content-Language request headers for simple CORS</a></em> et <em><a href="https://bugs.webkit.org/show_bug.cgi?id=166363" rel="nofollow noreferrer">Switch to a blacklist model for restricted Accept headers in simple CORS requests</a></em>. Aucun autre navigateur n'implémente ces restrictions supplémentaires, car elles ne font pas partie de la spécification.</p>
</div>

<p>Si, par exemple, on a un contenu web situé sous le domaine <code>http://toto.example</code> qui souhaite invoquer du contenu situé sous le domaine <code>http://truc.autre</code>, on pourrait utiliser du code JavaScript semblable à ce qui suit sur <code>toto.example</code> :</p>

<pre class="brush: js">var invocation = new XMLHttpRequest();
var url = 'http://truc.autre/resources/public-data/';

function callOtherDomain() {
  if(invocation) {
    invocation.open('GET', url, true);
    invocation.onreadystatechange = handler;
    invocation.send();
  }
}
</pre>

<p>Cela entraînera un échange simple entre le client et le serveur laissant aux en-têtes CORS le soin de gérer les privilèges d'accès :</p>

<p><img alt="" src="simple-req-updated.png"></p>

<p>Voyons dans le détail ce que le navigateur envoie au serveur et quelle sera sa réponse :</p>

<pre>GET /resources/public-data/ HTTP/1.1
Host: truc.autre
User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.5; en-US; rv:1.9.1b3pre) Gecko/20081130 Minefield/3.1b3pre
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Connection: keep-alive
Referer: http://toto.example/exemples/access-control/simpleXSInvocation.html
Origin: http://toto.example


HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 00:23:53 GMT
Server: Apache/2.0.61
Access-Control-Allow-Origin: *
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: application/xml

[XML Data]
</pre>

<p>Les lignes 1 à 10 correspondent aux en-têtes envoyés. L'en-tête qui nous intéresse particulièrement ici est {{HTTPHeader("Origin")}}, situé à la ligne 10 : on y voit que l'invocation provient du domaine <code>http://toto.example</code>.</p>

<p>Les lignes 13 à 22 détaillent la réponse HTTP du serveur situé sous le domaine <code>http://truc.autre</code>. Dans la réponse, le serveur renvoie un en-tête {{HTTPHeader("Access-Control-Allow-Origin")}} (visible à la ligne 16). On voit ici les en-têtes {{HTTPHeader("Origin")}} et {{HTTPHeader("Access-Control-Allow-Origin")}} pour un contrôle d'accès dans sa forme la plus simple. Ici, le serveur répond avec <code>Access-Control-Allow-Origin: *</code> ce qui signifie que la ressource peut être demandée par n'importe quel domaine. Si les propriétés de la ressource située sous <code>http://truc.autre</code> souhaitaient restreindre l'accès à la ressource à l'origine <code>http://toto.example</code>, ils auraient renvoyé :</p>

<p><code>Access-Control-Allow-Origin: http://toto.example</code></p>

<p>On notera que, dans ce cas, aucun autre domaine que <code>http://toto.example</code> (tel qu'identifié par l'en-tête <code>Origin</code>) ne pourra accéder à la ressource. L'en-tête <code>Access-Control-Allow-Origin</code> devrait contenir la valeur qui a été envoyée dans l'en-tête  <code>Origin</code> de la requête.</p>

<h3 id="Requêtes_nécessitant_une_requête_préliminaire">Requêtes nécessitant une requête préliminaire</h3>

<p>À la différence des <a href="#simples">requêtes simples</a>, les requêtes préliminaires envoient d'abord une requête HTTP avec la méthode {{HTTPMethod("OPTIONS")}} vers la ressource de l'autre domaine afin de déterminer quelle requête peut être envoyée de façon sécurisée. Les requêtes entre différents sites peuvent notamment utiliser ce mécanisme de vérification préliminaire lorsque des données utilisateurs sont impliquées.</p>

<p>Une requête devra être précédée d'une requête préliminaire si <strong>une</strong> des conditions suivantes est respectée :</p>

<ul>
 <li>La requête utilise une des méthodes suivantes :
  <ul>
   <li>{{HTTPMethod("PUT")}}</li>
   <li>{{HTTPMethod("DELETE")}}</li>
   <li>{{HTTPMethod("CONNECT")}}</li>
   <li>{{HTTPMethod("OPTIONS")}}</li>
   <li>{{HTTPMethod("TRACE")}}</li>
   <li>{{HTTPMethod("PATCH")}}</li>
  </ul>
 </li>
 <li><strong>Ou si</strong>, en dehors des en-têtes automatiquement paramétrés par l'agent utilisateur (comme {{HTTPHeader("Connection")}}, {{HTTPHeader("User-Agent")}} ou <a href="https://fetch.spec.whatwg.org/#forbidden-header-name">tout autre en-tête dont le nom est réservé dans la spécification</a>), la requête inclut <a href="https://fetch.spec.whatwg.org/#cors-safelisted-request-header">tout autre en-tête que ceux définis sur la liste blanche</a> :
  <ul>
   <li>{{HTTPHeader("Accept")}}</li>
   <li>{{HTTPHeader("Accept-Language")}}</li>
   <li>{{HTTPHeader("Content-Language")}}</li>
   <li>{{HTTPHeader("Content-Type")}} (cf. les contraintes supplémentaires ci-après)</li>
   <li>{{HTTPHeader("Last-Event-ID")}}</li>
   <li><code><a href="http://httpwg.org/http-extensions/client-hints.html#dpr">DPR</a></code></li>
   <li><code><a href="http://httpwg.org/http-extensions/client-hints.html#save-data">Save-Data</a></code></li>
   <li><code><a href="http://httpwg.org/http-extensions/client-hints.html#viewport-width">Viewport-Width</a></code></li>
   <li><code><a href="http://httpwg.org/http-extensions/client-hints.html#width">Width</a></code></li>
  </ul>
 </li>
 <li><strong>Ou si</strong> l'en-tête {{HTTPHeader("Content-Type")}} possède une valeur autre que :
  <ul>
   <li><code>application/x-www-form-urlencoded</code></li>
   <li><code>multipart/form-data</code></li>
   <li><code>text/plain</code></li>
  </ul>
 </li>
 <li><strong>Ou si</strong> un ou plusieurs gestionnaires d'évènements sont enregistrés sur l'objet {{domxref("XMLHttpRequestUpload")}} utilisé dans la requête.</li>
 <li><strong>Ou si</strong> un objet {{domxref("ReadableStream")}} est utilisé dans la requête.</li>
</ul>

<div class="note">
  <p><strong>Note :</strong> WebKit Nightly et Safari Technology Preview ajoutent des restrictions supplémentaires pour les valeurs autorisées des en-têtes {{HTTPHeader("Accept")}}, {{HTTPHeader("Accept-Language")}} et {{HTTPHeader("Content-Language")}}. Si l'un de ces en-têtes a une valeur non-standard, WebKit/Safari considère que la requête ne correspond pas à une requête simple. Les valeurs considérées comme non-standard par WebKit/Safari ne sont pas documentées en dehors de ces bugs WebKit : <em><a href="https://bugs.webkit.org/show_bug.cgi?id=165178" rel="nofollow noreferrer">Require preflight for non-standard CORS-safelisted request headers Accept, Accept-Language, and Content-Language</a></em>, <em><a href="https://bugs.webkit.org/show_bug.cgi?id=165566" rel="nofollow noreferrer">Allow commas in Accept, Accept-Language, and Content-Language request headers for simple CORS</a></em> et <em><a href="https://bugs.webkit.org/show_bug.cgi?id=166363" rel="nofollow noreferrer">Switch to a blacklist model for restricted Accept headers in simple CORS requests</a></em>. Aucun autre navigateur n'implémente ces restrictions supplémentaires, car elles ne font pas partie de la spécification.</p>
</div>

<p>Voici un exemple d'une requête qui devra être précédée d'une requête préliminaire :</p>

<pre class="brush: js">var invocation = new XMLHttpRequest();
var url = 'http://truc.autre/resources/post-here/';
var body = '&lt;?xml version="1.0"?&gt;&lt;personne&gt;&lt;nom&gt;Toto&lt;/nom&gt;&lt;/personne&gt;';

function callOtherDomain(){
  if(invocation)
    {
      invocation.open('POST', url, true);
      invocation.setRequestHeader('X-PINGOTHER', 'pingpong');
      invocation.setRequestHeader('Content-Type', 'application/xml');
      invocation.onreadystatechange = handler;
      invocation.send(body);
    }
}

......
</pre>

<p>Dans le fragment de code ci-avant, à la ligne 3, on crée un corps XML envoyé avec la requête <code>POST</code> ligne 8. Sur la ligne 9, on voit également un en-tête de requête HTTP non standard : <code>X-PINGOTHER: pingpong</code>. De tels en-têtes ne sont pas décrits par le protocole HTTP/1.1 mais peuvent être utilisés par les applications web. La requête utilisant un en-tête <code>Content-Type</code> qui vaut <code>application/xml</code> et un en-tête spécifique, il est nécessaire d'envoyer au préalable une requête préliminaire.</p>

<p><img alt="" src="preflight_correct.png"></p>

<div class="note">
<p><strong>Note :</strong> Comme décrit après, la vraie requête POST n'inclut pas les en-têtes <code>Access-Control-Request-*</code> qui sont uniquement nécessaires pour la requête OPTIONS.</p>
</div>

<p>Voyons ce qui se passe entre le client et le serveur. Le premier échange est la requête/réponse préliminaire :</p>

<pre>OPTIONS /resources/post-here/ HTTP/1.1
Host: truc.autre
User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.5; en-US; rv:1.9.1b3pre) Gecko/20081130 Minefield/3.1b3pre
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Connection: keep-alive
Origin: http://toto.example
Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-PINGOTHER, Content-Type


HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://toto.example
Access-Control-Allow-Methods: POST, GET
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
Access-Control-Max-Age: 86400
Vary: Accept-Encoding, Origin
Content-Encoding: gzip
Content-Length: 0
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain
</pre>

<p>Une fois que la requête préliminaire est effectuée, la requête principale est envoyée :</p>

<pre>POST /resources/post-here/ HTTP/1.1
Host: truc.autre
User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.5; en-US; rv:1.9.1b3pre) Gecko/20081130 Minefield/3.1b3pre
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Connection: keep-alive
X-PINGOTHER: pingpong
Content-Type: text/xml; charset=UTF-8
Referer: http://toto.example/exemples/preflightInvocation.html
Content-Length: 55
Origin: http://toto.example
Pragma: no-cache
Cache-Control: no-cache

&lt;?xml version="1.0"?&gt;&lt;personne&gt;&lt;nom&gt;Toto&lt;/nom&gt;&lt;/personne&gt;


HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:40 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://toto.example
Vary: Accept-Encoding, Origin
Content-Encoding: gzip
Content-Length: 235
Keep-Alive: timeout=2, max=99
Connection: Keep-Alive
Content-Type: text/plain

[Une charge utile GZIPée]
</pre>

<p>Entre les lignes 1 à 12 qui précèdent, on voit la requête préliminaire avec la méthode {{HTTPMethod("OPTIONS")}}. Le navigateur détermine qu'il est nécessaire d'envoyer cela à cause des paramètres de la requête fournie par le code JavaScript. De cette façon le serveur peut répondre si la requête principale est acceptable et avec quels paramètres. OPTIONS est une méthode HTTP/1.1 qui est utilisée afin de déterminer de plus amples informations à propos du serveur. La méthode OPTIONS est une méthode « sûre » (<em>safe</em>) et ne change aucune ressource. On notera, qu'avec la requête OPTIONS, deux autres en-têtes sont envoyés (cf. lignes 10 et 11) :</p>

<pre>Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-PINGOTHER, Content-Type
</pre>

<p>L'en-tête {{HTTPHeader("Access-Control-Request-Method")}} indique au serveur, pendant la requête préliminaire, que la requête principale sera envoyée avec la méthode <code>POST</code>. L'en-tête {{HTTPHeader("Access-Control-Request-Headers")}} indique au serveur que la requête principale sera envoyée avec un en-tête <code>X-PINGOTHER</code> et un en-tête <code>Content-Type</code> spécifique. Le serveur peut alors déterminer s'il souhaite accepter une telle requête.</p>

<p>Dans les lignes 14 à 26 qui suivent, on voit la réponse renvoyée par le serveur qui indique que la méthode de la requête (<code>POST</code>) ainsi que ses en-têtes (<code>X-PINGOTHER</code>) sont acceptables. Voici ce qu'on peut notamment lire entre les lignes 17 et 20 :</p>

<pre>Access-Control-Allow-Origin: http://toto.example
Access-Control-Allow-Methods: POST, GET
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
Access-Control-Max-Age: 86400</pre>

<p>Le serveur répond avec un en-tête <code>Access-Control-Allow-Methods</code> et indique que les méthodes <code>POST</code> et <code>GET</code> sont acceptables pour manipuler la ressource visée. On notera que cet en-tête est semblable à l'en-tête de réponse {{HTTPHeader("Allow")}}, toutefois, <code>Access-Control-Allow-Methods</code> est uniquement utilisé dans le cadre du contrôle d'accès.</p>

<p>Le serveur envoie également l'en-tête <code>Access-Control-Allow-Headers</code> avec une valeur "<code>X-PINGOTHER, Content-Type</code>" qui confirme que les en-têtes souhaités sont autorisés pour la requête principale. Comme <code>Access-Control-Allow-Methods</code>, <code>Access-Control-Allow-Headers</code> est une liste d'en-têtes acceptables séparés par des virgules.</p>

<p>Enfin, l'en-tête {{HTTPHeader("Access-Control-Max-Age")}} indique avec une valeur exprimée en secondes, la durée pendant laquelle cette réponse préliminaire peut être mise en cache avant la prochaine requête préliminaire. Ici, la réponse est 86400 secondes, ce qui correspond à 24 heures. On notera ici que chaque navigateur possède<a href="/fr/docs/Web/HTTP/Headers/Access-Control-Max-Age"> un maximum interne</a> qui a la priorité lorsque <code>Access-Control-Max-Age</code> lui est supérieur.</p>

<h4 id="Requêtes_préliminaires_et_redirection">Requêtes préliminaires et redirection</h4>

<p>À l'heure actuelle, la plupart des navigateurs ne prennent pas en charge les redirections pour les requêtes préliminaires. Si une redirection se produit pour une requête préliminaire, la plupart des navigateurs émettront un message d'erreur semblables à ceux-ci.</p>

<blockquote>
<p>La requête a été redirigée vers 'https://example.com/toto', ce qui n'est pas autorisé pour les requêtes multi-origines qui doivent être précédées d'une requête préliminaire. (<em>The request was redirected to 'https://example.com/toto', which is disallowed for cross-origin requests that require preflight.</em>)</p>
</blockquote>

<blockquote>
<p>Il est nécessaire d'effectuer une requête préliminaire pour cette requête, or, ceci n'est pas autorisé pour suivre les redirections multi-origines. (<em>Request requires preflight, which is disallowed to follow cross-origin redirect.</em>)</p>
</blockquote>

<p>Le protocole CORS demandait initialement ce comportement. Toutefois, <a href="https://github.com/whatwg/fetch/commit/0d9a4db8bc02251cc9e391543bb3c1322fb882f2">il a été modifié et ces erreurs ne sont plus nécessaires</a>. Toutefois, la plupart des navigateurs n'ont pas encore implémenté cette modification et conservent alors le comportement conçu initialement.</p>

<p>En attendant que les navigateurs comblent ce manque, il est possible de contourner cette limitation en utilisant l'une de ces deux méthodes :</p>

<ul>
 <li>Modifier le comportement côté serveur afin d'éviter la requête préliminaire ou la redirection (dans le cas où vous contrôler le serveur sur lequel la requête est effectuée)</li>
 <li>Modifier la requête afin que ce soit une <a href="#simples">requête simple</a> qui ne nécessite pas de requête préliminaire.</li>
</ul>

<p>S'il n'est pas possible d'appliquer ces changements, on peut également :</p>

<ol>
 <li>Effectuer <a href="#simples">une requête simple</a> (avec <code><a href="/fr/docs/Web/API/Response/url">Response.url</a></code> si on utilise l'API Fetch ou  <code><a href="/fr/docs/Web/API/XMLHttpRequest/responseURL">XHR.responseURL</a></code> si on utilise XHR) afin de déterminer l'URL à laquelle aboutirait la requête avec requête préliminaire.</li>
 <li>Effectuer la requête initialement souhaitée avec l'URL <em>réelle</em> obtenue à la première étape.</li>
</ol>

<p>Toutefois, si la requête déclenche une requête préliminaire suite à l'absence de l'en-tête {{HTTPHeader("Authorization")}}, on ne pourra pas utiliser cette méthode de contournement et il sera nécessaire d'avoir accès au serveur pour contourner le problème.</p>

<h3 id="Requêtes_avec_informations_dauthentification">Requêtes avec informations d'authentification</h3>

<p>Une des fonctionnalités intéressante mise en avant par le CORS (via {{domxref("XMLHttpRequest")}} ou <a href="/fr/docs/Web/API/Fetch_API">Fetch</a>) est la possibilité d'effectuer des requêtes authentifiées reconnaissant les <a href="/fr/docs/Web/HTTP/Cookies">cookies HTTP</a> et les informations d'authentification HTTP. Par défaut, lorsqu'on réalise des appels {{domxref("XMLHttpRequest")}} ou <a href="/fr/docs/Web/API/Fetch_API">Fetch</a> entre différents sites, les navigateurs n'enverront pas les informations d'authentification. Pour cela, il est nécessaire d'utiliser une option spécifique avec le constructeur {{domxref("XMLHttpRequest")}} ou {{domxref("Request")}} lorsqu'on l'appelle.</p>

<p>Dans cet exemple, le contenu chargé depuis <code>http://toto.example</code> effectue une requête GET simple vers une ressource située sous <code>http://truc.autre</code> qui définit des <em>cookies</em>. Voici un exemple de code JavaScript qui pourrait se trouver sur <code>toto.example</code> :</p>

<pre class="brush: js">var invocation = new XMLHttpRequest();
var url = 'http://truc.autre/resources/credentialed-content/';

function callOtherDomain(){
  if(invocation) {
    invocation.open('GET', url, true);
    invocation.withCredentials = true;
    invocation.onreadystatechange = handler;
    invocation.send();
  }
}</pre>

<p>À la ligne 7, on voit que l'option <code>withCredentials</code>, du constructeur {{domxref("XMLHttpRequest")}}, doit être activée pour que l'appel utilise les <em>cookies</em>. Par défaut, l'appel sera réalisé sans les <em>cookies</em>. Cette requête étant une simple requête <code>GET</code>, il n'est pas nécessaire d'avoir une requête préliminaire. Cependant, le navigateur rejettera tout réponse qui ne possède pas l'en-tête {{HTTPHeader("Access-Control-Allow-Credentials")}}<code>: true</code> et la réponse correspondante ne sera pas disponible pour le contenu web qui l'a demandée.</p>

<p><img alt="" src="cred-req-updated.png"></p>

<p>Voici un exemple d'échange entre le client et le serveur :</p>

<pre>GET /resources/access-control-with-credentials/ HTTP/1.1
Host: truc.autre
User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.5; en-US; rv:1.9.1b3pre) Gecko/20081130 Minefield/3.1b3pre
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Connection: keep-alive
Referer: http://toto.example/exemples/credential.html
Origin: http://toto.example
Cookie: pageAccess=2


HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:34:52 GMT
Server: Apache/2.0.61 (Unix) PHP/4.4.7 mod_ssl/2.0.61 OpenSSL/0.9.7e mod_fastcgi/2.4.2 DAV/2 SVN/1.4.2
X-Powered-By: PHP/5.2.6
Access-Control-Allow-Origin: http://toto.example
Access-Control-Allow-Credentials: true
Cache-Control: no-cache
Pragma: no-cache
Set-Cookie: pageAccess=3; expires=Wed, 31-Dec-2008 01:34:53 GMT
Vary: Accept-Encoding, Origin
Content-Encoding: gzip
Content-Length: 106
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain


[text/plain payload]
</pre>

<p>Bien que la ligne 11 contienne le <em>cookie</em> pour le contenu sous <code>http://truc.autre</code>, si <code>truc.autre</code> n'avait pas répondu avec {{HTTPHeader("Access-Control-Allow-Credentials")}}<code>: true</code> (cf. ligne 19), la réponse aurait été ignorée et n'aurait pas pu être consommée par le contenu web.</p>

<h4 id="Requêtes_authentifiées_et_jokers_wildcards">Requêtes authentifiées et jokers (<em>wildcards</em>)</h4>

<p>Lorsqu'il répond à une requête authentifiée, le serveur <strong>doit</strong> indiquer une origine via la valeur de l'en-tête <code>Access-Control-Allow-Origin</code>, il ne doit pas utiliser le joker "<code>*</code>".</p>

<p>Avec la requête précédente, on voit la présence d'un en-tête <code>Cookie</code> mais la requête échouerait si la valeur de l'en-tête de réponse <code>Access-Control-Allow-Origin</code> était "<code>*</code>". Ici, ce n'est pas le cas : <code>Access-Control-Allow-Origin</code> vaut "<code>http://toto.example</code>" et le contenu récupéré par la requête est alors envoyé au contenu web.</p>

<p>Dans cet exemple, on notera également que l'en-tête de réponse <code>Set-Cookie</code> définit un autre <em>cookie</em>. En cas d'échec, une exception (dépendant de l'API utilisée) sera levée.</p>

<h4 id="Cookies_tiers"><em>Cookies</em> tiers</h4>

<p>On notera que les <em>cookies</em> provenant de réponses CORS sont également sujets aux règles qui s'appliquent aux <em>cookies</em> tiers. Dans l'exemple précédent, la page est chargée depuis <code>toto.example</code> et, à la ligne 22, le <em>cookie</em> est envoyé par <code>truc.autre</code>. Aussi, ce <em>cookie</em> n'aurait pas été enregistré si l'utilisateur avait paramétré son navigateur pour rejeter les <em>cookies</em> tiers.</p>

<h2 id="En-têtes_de_réponse_HTTP">En-têtes de réponse HTTP</h2>

<p>Dans cette section, on liste les en-têtes de réponse HTTP qui sont renvoyés par le serveur pour le contrôle d'accès, tels que définis par la spécification <em>Cross-Origin Resource Sharing</em>. La section précédente illustre, avec des exemples concrets, leur fonctionnement.</p>

<h3 id="Access-Control-Allow-Origin"><code>Access-Control-Allow-Origin</code></h3>

<p>Une ressource de réponse peut avoir un en-tête {{HTTPHeader("Access-Control-Allow-Origin")}} avec la syntaxe suivante :</p>

<pre>Access-Control-Allow-Origin: &lt;origin&gt; | *
</pre>

<p>Le paramètre <code>origin</code> définit un URI qui peut accéder à la ressource. Le navigateur doit respecter cette contrainte. Pour les requêtes qui n'impliquent pas d'informations d'authentification, le serveur pourra indiquer un joker ("<code>*</code>") qui permet à n'importe quelle origine d'accéder à la ressource.</p>

<p>Si on souhaite, par exemple, autoriser <code>http://mozilla.org</code> à accéder à la ressource, on pourra répondre avec :</p>

<pre>Access-Control-Allow-Origin: http://mozilla.org</pre>

<p>Si le serveur indique une origine spécifique plutôt que "<code>*</code>", il pourra également inclure la valeur <code>Origin</code> dans l'en-tête de réponse {{HTTPHeader("Vary")}} pour indiquer au client que la réponse du serveur variera selon la valeur de l'en-tête de requête {{HTTPHeader("Origin")}}.</p>

<h3 id="Access-Control-Expose-Headers"><code>Access-Control-Expose-Headers</code></h3>

<p>L'en-tête {{HTTPHeader("Access-Control-Expose-Headers")}} fournit une liste blanche des en-têtes auxquels les navigateurs peuvent accéder. Ainsi :</p>

<pre>Access-Control-Expose-Headers: X-Mon-En-tete-Specifique, X-Un-Autre-En-tete
</pre>

<p>Cela permettra que les en-têtes <code>X-Mon-En-tete-Specifique</code> et <code>X-Un-Autre-En-tete</code> soient utilisés par le navigateur.</p>

<h3 id="Access-Control-Max-Age"><code>Access-Control-Max-Age</code></h3>

<p>L'en-tête {{HTTPHeader("Access-Control-Max-Age")}} indique la durée pendant laquelle le résultat de la requête préliminaire peut être mis en cache (voir les exemples ci-avant pour des requêtes impliquant des requêtes préliminaires).</p>

<pre>Access-Control-Max-Age: &lt;delta-en-secondes&gt;
</pre>

<p>Le paramètre <code>delta-en-seconds</code> indique le nombre de secondes pendant lesquelles les résultats peuvent être mis en cache.</p>

<h3 id="Access-Control-Allow-Credentials"><code>Access-Control-Allow-Credentials</code></h3>

<p>L'en-tête {{HTTPHeader("Access-Control-Allow-Credentials")}} indique si la réponse à la requête doit être exposée lorsque l'option <code>credentials</code> vaut <code>true</code>. Lorsque cet en-tête est utilisé dans une réponse préliminaire, cela indique si la requête principale peut ou non être effectuée avec des informations d'authentification. On notera que les requêtes <code>GET</code> sont des requêtes simples et si une requête est effectuée, avec des informations d'authentification pour une ressource, et que cet en-tête n'est pas renvoyé, la réponse sera ignorée par le navigateur et sa charge ne pourra pas être consommée par le contenu web.</p>

<pre>Access-Control-Allow-Credentials: true
</pre>

<p><a href="#credentials">Voir les scénarios ci-avant pour des exemples</a>.</p>

<h3 id="Access-Control-Allow-Methods"><code>Access-Control-Allow-Methods</code></h3>

<p>L'en-tête {{HTTPHeader("Access-Control-Allow-Methods")}} indique la ou les méthodes qui sont autorisées pour accéder à la ressoure. Cet en-tête est utilisé dans la réponse à la requête préliminaire (voir ci-avant <a href="#preflight">les conditions dans lesquelles une requête préliminaire est nécessaire</a>).</p>

<pre>Access-Control-Allow-Methods: &lt;methode&gt;[, &lt;methode&gt;]*
</pre>

<p><a href="#preflight">Voir un exemple ci-avant pour l'utilisation de cet en-tête</a>.</p>

<h3 id="Access-Control-Allow-Headers"><code>Access-Control-Allow-Headers</code></h3>

<p>L'en-tête {{HTTPHeader("Access-Control-Allow-Headers")}} est utilisé dans une réponse à une requête préliminaire afin d'indiquer les en-têtes HTTP qui peuvent être utilisés lorsque la requête principale est envoyée.</p>

<pre>Access-Control-Allow-Headers: &lt;nom-champ&gt;[, &lt;nom-champ&gt;]*
</pre>

<h2 id="En-têtes_de_requête_HTTP">En-têtes de requête HTTP</h2>

<p>Dans cette section, nous allons décrire les en-têtes que les clients peuvent utiliser lors de l'envoi de requêtes HTTP afin d'exploiter les fonctionnalités du CORS. Ces en-têtes sont souvent automatiquement renseignés lors d'appels aux serveurs. Les développeurs qui utilisent {{domxref("XMLHttpRequest")}} pour les requêtes multi-origines n'ont pas besoin de paramétrer ces en-têtes dans le code JavaScript.</p>

<h3 id="Origin"><code>Origin</code></h3>

<p>L'en-tête {{HTTPHeader("Origin")}} indique l'origine de la requête (principale ou préliminaire) pour l'accès multi-origine.</p>

<pre>Origin: &lt;origine&gt;
</pre>

<p>L'origine est un URI qui indique le serveur à partir duquel la requête a été initiée. Elle n'inclut aucune information relative au chemin mais contient uniquement le nom du serveur.</p>

<div class="note">
  <p><strong>Note :</strong> <code>origine</code> peut être une chaîne vide (ce qui s'avère notamment utile lorsque la source est une URL de donnée).</p>
</div>

<p>Pour chaque requête avec contrôle d'accès, l'en-tête {{HTTPHeader("Origin")}} sera <strong>toujours</strong> envoyé.</p>

<h3 id="Access-Control-Request-Method"><code>Access-Control-Request-Method</code></h3>

<p>L'en-tête {{HTTPHeader("Access-Control-Request-Method")}} est utilisé lorsqu'on émet une requête préliminaire afin de savoir quelle méthode HTTP pourra être utilisée avec la requête principale.</p>

<pre>Access-Control-Request-Method: &lt;methode&gt;
</pre>

<p>Voir <a href="#preflight">ci-avant pour des exemples d'utilisation de cet en-tête</a>.</p>

<h3 id="Access-Control-Request-Headers"><code>Access-Control-Request-Headers</code></h3>

<p>L'en-tête {{HTTPHeader("Access-Control-Request-Headers")}} est utilisé lorsqu'on émet une requête préliminaire afin de communiquer au serveur les en-têtes HTTP qui seront utilisés avec la requête principale.</p>

<pre>Access-Control-Request-Headers: &lt;nom-champ&gt;[, &lt;nom-champ&gt;]*
</pre>

<p>Voir <a href="#preflight">ci-avant pour des exemples d'utilisation de cet en-tête</a>.</p>

<h2 id="Spécifications">Spécifications</h2>

<table class="standard-table">
 <tbody>
  <tr>
   <th scope="col">Spécification</th>
   <th scope="col">État</th>
   <th scope="col">Commentaires</th>
  </tr>
  <tr>
   <td>{{SpecName('Fetch', '#cors-protocol', 'CORS')}}</td>
   <td>{{Spec2('Fetch')}}</td>
   <td>Nouvelle définition, remplace la spécification <a href="https://www.w3.org/TR/cors/">W3C pour le CORS</a>.</td>
  </tr>
 </tbody>
</table>

<h2 id="Compatibilité_des_navigateurs">Compatibilité des navigateurs</h2>

<p>{{Compat("http.headers.Access-Control-Allow-Origin")}}</p>

<h3 id="Notes_de_compatibilité">Notes de compatibilité</h3>

<ul>
 <li>Internet Explorer 8 et 9 exposent les fonctionnalités relatives au CORS via l'objet <code>XDomainRequest</code>. L'implémentation complète est disponible à partir d'IE 10.</li>
 <li>Bien que Firefox 3.5 ait introduit la prise en charge des requêtes <code>XMLHttpRequest</code> entre différents sites et des polices web, certaines requêtes étaient limitées jusqu'à des versions ultérieures. Plus précisément, Firefox 7 permet les requêtes multi-origines pour les textures WebGL et Firefox 9 permet la récupération d'images dessinées sur un canevas via <code>drawImage</code>.</li>
</ul>

<h2 id="Voir_aussi">Voir aussi</h2>

<ul>
 <li><a class="external" href="https://arunranga.com/examples/access-control/">Exemples de codes utilisant <code>XMLHttpRequest</code> et le CORS (en anglais)</a></li>
 <li><a href="https://github.com/jackblackevo/cors-jsonp-sample">Exemples de code côté client et côté serveur utilisant le CORS (en anglais)</a></li>
 <li><a href="/fr/docs/Web/HTTP/Server-Side_Access_Control">Le CORS vu côté serveur (PHP, etc.)</a></li>
 <li>{{domxref("XMLHttpRequest")}}</li>
 <li><a href="/fr/docs/Web/API/Fetch_API">L'API Fetch</a></li>
 <li><a href="https://www.html5rocks.com/en/tutorials/cors/">Utiliser le CORS - HTML5 Rocks (en anglais)</a></li>
 <li><a href="https://stackoverflow.com/questions/43871637/no-access-control-allow-origin-header-is-present-on-the-requested-resource-whe/43881141#43881141">Une réponse Stack Overflow pour répondre aux problèmes fréquemment posés par le CORS (en anglais)</a> :
  <ul>
   <li>Comment éviter les requêtes préliminaires</li>
   <li>Comment utiliser un proxy CORS pour contourner <em>No Access-Control-Allow-Origin header</em></li>
   <li>Comment corriger <em>Access-Control-Allow-Origin header must not be the wildcard</em></li>
  </ul>
 </li>
</ul>