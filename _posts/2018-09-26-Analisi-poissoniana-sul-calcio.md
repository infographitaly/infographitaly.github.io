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
{: .notice--success .text-center}
Subito dopo però affermiamo anche la seguente ipotesi, che chiameremo <i>ipotesi alternativa</i>


Le due medie sono diverse
{: .notice--success .text-center}

Quale è vera? Per rispondere correttamente dobbiamo rivolgerci al signor William Sealy Gosset, noto anche come Student.

<div class="dida_right">
  <h4>Info box</h4>
W. S. Gosset è stato un chimico e matematico agli inizi del 900. Amico di [Pearson](http://www-history.mcs.st-andrews.ac.uk/Biographies/Pearson.html), lavorò alla Guiness ed è lì che sviluppò il t-test.
Poiché prima di lui un dipendente della fabbrica pubblicò il segreto della produzione della birra, per prevenire la fuga di informazioni la società impedì ai suoi dipendenti la pubblicazione di articoli contenente informazioni sensibili.
Per ovviare a questo problema Gosset pubblicò i suoi articoli con lo pseudonimo di Student. Ecco perché il test prende il nome di test di student!
fu proibito di 
</div>
Il test di student, noto anche come t-test, aiuta a capire se la differenza tra due medie è significativa o è dovuta al caso. Per fare ciò, va calcolato il valore <i>t</i> e confrontato i valori riportati nelle [tavole](http://stat.unicas.it/vistoccoDownload/stat/materiale/tStudent.pdf).
Dalla libreria di python <i>scipy</i> usiamo la funzione ttest_ind_from_stats del modulo <i>stats</i>. Calcoliamo i valori medi e le varianze dei gol fuori casa e in casa,
```
from scipy.stats import ttest_ind_from_stats
mean_home = Partite_1718['FTHG'].mean() #Calolo dei valori medi
mean_away = Partite_1718['FTAG'].mean() #
var_home = Partite_1718['FTHG'].var()   #Calcolo delle varianze
var_away = Partite_1718['FTAG'].var()   #
dof = len(Partite_1718['FTHG'])+len(Partite_1718['FTAG'])-2
t_stud, p_val = ttest_ind_from_stats(mean_home,var_home,dof,mean_away,var_away,dof)
print("===mean value Home/mean value Away===")
print(mean_home,"/",mean_away)
print("===var value Home/var value Away===")
print(var_home,"/",var_away)
print("===T-Student/P-Value===")
print(t_stud,"/",p_val)
```
I valori ottenuti sono:

<table class="table table-bordered table-hover table-condensed">
<thead><tr><th title="Field #1">Valori</th>
<th title="Field #2">Casa</th>
<th title="Field #3">Trasferta</th>
</tr></thead>
<tbody><tr>
<td >Media</td>
<td>1.45526</td>
<td >1.22105</td>
</tr>
<tr>
<td >Varianza</td>
<td>1.720958</td>
<td >1.41275</td>
</tr>
</tbody></table>

In particolare...

<table class="table table-bordered table-hover table-condensed">
<tbody><tr>
<td >T-Student</td>
<td>2.04783</td>
</tr>
<tr>
<td >P-Value</td>
<td>0.04092</td>
</tr>
</tbody></table>
Abbiamo quindi ottenuto un valore di t di 2.04783 e, considerando i nostri gradi di libertà (758), allora possiamo stabilire
che, dato il valore di p-value di 0.04 esso è minore dei valori di soglia del 95% e del 99% e, quindi possiamo rigettare l'ipotesi nulla e accettare quella alternativa.
Quello che stiamo sostanzialmente dicendo è che i due valori medi sono "oggettivamente" differenti.

Le motivazione su quesa differenza sono molteplici, come, ad esempio, il supporto dei tifosi (you'll never walk alone!).

In principio quindi dovremmo considerare anche questo fattore del giocare in <i>Casa</i>, ma per ora passeremo oltre e (forse) ci torneremo più in la.

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


Effettivamente l'andamento tra i valori calcolati e quelli misurati sembrano essere simili...ma questo non basta, poiché dobbiamo affidarci ancora una volta ad un test. 
Iniziamo con l'affermare, quindi, che 

La distribuzione dei goal totali seguono approssimativamente una distribuzione di Poisson.
{: .notice--success}
Per poter dire se questa affermazione sia vero o meno, dobbiamo ricorrere al test del chi quadro. 
<div class="dida_right">
  <h4>Info box</h4>
Il test del chi quadro è un test necessario nel caso si voglia verificare che le frequenze dei valori osservati abbiano un andamento simile alle frequenze teoriche di una distribuzione di probabilità scelta.
La formula per ottenere il valore &chi;<SUP>2</SUP> è
$$
\chi^{2}=\sum _{i=1}^{k}\frac{(o_{i}-e_{i})^{2}}{e_{i}}  
$$
dove o<SUB>i</SUB> e e<SUB>i</SUB> sono, rispettivamente, le frequenze osservate e teoriche. Questo test fu ideato da Karl Pearson, padre del Pearson citato prima!
</div>Partendo dalle frequenze delle osservazioni misurate (cioè quante volte un certo numero di gol sono stati effettuati nelle partite), 
ottenendo le frequenze teoriche, possiamo vedere quante queste differiscono tra loro. Il valore $$\chi ^{2}$$ quantifica questa informazione e, tramite le tavole [tavole della distribuzione](http://www00.unibg.it/dati/corsi/40025/74822-tavola_chi2.pdf)
è possibile determinare se l'ipotesi nulla sia verificata o meno.

Passando dalle parole ai fatti, vediamo lo script python usato per ottenere i risultati:


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
	
```

Da questo breve programma, ciò che abbiamo ottenuto è

<table class="table table-bordered table-hover table-condensed">
<thead><tr><th title="Field #1">Valori</th>
<th title="Field #2">Casa</th>
<th title="Field #3">Trasferta</th>
</tr></thead>
<tbody><tr>
<td >Chi2 Stat</td>
<td>8.53308</td>
<td >8.16788</td>
</tr>
<tr>
<td >P-Value</td>
<td>0.28794</td>
<td >0.31802</td>
</tr>
</tbody></table>
	





Questo ci dice che, per esempio, nel caso dei gol in casa i valori di $$\chi$$ quadro sono 8.53308 con 7 gradi di libertà e un p-value $$(P(> X^2) = 0.28794)$$,
 il ché significa che la probabilità del valore di chi quadro nelle [tavole della distribuzione del chi quadrato](http://www00.unibg.it/dati/corsi/40025/74822-tavola_chi2.pdf) di essere maggiore è
 del ≈ 29%. Perciò è possibile dire che c'è il 29% di possibilità che le due distribuzioni di valori siano uguali e che quindi l'ipotesi nulla può essere accettata!

<div style="align: center; text-align:center;">
    <img src="https://i.gifer.com/Fh5.gif"  class="center">
	<div class="caption"><small>Felicità!</small></div>
</div>

## Andiamo avanti...
Procediamo e cerchiamo di costruire il modello. Quello che abbiamo è che che la distribuzione delle frequenze dei gol segue una distribuzione di Poisson.
 Ma come ci può aiutare a predirre la futura vincitrice della serie A?
Possiamo calcolare la probabilità di vittoria di una squadra rispetto ad un'altra, costruendo un modello generalizzato di regressione lineare Poissoniano. 
Fermiamoci un attimo e respiriamo
Il modello di regressione lineare è un modo generale per cercare di trovare funzione che rappresenti tutti punti in un grafico. Se i punti non sono distribuiti normalmente (cioè non seguono una distribuzione di Gauss),
allora dobbiamo ricorrere al modello <b>generalizzato</b> di regressione lineare e poiché nel nostro caso i punti seguono una distribuzione di Poisson, allora ecco che useremo il glm Poissoniano!
Lo script python che farà ciò è:
```
import statsmodels.api as sm
import statsmodels.formula.api as smf

goal_model_data = pd.concat([Partite_1718[['HomeTeam','AwayTeam','FTHG']].assign(home=1).rename(
            columns={'HomeTeam':'team', 'AwayTeam':'opponent','FTHG':'goals'}),
           Partite_1718[['AwayTeam','HomeTeam','FTAG']].assign(home=0).rename(
            columns={'AwayTeam':'team', 'HomeTeam':'opponent','FTAG':'goals'})])
poisson_model = smf.glm(formula="goals ~ home + team + opponent", data=goal_model_data, 
                        family=sm.families.Poisson()).fit()
poisson_model.summary()
```

Per ottenere

{% include /Articolo1/Tabella_modello_1.html %}
{% include /Articolo1/Tabella_modello_2.html %}
Dalla prima tabella possiamo estrarre qualche informazione, come ad esempio sappiamo che la variabile dipendente sono i gol, che il numero di dati disponibili sono 760 e che la funzione usata, ovvero la fuzione link, è il logaritmo.
Nella seconda, invece, abbiamo diverse colonne che rappresentano:
- il coefficiente e l'errore associato
- il valore z, pari al rapporto tra il coefficiente e l'errore associato, e il relativo P-Value
- l'intervallo di confidenza
Di tutti questi valori quello che sicuramente ci interessa maggiormente è il P-value, poiché ci permette di capire quale variabile influisce maggiormente: infatti se fissiamo il livello di signicatifità 
del p-value al 5%, allora possiamo notare che sicuramente una variabile importante potrebbe essere la media gol della Juventus in casa (pari al 2%) o, anche, quello della Sampdoria fuori casa (pari al 3.6%).
I valori predenti nella colonna <b>Coef</b> rappresentano i valori della funzione logaritmo (per un maggiore dettaglio vedere [qui](http://www.dima.unige.it/~rogantin/ModStat/Epidemiologia/teoria.pdf)).
Quelli indicati con Team[Squadra] indicano, quando positivi, una maggiore predisposizione al gol, mentre, quando più vicini al valore 0, a nessun effetto. Viceversa i valori indicati con Opponent[squadra]
indicano quelle squadre con una maggiore capacità di resistere al gol. Alla fine della colonna troviamo il valore <i>home</i> il quale indica la maggiore disposizione delle squadre che giocano in casa a segnare maggiormente rispetto a quelle in trasferta.
Passiamo a un caso pratico, che tutte queste parole confondono.
METTICI LA GIF
Prendiamo come esempio la Juventus: essa presenta come valore team 0.3966, mentre come valore opponent -0.4564. Possiamo fare un confronto con la Lazio la quale presenta una capacità di fare goal maggiore rispetto alla Vecchia Signora (infatti presenta un valore home di 0.4578),
ma ha una capacità di difendersi nettamente peggiore (valore opponent 0.2630). Infine possiamo notare che le squadre con una migliore difesa sono quelle che risultano essere entrate in Champion's League l'anno scorso (quindi è vero che"In Serie A vince chi non subisce goal"!)
<table class="table table-bordered table-hover table-condensed">
<thead><tr><th title="Field #1">Squadre</th>
<th title="Field #2">Casa</th>
<th title="Field #3">Trasferta</th>
</tr></thead>
<tbody><tr>
<td >1</td>
<td>Juventus</td>
<td >-0.4564</td>
</tr>
<tr>
<td >2</td>
<td>Napoli</td>
<td >-0.2762</td>
</tr>
<tr>
<td >3</td>
<td>Roma</td>
<td >-0.3279</td>
</tr>
<tr>
<td >4</td>
<td>Inter</td>
<td >-0.2536</td>
</tr>
</tbody></table>
Cerchiamo ora di avvicinarci alla simulazione del campionato di Serie A.
Il primo step è simulare l'esito di una partita tra due squadre, partendo da quanto ottenuto fino ad ora.
Prendiamo la Juventus e il Napoli, rispettivamente, prima e seconda classificata del passato campionato italiano di Serie A.
Supponiamo che, per semplicità, i possibili risultati varino in un intervello che va dal 3 a 0 in casa allo 0 a 3 per la squadra in trasferta (ovvero 3-0,3-1,3-2,3-3,0-3,1-3,2-3).
Allora quello che avremo è una matrice di probabilità, che sembra una cosa complicata ma in realtà è molto semplice: 
lunghe le righe abbiamo la probabilità che il Napoli possa segnare quel numero di gol entro la fine della partita, mentre, viceversa, sulle colonne abbiamo le probabilità che la Juventus possa segnare quel goal.
Facendo la simulazione abbiamo:
<table class="table table-bordered table-hover table-condensed">

<tbody>
<tr>
<td >Napli\Juventus</td>
<td>0</td>
<td >1</td>
<td >2</td>
<td >3</td>
</tr>

<tr>
<td >0</td>
<td>0.10069245 </td>
<td >0.089384</td>
<td >0.03967278</td>
<td >0.01173909</td>
</tr>
<tr>
<td >1</td>
<td >0.1417741 </td>
<td >0.12585189</td>
<td >0.05585893</td>
<td >0.01652853</td>
</tr>
<tr>
<td >2</td>
<td >0.09980834</td>
<td >0.08859918</td>
<td >0.03932444</td>
<td >0.01163601</td>
</tr>
<tr>
<td >3</td>
<td >0.04684309</td>
<td >0.04158229</td>
<td >0.01845616</td>
<td >0.00546114</td>
</tr>
</tbody></table>


