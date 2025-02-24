---
layout: article
title: "Ottimizzazione dei font web"
description: "La tipografia è fondamentale per un design, un branding, una leggibilità e accessibilità idonei. I font web consentono di ottenere tutto ciò, e non solo: il testo è selezionabile, ricercabile, ingrandibile e ampiamente DPI friendly, offrendo un rendering coerente e conciso, indipendentemente dalle dimensioni e dalla risoluzione del monitor. I font web sono fondamentali per ottenere design, UX e prestazioni ottimali."
introduction: "La tipografia è fondamentale per un design, un branding, una leggibilità e accessibilità idonei. I font web consentono di ottenere tutto ciò, e non solo: il testo è selezionabile, ricercabile, ingrandibile e ampiamente DPI friendly, offrendo un rendering coerente e conciso, indipendentemente dalle dimensioni e dalla risoluzione del monitor. I font web sono fondamentali per ottenere design, UX e prestazioni ottimali."
article:
  written_on: 2014-09-20
  updated_on: 2014-09-30
  order: 4
collection: optimizing-content-efficiency
authors:
  - ilyagrigorik
key-takeaways:
  anatomy:
    - "I font Unicode possono contenere migliaia di glifi"
    - "I formati dei font sono quattro: WOFF2, WOFF, EOT, TTF"
    - "Per alcuni formati font è necessario utilizzare una compressione GZIP"
  font-family:
    - "Utilizza il format() hint per specificare più formati font"
    - "Raggruppa i font unicode più ampi in subset per migliorare le prestazioni: utilizza subset unicode-range e consenti in alternativa un subsetting manuale per versioni precedenti dei browser"
    - "Riduci il numero di varianti dei font di stile per migliorare le prestazioni della pagina e il rendering del testo"
  font-crp:
    - "Le richieste di font vengono ritardate sino alla costruzione del render tree; ciò può comportare un ritardo nel rendering del testo"
    - "Il Font Loading API ci consente di caricare font e applicare strategie di rendering personalizzate al posto del font loading in lazy load predefinito"
    - "Il Font inlining ci consente di sovrascrivere il fontloading in lazy load predefinito in versioni precedenti dei browser"

notes:
  svg:
    - "Tecnicamente esiste anche l'<a href='http://caniuse.com/svg-fonts'>SVG font container</a>, ma non è mai stato supportato né in IE, né in Firefox e non è approvato in Chrome. Di conseguenza, ha un utilizzo limitato e lo ometteremo intenzionalmente nella presente guida."
  zopfli:
    - "Valuta l`utilizzo della <a href='http://en.wikipedia.org/wiki/Zopfli'>compressione Zopfli</a> per i formati EOT, TTF e WOFF. Zopfli è un compressore compatibile con zlib che offre una riduzione delle dimensioni file ~5% superiore a gzip."
  local-fonts: 
    - "A meno che non faccia riferimento a uno dei font di sistema predefiniti, nella realtà è raro che l`utente l`abbia installato localmente, soprattutto su dispositivi mobili, dove è di fatto impossibile `installare` font aggiuntivi. Di conseguenza, dovrai sempre fornire un elenco di percorsi font esterni."
  font-order:
    - "L`ordine di specifica delle varianti del font ha una data importanza. Il browser sceglierà il primo formato supportato. Di conseguenza, se desideri che i browser più recenti utilizzino WOFF2, dovrai posizionare il riferimento WOFF2 sopra WOFF, e così via."
  unicode-subsetting:
    - "I subset unicode-range sono particolarmente importanti per le lingue asiatiche, il cui numero di glifi è molto più elevato rispetto alle lingue occidentali e un tipico font `intero` è spesso misurato in megabyte, invece che in decine di kilobyte!"
  synthesis:
    - "Per una maggiore coerenza nei risultati visivi, non affidarti alla font-synthesis. Minimizza invece il numero di varianti del font utilizzate e specificane il percorso, in modo che il browser possa scaricarle quando sono utilizzate nella pagina. Detto ciò, in alcuni casi una variante synthesized <a href='https://www.igvita.com/2014/09/16/optimizing-webfont-selection-and-synthesis/'>può rappresentare un`opzione valida</a> - utilizzala con cautela."
  webfontloader:
    - "Font Loading API è ancora <a href='http://caniuse.com/#feat=font-loading'>in fase di sviluppo in alcuni browser</a>. Valuta l`utilizzo del <a href='https://github.com/bramstein/fontloader'>FontLoader polyfill</a> o della <a href='https://github.com/typekit/webfontloader'>webfontloader library</a> per ottenere funzionalità simili, sebbene con un`ulteriore dipendenza da JavaScript."
  font-inlining: 
    - "Utilizza l`inlining con criterio selettivo! Ricorda che@font-face utilizza il lazy load per evitare di scaricare varianti e subset di font non necessari. Inoltre, l`aumento delle dimensioni del tuo CSS con un inlining aggressivo avrà un impatto negativo sul <a href='/web/fundamentals/performance/critical-rendering-path/'>percorso di rendering critico</a> - il  browser deve scaricare l`intero CSS prima di costruire il CSSOM e il render tree e visualizzare i contenuti della pagina sullo schermo."
---

{% wrap content%}

{% include modules/toc.liquid %}

L`ottimizzazione dei font web rappresenta una fase critica nell`intera strategia prestazionale. Ogni font costituisce una risorsa aggiuntiva, e alcuni font possono bloccare il rendering del testo, ma il solo fatto che la pagina utilizzi i font web non significa che il rendering debba essere più lento. Al contrario, un font ottimizzato, assieme a una strategia ponderata in merito al loro caricamento e alla loro applicazione nella pagina, può ridurre le dimensioni totali di quest`ultima e migliorare i tempi di rendering.

## Anatomia di un font web

{% include modules/takeaway.liquid list=page.key-takeaways.anatomy %}

Un font web è una raccolta di glifi, ciascuno dei quali rappresenta una forma vettoriale che descrive una lettera o un simbolo. Di conseguenza, le dimensioni di un particolare file font sono determinate da due semplici variabili: la complessità dei percorsi vettoriali di ogni glifo e il numero di glifi di un determinato font. Ad esempio, OpenSans, uno dei font web più popolari, contiene 897 glifi, che includono caratteri latini, greci e cirillici.

<img src="images/glyphs.png" class="center" alt="Tabella dei glifi font">

Nella scelta di un font, è importante considerare i set di caratteri supportati. Per localizzare i contenuti di una pagina in più lingue, dovrai utilizzare un font che sia in grado di garantire un aspetto e un`esperienza coerenti agli utenti. Ad esempio, la [famiglia di font Noto di Google](https://www.google.com/get/noto/) mira a supportare tutte le lingue del mondo. Tuttavia, ricorda che le dimensioni totali di Noto, incluse tutte le lingue, ammontano a più di 130 MB. Scarica ZIP! 

Naturalmente, l`utilizzo di font web richiede una progettazione attenta per garantire che la tipografia non intralci le prestazioni. Per fortuna, la piattaforma web offre tutte le primitive necessarie, mentre nella presente guida daremo un`occhiata pratica a come ottenere il meglio da entrambi i lati.

### Formati dei font web

Oggigiorno, sul web sono disponibili quattro famiglie di font: [EOT](http://en.wikipedia.org/wiki/Embedded_OpenType), [TTF](http://en.wikipedia.org/wiki/TrueType), [WOFF](http://en.wikipedia.org/wiki/Web_Open_Font_Format), e [WOFF2](http://www.w3.org/TR/WOFF2/). Sfortunatamente, nonostante l`ampia scelta, non esiste un unico formato universale che funzioni in tutti i browser, più o meno recenti: EOT funziona [solo su IE](http://caniuse.com/#feat=eot), TTF è [parzialmente supportato in IE](http://caniuse.com/#search=ttf), WOFF è maggiormente supportato [non disponibile in alcuni browser precedenti](http://caniuse.com/#feat=woff), mentre il supporto di WOFF 2.0 è ancora [in progress in molti browser](http://caniuse.com/#feat=woff2).

Dunque, come dobbiamo muoverci? Non esiste un unico formato che funzioni in tutti i browser, il che significa che dobbiamo utilizzare più formati per offrire un`esperienza coerente:

* Offrire la variante WOFF 2.0 ai browser che la supportano
* Offrire la variante WOFF alla maggior parte dei browser
* Offrire la variante TTF ai vecchi browser Android (precedenti al 4.4)
* Offrire la variante EOT ai vecchi browser IE (precedenti a IE9)
^

{% include modules/remember.liquid title="Note" list=page.notes.svg %}

### Riduzione delle dimensioni del font tramite compressione

Un font è una raccolta di glifi, ciascuno dei quali è composto da un insieme di percorsi de definiscono la forma del carattere. I singoli glifi sono naturalmente diversi, ma contengono comunque molte informazioni simili che possono essere compresse con GZIP o altro compressore compatibile: 

* I formati EOT e TTF non sono compressi per default: assicurati che i server siano configurati per applicare la [compressione GZIP](/web/fundamentals/performance/optimizing-content-efficiency/optimize-encoding-and-transfer#text-compression-with-gzip) quando utilizzi tali formati.
* WOFF ha una compressione integrata - assicurati che il tuo compressore WOFF utilizzi impostazioni ottimali. 
* WOFF2 utilizza algoritmi di pre-elaborazione e compressione personalizzati per garantire una riduzione di ~30% delle dimensioni file rispetto ad altri formati - vedi [report](http://www.w3.org/TR/WOFF20ER/).

Vale infine la pena notare che alcuni formati font contengono metadati aggiuntivi, quali informazioni di  [font hinting](http://en.wikipedia.org/wiki/Font_hinting) e [kerning](http://en.wikipedia.org/wiki/Kerning) che possono non essere necessarie su alcune piattaforme, ma che consentono un`ulteriore ottimizzazione delle dimensioni file. Consulta le opzioni di ottimizzazione messe a disposizione dal tuo compressore font e, se scegli questo percorso, assicurati di disporre delle infrastrutture adatte per testare e utilizzare tali font ottimizzati in ogni browser. Ad es., Google Fonts offre più di 30 varianti ottimizzate per ogni font, individuando e fornendo automaticamente la variabile ottimale per ogni piattaforma e browser.

{% include modules/remember.liquid title="Note" list=page.notes.zopfli %}

## Definire una famiglia di font con @font-face

{% include modules/takeaway.liquid list=page.key-takeaways.font-family %}

L`at-rule nel CSS @font-face ci consente di definire il percorso di un determinato font, nonché le sue caratteristiche stilistiche e i codepoint Unicode per cui deve essere utilizzato. È possibile utilizzare una combinazione di tali dichiarazioni @font-face per costruire una `famiglia di font` che il browser userà per stabilire quali font debbano essere scaricati e applicati alla pagina corrente. Diamo un`occhiata più da vicino a come funziona dietro le quinte.

### Selezione formato

Ogni dichiarazione @font-face fornisce il nome della famiglia di font, utilizzato come gruppo logico di dichiarazioni multiple, le [proprietà del font](http://www.w3.org/TR/css3-fonts/#font-prop-desc) quali stile, spessore ed estensione, e lo [src descriptor](http://www.w3.org/TR/css3-fonts/#src-desc), che specifica un elenco prioritario di percorsi per il font.

{% highlight css  %}
@font-face {
  font-family: 'Awesome Font';
  font-style: normal;
  font-weight: 400;
  src: local('Awesome Font'),
       url('/fonts/awesome.woff2') format('woff2'), 
       url('/fonts/awesome.woff') format('woff'),
       url('/fonts/awesome.ttf') format('ttf'),
       url('/fonts/awesome.eot') format('eot');
}

@font-face {
  font-family: 'Awesome Font';
  font-style: italic;
  font-weight: 400;
  src: local('Awesome Font Italic'),
       url('/fonts/awesome-i.woff2') format('woff2'), 
       url('/fonts/awesome-i.woff') format('woff'),
       url('/fonts/awesome-i.ttf') format('ttf'),
       url('/fonts/awesome-i.eot') format('eot');
}
{% endhighlight %}

Nota innanzitutto che gli esempi precedenti definiscono un`unica famiglia _Awesome Font_ con due stili (normale e _itaic_), ciascuno dei quali si riferisce a un insieme diverso di font. Ogni `src` descriptor contiene un elenco prioritario, separato da virgole, di varianti: 

* La direttiva `local()` ci consente di riferirci, caricare e utilizzare i font installati localmente.
* La direttiva `url()` dci consente di caricare font esterni, che possono contenere un suggerimento `format()` aggiuntivo che indichi il formato del font cui fa riferimento l`URL fornito.

^
{% include modules/remember.liquid title="Note" list=page.notes.local-fonts %}

Quando stabilisce che il font è necessario, il browser scorre l`elenco di risorse fornito nell`ordine specificato e prova a caricare la risorsa corretta. Ad esempio, seguendo l`esempio precedente:

1. Il browser esegue il layout di pagina e stabilisce quali varianti del font sono necessarie per il rendering del testo.
2. Per ogni font richiesto, il browser verifica se è disponibile localmente.
3. In caso contrario, il browser fa riferimento a definizioni esterne:
  * Se è presente un format hint, il browser verifica se è supportato prima di avviare il download, altrimenti passa al successivo.
  * Se non è presente alcun format hint, il browser scarica la risorsa.

La combinazione di direttive interne ed esterne con format hint idonei ci consente di specificare tutti i formati font disponibili e lasciare al browser la scelta: il browser definisce le risorse necessarie e seleziona il formato ottimale per nostro conto.

{% include modules/remember.liquid title="Note" list=page.notes.font-order %}

### Subset Unicode-range

Oltre alle proprietà del font, quali stile, spessore ed estensione, la norma @font-face ci consente di definire un insieme di codepoin Unicode supportati da ogni risorsa. Questo ci consente di suddividere un font Unicode vasto in subset più piccoli (ad es. latino, cirillico, greco) e scaricare soltanto i glifi necessari per il rendering del testo di una data pagina.

L`[unicode-range descriptor](http://www.w3.org/TR/css3-fonts/#descdef-unicode-range) ci consente di specificare un elenco delimitato da virgole di valori di range, ciascuno dei quali può collocarsi in uno dei tre gruppi seguenti:

* Codepoint singolo (ad es. U+416)
* Range di intervallo (ad es. U+400-4ff): indica il codepoint iniziale e quello finale di un range
* Range wildcard (ad es. U+4??): `?` caratteri indica una cifra esadecimale

Ad esempio, possiamo suddividere la nostra famiglia _Awesome Font_ nei subset Latino e Giapponese, ciascuno dei quali verrà scaricato dal browser in base alla necessità: 

{% highlight css %}
@font-face {
  font-family: 'Awesome Font';
  font-style: normal;
  font-weight: 400;
  src: local('Awesome Font'),
       url('/fonts/awesome-l.woff2') format('woff2'), 
       url('/fonts/awesome-l.woff') format('woff'),
       url('/fonts/awesome-l.ttf') format('ttf'),
       url('/fonts/awesome-l.eot') format('eot');
  unicode-range: U+000-5FF; /* Latin glyphs */
}

@font-face {
  font-family: 'Awesome Font';
  font-style: normal;
  font-weight: 400;
  src: local('Awesome Font'),
       url('/fonts/awesome-jp.woff2') format('woff2'), 
       url('/fonts/awesome-jp.woff') format('woff'),
       url('/fonts/awesome-jp.ttf') format('ttf'),
       url('/fonts/awesome-jp.eot') format('eot');
  unicode-range: U+3000-9FFF, U+ff??; /* Japanese glyphs */
}
{% endhighlight %}

{% include modules/remember.liquid title="Note" list=page.notes.unicode-subsetting %}

L`utilizzo di subset unicode-range e di file distinti per ogni variante stilistica del font ci consente di definire una famiglia di font composita più rapida ed efficiente da scaricare; il visitatore dovrà infatti scaricare soltanto le varianti e i subset necessari, senza essere obbligato a scaricare subset che potrebbe non utilizzare mai né visualizzare sulla pagina. 

detto ciò, unicode-range ha un piccolo difetto: [non tutti i browser lo supportano](http://caniuse.com/#feat=font-unicode-range), o almeno, non ancora. Alcuni browser ignorano semplicemente l'hint unicode-range e scaricano tutte le varianti, mentre altri non elaborano proprio la dichiarazione @font-face. Per risolvere tutto ciò, dobbiamo eseguire un "subsetting manuale" nelle vecchie versioni dei browser.

Poiché i vecchi browser non sono in grado di selezionare automaticamente i subset necessari, né di costruire un font composito, dobbiamo fornire un unico font che contenga tutti i subset necessari, nascondendo il resto al browser. Ad esempio, se la pagina utilizza soltanto caratteri latini, possiamo eliminare gli altri glifi, fornendo tale subset come unica risorsa. 

1. **Come possiamo stabilire quali sono i subset necessari?** 
  - Se il browser supporta unicode-range, allora selezionerà automaticamente il subset giusto. La pagina dovrà soltanto fornire i file di subset e specificare gli unicode-range applicabili nelle proprietà @font-face.
  - Se unicode-range non è supportato, allora la pagina dovrà nascondere tutti i subset non necessari, ovvero il developer dovrà specificare i subset necessari.
2. **Come facciamo a generare un subset di font?**
  - Utilizza lo [strumento pyftsubset] open-source(https://github.com/behdad/fonttools/blob/master/Lib/fontTools/subset.py#L16) per suddividere in subset ed ottimizzare i tuoi font.
  - Per alcuni font è consentito il subsetting manuale tramite parametri query personalizzati, utilizzabili per specificare manualmente il subset necessario per la pagina. Consulta la documentazione del tuo font provider.


### Selezione dei font e font synthesis

Ogni famiglia di font è composta da diverse varianti stilistiche (normale, grassetto, corsivo) e diversi spessori per ogni stile, ciascuno dei quali può contenere glifi di forme molto diverse, ad esempio con spaziature, dimensioni o forme diverse. 

<img src="images/font-weights.png" class="center" alt="Spessore del font">

Ad esempio, il diagramma precedente illustra una famiglia di font che offre tre diversi spessori per il grassetto: 400 (normale), 700 (grassetto) e 900 (extra bold). Tutte le varianti intermedie (indicate in grigio) sono mappate automaticamente dal browser in base alla variante più prossima. 

<div class="quote">
  <div class="container">
    <blockquote class="quote__content g-wide--push-1 g-wide--pull-1 g-medium--push-1">Quando è specificato uno spessore per cui non esiste font-face, ne verrà utilizzata una con spessore simile. In generale, i caratteri in grassetto vengono associati a font-face con spessore maggiore e quelli normali a font face con spessore minore.
    <p><a href="http://www.w3.org/TR/css3-fonts/#font-matching-algorithm">Algoritmo di corrispondenza dei font CSS3</a></p>
    </blockquote>
  </div>
</div>

Un`operazione logica simile si applica alle varianti _italic_. Il font designer controlla quali varianti produrrà, mentre noi verifichiamo le varianti che useremo sulla pagina. Visto che ogni variante prevede un download separato, è buona norma scaricarne un numero esiguo! Ad esempio, possiamo definire due varianti di grassetto per la nostra famiglia _Awesome Font_: 

{% highlight css %}
@font-face {
  font-family: 'Awesome Font';
  font-style: normal;
  font-weight: 400;
  src: local('Awesome Font'),
       url('/fonts/awesome-l.woff2') format('woff2'), 
       url('/fonts/awesome-l.woff') format('woff'),
       url('/fonts/awesome-l.ttf') format('ttf'),
       url('/fonts/awesome-l.eot') format('eot');
  unicode-range: U+000-5FF; /* Latin glyphs */
}

@font-face {
  font-family: 'Awesome Font';
  font-style: normal;
  font-weight: 700;
  src: local('Awesome Font'),
       url('/fonts/awesome-l-700.woff2') format('woff2'), 
       url('/fonts/awesome-l-700.woff') format('woff'),
       url('/fonts/awesome-l-700.ttf') format('ttf'),
       url('/fonts/awesome-l-700.eot') format('eot');
  unicode-range: U+000-5FF; /* Latin glyphs */
}
{% endhighlight %}

Nell`esempio precedente, la famiglia _Awesome Font_ risulta composta da due risorse che coprono il medesimo insieme di glifi latini (U+000-5FF), ma offre due `spessori` diversi: normale (400) e grassetto (700). Tuttavia, che cosa accade se una delle nostre regole CSS specifica uno spessore diverso o definisce la proprietà font-style come corsivo?

* Se non è disponibile un font corrispondente, il browser lo sostituisce con quello più simile.
* Se non viene trovata alcuna corrispondenza stilistica (nell`esempio precedente, non dichiariamo alcuna variante corsiva), il browser sintetizzerà la propria variante. 

<img src="images/font-synthesis.png" class="center" alt="Font synthesis">

<div class="quote">
  <div class="container">
    <blockquote class="quote__content g-wide--push-1 g-wide--pull-1 g-medium--push-1">Gli autori devono sempre tenere a mente che tale approccio può non essere idoneo per alcuni caratteri, ad es. il cirillico, in cui il corsivo ha forme molto diverse. È sempre meglio utilizzare un font corsivo reale piuttosto che una versione sintetizzata.
    <p><a href="http://www.w3.org/TR/css3-fonts/#propdef-font-style">CSS3 font-style</a></p>
    </blockquote>
  </div>
</div>

L`esempio precedente illustra le differenze tra font reale e sintetizzato per Open-Sans; tutte le varianti sintetizzate sono generate dal medesimo font con spessore 400. Come puoi vedere, i risultati sono notevolmente diversi. I dettagli di generazioni delle varianti grassetto e diagonale non sono specificati. I risultati varieranno quindi da browser a browser, e dipenderanno anche molto dal font.

{% include modules/remember.liquid title="Note" list=page.notes.synthesis %}


## Ottimizzazione di caricamento e rendering

{% include modules/takeaway.liquid list=page.key-takeaways.font-crp %}

Un font web `completo` che includa tutte le varianti stilistiche, che potrebbero non servirci, nonché tutti i glifi, che potremmo non utilizzare mai, può comportare un download di diversi megabyte. Per risolvere tale problema, la norma CSS @font-face è finalizzata specificatamente a consentire la suddivisione della famiglia di font in una raccolta di risorse: subset unicode, varianti di stile distinte, e così via. 

Stabilito ciò il browser definisce i subset e le varianti necessarie e scarica l`insieme minimo richiesto per il rendering del testo. Tale percorso risulta particolarmente conveniente, ma, se non stiamo attenti, può anche creare un collo di bottiglia nel percorso di rendering critico, ritardandolo; un effetto che sicuramente vogliamo evitare! 

### Font web e percorso di rendering critico

Il lazy load di un font implica un aspetto importante che può ritardare il rendering del testo: il browser deve [costruire il render tree](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-tree-construction), che dipende a sua volta dagli alberi DOM e CSSOM, per poter capire quali risorse saranno necessarie per il rendering. Di conseguenza, le richieste di font sono ritardate dopo altre risorse critiche, e il browser può bloccare il rendering del testo fino al recupero della risorsa.

<img src="images/font-crp.png" class="center" alt="Percorso di rendering critico dei font">

1. Il browser richiede il documento HTML
2. Il browser inizia a scansionare la risposta HTML e a costruire il DOM
3. Il browser scopre risorse CSS, JS e di altro tipo e indirizza le richieste
4. Il browser crea il CSSOM una volta ricevuto tutto il contenuto CSS e lo combina con l`albero DOM per costruire il render tree
  * Una volta che il render tree indica quali varianti sono necessarie per il rendering del testo specifico presente sulla pagina, vengono inviate le richieste di font
5. Il browser definisce il layout e inserisce i contenuti sullo schermo
  * Se il font non è ancora disponibile, il browser potrebbe non renderizzare alcun pixel di testo
  * Non appena il font è disponibile, il browser inserisce i pixel di testo

La `competizione` tra la prima rappresentazione dei contenuti della pagina, che può essere realizzata subito dopo la creazione del render tree, e la richiesta di risorse font, è quella che crea il `problema degli spazi bianchi` in cui il browser deve renderizzare il layout della pagina ma omette qualsiasi testo. Il risultato differisce tra i diversi browser:

* Safari effettua il rendering del testo soltanto una volta completato il download del font.
* Chrome e Firefox bloccano il rendering per 3 secondi, dopo i quali utilizzano un font alternativo e, una volta scaricato il font, renderizzano nuovamente il testo in base ad esso.
* IE esegue immediatamente il rendering con il font alternativo se quello richiesto non è ancora disponibile, per poi eseguire nuovamente il rendering una volta completato il download.

Vi sono vantaggi e svantaggi in ogni strategia di rendering: alcuni trovano fastidioso il re-rendering, mentre altri preferiscono visualizzare immediatamente i risultati e non si preoccupano del refresh della pagina una volta terminato il download del font; non ne discuteremo in dettaglio in questa sede. Il punto importante è che il lazy load riduce il numero di byte, ma può anche potenzialmente ritardare il rendering del testo. Vediamo quindi come possiamo ottimizzare tale aspetto.

### Ottimizzazione del rendering del font con Font Loading API

[Font Loading API](http://dev.w3.org/csswg/css-font-loading/) offre un`interfaccia di scripting per definire e manipolare le font face CSS, tracciarne il download e sostituire il loro comportamento lazy load. Ad esempio, se siamo sicuri che sarà necessaria una specifica variante del font, possiamo definirla e comunicare al browser di iniziare immediatamente a recuperare la risorsa:

{% highlight javascript %}
var font = new FontFace("Awesome Font", "url(/fonts/awesome.woff2)", {
  style: 'normal', unicodeRange: 'U+000-5FF', weight: '400'
});

font.load(); // don't wait for render tree, initiate immediate fetch!

font.ready().then(function() {
  // apply the font (which may rerender text and cause a page reflow)
  // once the font has finished downloading
  document.fonts.add(font);
  document.body.style.fontFamily = "Awesome Font, serif";

  // OR... by default content is hidden, and rendered once font is available
  var content = document.getElementById("content");
  content.style.visibility = "visible";

  // OR... apply own render strategy here... 
});
{% endhighlight %}

inoltre, potendo verificare lo stato del font (tramite [check()](http://dev.w3.org/csswg/css-font-loading/#font-face-set-check)) e tracciarne il download, possiamo anche definire una strategia personalizzata per il rendering del testo sulle nostre pagine: 

* Possiamo bloccare il rendering fino a quando il font non è disponibile.
* Possiamo applicare un timeout personalizzato per ogni font.
* Possiamo utilizzare il font alternativo per sbloccare il rendering e inserire un nuovo stile che utilizzi il font desiderato una volta disponibile.

La cosa migliore è mixare e combinare le strategie precedenti per i diversi contenuti della pagina; ad esempio, bloccare il rendering di alcune sezioni fino a quando il font non è disponibile, utilizzare un font alternativo e poi renderizzarlo una volta terminato il download del font, specificare timeout diversi, e così via. 

{% include modules/remember.liquid title="Note" list=page.notes.webfontloader %}

### Ottimizzazione del rendering con inlining

Una semplice strategia alternativa per utilizzare Font Loading API consiste nell`eliminare il `problema degli spazi bianchi` con priorità alta, poiché sono necessari per costruire il CSSOM.

* Gli stylesheet CSS come media query corrispondenti sono scaricati automaticamente dal browser con priorità alta, essendo necessari per costruire il CSSOM.
* L`inlining dei dati del font nello stylesheet CSS obbliga il browser a scaricare il font senza priorità alta e senza attendere il render tree, agendo quindi come override manuale sul comportamento lazy load predefinito.

La strategia di inlining non è flessibile e non ci consente di definire alcun timeout personalizzato o strategia di rendering per contenuti diversi, ma rappresenta comunque una soluzione semplice ed efficace in tutti i browser. Per ottenere migliori risultati, separa i font sottoposti ad inlining in stylesheet separati e imposta una proprietà max-age lunga; in tal modo, all`aggiornamento del CSS, non obbligherai i visitatori a riscaricare i font. 

{% include modules/remember.liquid title="Note" list=page.notes.font-inlining %}

### Ottimizzazione per il riutilizzo con HTTP Caching

Di norma, io font sono risorse statiche che non conoscono aggiornamenti frequenti. Di conseguenza, sono ideali per una scadenza max-age lunga; assicurati di specificare sia una[intestazione ETag condizionale](/web/fundamentals/performance/optimizing-content-efficiency/http-caching#validating-cached-responses-with-etags), sia una [policy di Cache-Control ottimale](/web/fundamentals/performance/optimizing-content-efficiency/http-caching#cache-control) per tutti i font. 
    
Non è necessario immagazzinare i font nel local storage o attraverso altri meccanismi; ciascuno di essi dispone del proprio insieme di gotcha di prestazioni. La cache HTTP del browser, in combinazione con Font Loading API o con la webfontloader library, offre il meccanismo migliore e più efficace per inviare dei font al browser.


## Checklist di ottimizzazione

Contrariamente a quanto si crede, l`utilizzo di font web non ritarda necessariamente il rendering della pagina né ha un impatto negativo su altre prestazioni. Un utilizzo dei font ottimizzato può garantire all`utente un`esperienza migliore sotto ogni punto di vista: branding perfetto, migliore leggibilità, fruibilità e ricercabilità, offrendo al contempo una risoluzione multipla scalabile che ben si adatta a qualsiasi formato e risoluzione del monitor. Non temere di utilizzare i font web! 

Certo, un`applicazione non ponderata può causare ritardi evitabili e download eccessivamente pesanti. È per questo che dobbiamo svecchiare i nostri strumenti di ottimizzazione e supportare il browser ottimizzando i font stessi e il relativo recupero e utilizzo sulle nostre pagine. 

1. **Verifica e controlla il tuo utilizzo dei font:** non utilizzare troppi font sulle tue pagine e, per ogni font, minimizza il numero di varianti utilizzate. Ciò ti consentirà di offrire ai tuoi utenti un`esperienza più rapida e uniforme.
2. **Suddividi i font in subset:** molti font possono essere suddivisi in subset o in più unicode-range, così da utilizzare soltanto i glifi richiesti da una pagina specifica - così facendo, le dimensioni file si riducono, mentre la velocità di download della risorsa aumenta. Tuttavia, nella definizione dei subset, fa attenzione ad ottimizzarli per il riutilizzo dei font; ad es., per ogni pagina non scaricare un set di caratteri diverso ma sovrapposto. È buona norma creare subset in base allo script, ad es. latino, cirillico, e così via.
3. **Orttimizza il formato dei font per ogni browser:** ogni font dovrebbe essere fornito in formato WOFF2, WOFF, EOT e TTF. Assicurati di comprimere con GZIP i formati EOT e TTF, non compressi per default.
4. **Specifica le policy ottimali di riconvalida e caching:** i font sono risorse statiche che vengono aggiornate molto raramente. Assicurati che i tuoi server prevedano un timestamp max-age di lunga durata e un token di riconvalida per consentire un riutilizzo efficace tra pagine diverse.
5. **Utilizza FontLoading APO per ottimizzare il percorso di rendering critico:** il lazy loading predefinito può comportare un ritardo nel rendering del testo. Font Loading API ci consente di evitarlo per determinati font, specificando una strategia di rendering e timeout personalizzata in base ai diversi contenuti della pagina. Per le versioni più vecchie dei browser che non supportano l`API, puoi utilizzare la webfontloader JavaScript library o la strategia di inlining del CSS.

{% endwrap %}

