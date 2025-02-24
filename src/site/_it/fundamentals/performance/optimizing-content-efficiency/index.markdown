---
layout: section
title: "Ottimizzazione dell`efficienza dei contenuti"
description: "La quantità di dati scaricati da ogni app cresce costantemente. Per garantire prestazioni ottimali dobbiamo ottimizzare l`utilizzo di ogni singolo byte!"
introduction: "La nostra applicazione web continua a crescere in termini di ambito di applicazione, ambizioni e funzionalità, ed è un bene. Tuttavia, la marcia inarrestabile verso un web sempre più ricco porta con sé un`altra tendenza: la quantità di dati scaricati da ogni applicazione continua a crescere incessantemente. Per garantire prestazioni ottimali dobbiamo ottimizzare l`utilizzo di ogni singolo byte di dati!"
article:
  written_on: 2014-04-01
  updated_on: 2014-04-29
  order: 2
id: optimizing-content-efficiency
authors:
  - ilyagrigorik
collection: performance
---

{% wrap content%}

<style>
  img, video, object {
    max-width: 100%;
  }

  img.center {
    display: block;
    margin-left: auto;
    margin-right: auto;
  }
</style>

Come appare una moderna applicazione web? [HTTP Archive](http://httparchive.org/) può aiutarci a rispondere a questa domanda. Il progetto traccia il modo in cui il web è costruito, infiltrandosi periodicamente nei siti più popolari (300.00+ in base all`elenco Alexa Top 1M), registrando e mettendo a confronto i dati relativi a risorse, tipi di contenuti e altri metadati per ogni singola destinazione.

<img src="images/http-archive-trends.png" class="center" alt="Trend HTTP Archive">

<table class="table-4">
<colgroup><col span="1"><col span="1"><col span="1"><col span="1"></colgroup>
<thead>
  <tr>
    <th></th>
    <th>50° percentile</th>
    <th>75° percentile</th>
    <th>90° percentile</th>
  </tr>
</thead>
<tr>
  <td data-th="type">HTML</td>
  <td data-th="50%">13 KB</td>
  <td data-th="75%">26 KB</td>
  <td data-th="90%">54 KB</td>
</tr>
<tr>
  <td data-th="type">Immagini</td>
  <td data-th="50%">528 KB</td>
  <td data-th="75%">1213 KB</td>
  <td data-th="90%">2384 KB</td>
</tr>
<tr>
  <td data-th="type">JavaScript</td>
  <td data-th="50%">207 KB</td>
  <td data-th="75%">385 KB</td>
  <td data-th="90%">587 KB</td>
</tr>
<tr>
  <td data-th="type">CSS</td>
  <td data-th="50%">24 KB</td>
  <td data-th="75%">53 KB</td>
  <td data-th="90%">108 KB</td>
</tr>
<tr>
  <td data-th="type">Altro</td>
  <td data-th="50%">282 KB</td>
  <td data-th="75%">308 KB</td>
  <td data-th="90%">353 KB</td>
</tr>
<tr>
  <td data-th="type"><strong>Totale</strong></td>
  <td data-th="50%"><strong>1054 KB</strong></td>
  <td data-th="75%"><strong>1985 KB</strong></td>
  <td data-th="90%"><strong>3486 KB</strong></td>
</tr>
</table>

I dati precedenti definiscono il trend di crescita del numero di byte scaricati per alcune destinazioni web popolari tra gennaio 2013 e gennaio 2014. Naturalmente, non tutti i siti crescono alla medesima velocità o richiedono la stessa quantità di dati; ecco perché abbiamo sottolineato i diversi quantili di distribuzione: 50° (medio), 75° e 90°.

Un sito medio all`inizio del 2014 è composto da 75 richieste che aggiungono fino a 1054 KB di byte trasferiti totali, e il numero totale di byte (e richieste) è aumentato a ritmo costante nel corso dell`anno precedente. Tale dato da solo non sarebbe così sorprendente, ma comporta delle importanti implicazioni prestazionali: è vero, la velocità di internet sta aumentando sempre di più, ma aumenta in modo diverso in paesi diversi, e molti utenti sono ancora soggetti a limiti per il download di dati e piani tariffari costosi, specialmente su dispositivi mobili.

A differenza delle controparti desktop, le applicazioni web non richiedono una procedura di installazione distinta: basta inserire l`URL ed ecco che possiamo navigare. Questa è una delle funzioni chiave del web. Tuttavia, perché ciò sia possibile, dobbiamo spesso recuperare dozzine, talvolta centinaia di risorse disparate, che aggiungono tutte megabyte di dati e devono essere individuate in centinaia di millisecondi per consentirci di vivere l`esperienza web istantanea che ci aspettiamo.**

Poter vivere un`esperienza simile conoscendo tali requisiti non è cosa da poco; ecco perché l`ottimizzazione dell`efficienza dei contenuti è fondamentale, eliminando i download non necessari, ottimizzando la codifica dei dati trasferiti per ogni risorsa tramite diverse tecniche di compressione e avvalendosi del caching, laddove possibile, per eliminare eventuali download ridondanti.

{% endwrap %}

