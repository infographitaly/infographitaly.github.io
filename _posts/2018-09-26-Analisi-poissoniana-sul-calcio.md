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
mathjax: true
author: "V A"
date: "26 Settembre 2018"
toc: true
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

<div style="align: center; text-align:center;">
    <img src="https://imgs.xkcd.com/comics/python.png"  class="center">
	<div class="caption"><small>Immagine presa da <a href="https://imgs.xkcd.com/comics/python.png">xkcd</a></small></div>
</div>


Per prima cosa dobbiamo importare pandas (necessaria per lavorare con i dataframe) e poi usare la funzione <em>open_csv</em> per aprire (indovinate un po?!) i csv. 
In seguito abbiamo calcolato la media dei gol segnati in casa e fuori casa specificando la colonna e utilizzando <i>.mean()</i>. 
```
import pandas
Dati_1718 = pd.read_excel('path\Dati_1718.xlsx')
Partite_1718 = pd.read_excel('path\Partite_1718.xlsx')

mean_home_FT = Partite_1718['FTHG'].mean()
mean_away_FT = Partite_1718['FTAG'].mean()


Media Gol in casa    1.45526
Media Gol fuori casa    1.22105
```
AH!
Possiamo subito notare una caratteristica di questi dati! La media gol in casa è maggiore che fuori casa...ma come sempre dobbiamo affidarci a qualcosa di oggettivo per sapere se ciò è vero!
Quindi facciamo un'ipotesi, che possiamo chiamare <i>ipotesi nulla</i>, e diciamo che 


La media gol in casa è uguale alla media dei gol segnati fuori casa.
{: .notice--primary}
Subito dopo però affermiamo anche la seguente ipotesi, che chiameremo <i>ipotesi alternativa</i>


Le due medie sono diverse
{: .notice--primary}




Quale è vera? Per rispondere correttamente dobbiamo rivolgerci al signor William Sealy Gosset, noto anche come Student. BLABLABLABLABLABLABLABLABLABLABLABLABLABLABLABLABLABLABLABLABLA
{% capture notice-text %}
W. S. Gosset è stato un chimico e matematico agli inizi del 900. Amico di [Pearson](http://www-history.mcs.st-andrews.ac.uk/Biographies/Pearson.html), lavorò alla Guiness ed è lì che sviluppò il t-test.
Poiché prima di lui un dipendente della fabbrica pubblicò il segreto della produzione della birra, per prevenire la fuga di informazioni la società impedì ai suoi dipendenti la pubblicazione di articoli contenente informazioni sensibili.
Per ovviare a questo problema Gosset pubblicò i suoi articoli con lo pseudonimo di Student. Ecco perché il test prende il nome di test di student!
fu proibito di 
{% endcapture %}
<div class="notice--info align-right text-right">
  <h4>Info box</h4>
  {{ notice-text | markdownify }}
</div>



e le motivazioni (penso) siano ovvie...una su tutte: il supporto dei propri tifosi (you'll never walk alone!).
In principio quindi dovremmo considerare anche questo fattore, ma per passeremo oltre e (forse) ci torneremo più in la.
A questo punto possiamo introdurre la distribuzione di Poisson.
<div style="align: center; text-align:center;">
    <img src="https://imgs.xkcd.com/comics/poisson.jpg"  class="center">
	<div class="caption"><small>Toh!Un'altra immagine presa da <a href="https://xkcd.com/12/">xkcd</a></small></div>
	<div class="caption"><small>(Una breve spiegazione della vignetta <a href="http://leganerd.com/2011/04/15/xkcd-poisson/">qui</a>)</small></div>
</div>
La distribuzione di poisson viene utilizzata quando si vuole conoscere la probabilità di ottenere un certo numero di successi per unità di tempo, sapendo che <i>mediamente</i> se ne verifica un numero $$\lambda$$:

$$
P\left( x \right) = \frac{e^{-\lambda} \lambda ^x }{x!}, \lambda>0
$$

Aspettate, c'è scritto mediamente? Ma allora è proprio quello che fa al caso nostro! Il problema è che ci sono alcune condizioni da rispettare (altrimenti il signor Siméon-Denis "El Tanque" Poisson ce accappotta co na manata):
- gli eventi devono essere indipendenti -> Questo non ci piace perché significa che se viene segnato un gol al primo minuto la squadra non ne risente e si comporta come se nulla fosse accaduto!
- la probabilità che in una determinata unità di tempo si verifichi un esito favorevole deve essere la stessa per tutte le unità -> Ci piace? Naturalmente no, in quanto questo significa che la probabilità di segnare un gol al primo tempo è la stessa che al secondo....ma i giocatori risento del calo fisico!
- il numero medio di successi per unità di tempo deve essere costante -> Finalmente una buona notizia...ah no...questo ci sta dicendo che la media gol si mantiene costante durante la partita (forse con Zeman è così)
Ma è un distrastro! Non ne va bene una! ... Ma lo è veramente? Potremmo ragionarci su e aggirare il problema considerando un'unità di tempo piuttosto larga e consona a quello che ci interessa a noi. Questa finestra temporale potrebbe essere proprio lunga 90 minuti e assumere così indipendenza! In questo modo avremmo risolto la maggior parte dei problemi! 
 

<div style="align: center; text-align:center;">
    <img src="http://www.nov-art.eu/img/92MinutiDiApplausi.gif"  class="center">
	<div class="caption"><small>Qui ci vogliono 92 minuti di applausi! (meglio 90...)</small></div>
</div>


Quindi possiamo fare un breve confronto:
{% include /Articolo1/home_obs_teo_poisson.html %}


Effettivamente l'andamento tra i valori calcolati e quelli misurati sembrano avere lo stesso...ma questo non basta, poiché dobbiamo affidarci al test del chi quadro per confermare la seguente ipotesi nulla
```
La distribuzione dei goal totali seguono approssimativamente una distribuzione di Poisson.
```
Fermi fermi fermi
Abbiamo detto un sacco di cose in una frase! Ipotesi nulla?! test del chicchirichì?!
Iniziamo con il dire che il test del chi quadro è un test in statistica che usa la variabile $$chi$$ elevata al quadrato per accettare o no l'ipotesi nulla. Ok, ma cos'è l'ipotesi nulla? Diciamo che
assumiamo una certa ipotesi di partenza e che vogliamo verificarla, allora questa possiamo considerarla come ipotesi nulla (o zero) [per un dettaglio maggiore consultare](http://www.quadernodiepidemiologia.it/epi/assoc/pro_sig.htm).
Senza aggiungere ulteriori formule, vediamo cosa abbiamo ottenuto:
Questo è lo script python:

```
from scipy.stats import chisquare

for state in ['Home','Away']:
    
    obs = df_FT[state].values

    sum_tot = np.sum(obs)
	
    if state == 'Home':
        mean_to_use = mean_home_FT
    else:
        mean_to_use = mean_away_FT


    exp = [poisson.pmf(i, mean_to_use)*sum_tot for i in range(0,len(obs))]

    chi2_stat, p_val = chisquare(np.array(obs), np.array(exp), ddof=0)
    print("==="+state+"===")    
    print("===Chi2 Stat===")
    print(chi2_stat)
    print("===P-Value===")
    print(p_val)
	
	
===Home===
===Chi2 Stat===
8.533080509475663
===P-Value===
0.28793818912205194
===Away===
===Chi2 Stat===
8.167882410677105
===P-Value===
0.31802337146494697

```


Questo ci dice che, per esempio, nel caso dei gol in casa i valori di $$\chi$$ quadro sono 8.53308 con 7 gradi di libertà e un p-value $$(P(> X^2) = 0.28794)$$,
 il ché significa che la probabilità del valore di chi quadro nelle [tavole della distribuzione del chi quadrato](http://www00.unibg.it/dati/corsi/40025/74822-tavola_chi2.pdf) di essere maggiore è
 del ≈ 29%. Perciò è possibile dire che c'è il 29% di possibilità che le due distribuzioni di valori siano uguali e che quindi l'ipotesi nulla può essere accettata!

<div style="align: center; text-align:center;">
    <img src="https://i.gifer.com/Fh5.gif"  class="center">
	<div class="caption"><small>Felicità!</small></div>
</div>

## Andiamo avanti...
Procediamo e cerchiamo di complicare il modello. Quello che abbiamo ottenuto è sapere che la distribuzione delle frequenze dei gol segue una distribuzione di Poisson.
 Ma come ci può aiutare a predirre la futura vincitrice della serie A?

