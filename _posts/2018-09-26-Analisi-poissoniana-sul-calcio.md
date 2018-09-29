---
title: "Analisi poissoniana sul Calcio"
excerpt: "Questo post espone un'analisi statistica condotta sui dati disponibili sulla serie A della stagione 2017-2018"
layout: single
header:
  overlay_image: /images/2018-09-26-Analisi-poissoniana-sul-calcio/Roma_stadio_Olimpico.jpg
  overlay_filter: 0.5
  caption: "Stadio Olimpico di Roma"
categories:
  - data analyst
  - python
  - calcio
tags:
  - calcio
  - football
  - soccer
  - python
  - scrapy
  - d3
author: "V A"
date: "26 Settembre 2018"
---
## Intro
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Aenean non fermentum mauris, sit amet tempus mi. Etiam sit amet lacus faucibus, malesuada arcu id, semper metus. Pellentesque molestie luctus accumsan. Etiam suscipit eu dui nec cursus. In justo quam, ullamcorper non faucibus quis, auctor eget lectus. Vivamus ac est iaculis, pellentesque felis a, ullamcorper elit. Donec vel nibh nisl. Morbi commodo leo non ipsum posuere, id imperdiet arcu tristique.

Donec et luctus sem. Nullam pellentesque ipsum vitae tincidunt faucibus. Maecenas tempor, dui et ullamcorper ullamcorper, purus urna porttitor leo, sed bibendum risus diam eget felis. Donec ipsum purus, iaculis nec sodales vel, cursus sed justo. In id turpis rutrum, tincidunt ipsum nec, vulputate dolor. Maecenas interdum, sapien et aliquam suscipit, sem nunc sollicitudin elit, et porta elit leo ac lectus. Morbi vel fermentum lorem.

## Approccio iniziale
Come primo approccio bisogna trovare i dati su cui poi dobbiamo andare a lavorare. Effettuando una ricerca sul sito della Serie A non ho trovato un database ufficiale, quindi ho dovuto ripiegare su siti esterni. Ve ne sono diversi a disposizione, ma sicuramente quelli con uno storico e una disponibilità maggiore sono [www.football-data.co.uk](http://www.football-data.co.uk/italym.php) e [http://www.footstats.co.uk/](http://www.footstats.co.uk/index.cfm?task=league_full).

Mentre nel primo sono disponibile le statistiche di tutte le partite giocate per i principali campionati (e leghe minori....), nel secondo sono forniti i valori riassuntivi della classifica (suddivisi per half e full time) e inoltre, per entrambi i siti, è possibile scaricare i file in formato csv.

Nella tabella successiva sono riportati gli esempi pratici di qualche dato usato durante l'analisi:

{% include /prova/prova.html %}

Il primo step da seguire è l'analisi dei dati, che in questo caso verrà fatto con python (e in particolare pandas).

<figure class="half">
    <img src="/images/2018-09-26-Analisi-poissoniana-sul-calcio/python.png"  class="center">
    <figcaption class="center">Immagine presa da <a href="https://imgs.xkcd.com/comics/python.png">xkcd</a></figcaption>
</figure>

![your image]({{ '/images/2018-09-26-Analisi-poissoniana-sul-calcio/python.png' | https://imgs.xkcd.com/comics/python.png }})



Analizzando i dati delle partite giocate nella scorsa stagione possiamo notare tra la media gol delle squadre in case e fuori casa.
