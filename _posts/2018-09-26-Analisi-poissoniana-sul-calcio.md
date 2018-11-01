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
  - Amchart
  - Chart.js
mathjax: true
author: "V A"
date: "30 Ottobre 2018"
toc: false
---
## Intro
Benvenuti al primo articolo di questo blog!

L'idea è quella di divulgare informazioni scientifiche tramite argomenti semplici. Come ad esempio (dovrebbe) fare questo articolo! Infatti l'idea è quella vedere se c'è una qualche relazione tra la statistica il mondo del calcio. Partendo da idee piuttosto semplici e da un'analisi sulla media dei gol segnati dalle squadra in caaa e in trasferta, ottenuta tramitei dati del campionato 2017/2018, procederemo con l'introdurre concetti di statistica cercando di renderli il più ssemplice possibile.
Costruendo, quindi, il modello di regressione e <i>dando in pasto</i> i dati che abbiamo estratto da internet, simuleremo le partite e dal risultato costruirime la classifica. A questo punto saremo in grado di stabilire se il modello è efficace confrontando la posizione delle squadre nella classifica calcolata con quella reale. 
Le tecnologie usate saranno per il data mining Pandas, Scipy, Statsmodel, mentre per la visualizzazione grafica Amchart e Chart.js. Sebbene anche Python abbia ottime librerie grafiche, l'idea di poter creare grafici interattivi mi ha stimolato a imparare e a rendere più interessanti i dati rappresentati.


A questo punto non posso che augurarvi una buona lettura....si comincia!
## Approccio iniziale
Come primo approccio bisogna trovare i dati su cui poi dobbiamo andare a lavorare. Effettuando una ricerca sul sito della Serie A non ho trovato un database ufficiale, quindi ho dovuto ripiegare su siti esterni. Ve ne sono diversi a disposizione, ma sicuramente quelli con uno storico e una disponibilità maggiore sono [www.football-data.co.uk](http://www.football-data.co.uk/italym.php) e [http://www.footstats.co.uk/](http://www.footstats.co.uk/index.cfm?task=league_full).

Mentre nel primo sono disponibile le statistiche di tutte le partite giocate per i principali campionati (e leghe minori....), nel secondo sono forniti i valori riassuntivi della classifica (suddivisi per half e full time).Da sottolineare che per entrambi i siti, è possibile scaricare i file in formato csv.

Solo a titolo di esempio, nella tabella successiva sono riportati gli esempi pratici di qualche dato usato durante l'analisi:

{% include /prova/prova.html %}

Il primo step da seguire è analizzare i dati presi in considerazione e per fare ciò useremo una libreria specifica di Python, ovvero Pandas.

<div style="align: center; text-align:center;">
    <img src="https://imgs.xkcd.com/comics/python.png"  class="center">
	<div class="caption"><small>Immagine presa da <a href="https://imgs.xkcd.com/comics/python.png">xkcd</a></small></div>
</div>


Iniziamo con l'importare Pandas e aprire il  file csv tramite la funzione apposita <i>open_csv</i>. In seguito calcoleremo la media dei gol segnati in casa e fuori casa specificando la colonna e utilizzando <i>.mean()</i>. 
```
import pandas #Importiamo Pandas
Partite_1718 = pd.read_excel('path\Partite_1718.xlsx') #Apriamo i file

mean_home = Partite_1718['FTHG'].mean()	#Calcoliamo la media
mean_away = Partite_1718['FTAG'].mean()	#

print("Media Gol in casa \t",mean_home)	#Stampiamo i valori
print("Media Gol fuori casa \t",mean_away)	#

```

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

Possiamo subito notare una caratteristica di questi dati! La media dei goal segnati in casa durante il campionato italiano 2017/2018 è maggiore di quelli fuori casa. Intuitivamente questo può avere senso, in quanto chi gioca in casa ha numerosi vantaggi rispetto alla squadra ospite, come il supporto dei tifosi, il non dover affrontare il viaggio, etc....

Come possiamo stabilire (più o meno) oggettivamente se questi due valori sono effettivamente differenti? Grazie a dei test statistici! 
Iniziamo facendo un'ipotesi, che chiameremo <i>ipotesi nulla</i>, tale per cui 


La media gol in casa è uguale alla media dei gol segnati fuori casa.
{: .notice--success .text-center}

In contrapposizione a quanto appena detto, ipotizziamo che

Le due medie sono diverse
{: .notice--success .text-center}

Chiamiamo quest'ultima ipotesi come <i>ipotesi alternativa</i>.

Per stabilire quale delle due è corretta dobbiamo rivolgerci al signor William Sealy Gosset, noto anche come Student.

<div class="dida_right">
  <h1 align="center">Info box</h1>
W. S. Gosset è stato un chimico e matematico agli inizi del 900. Amico di Pearson, lavorò alla Guiness ed è lì che sviluppò il t-test.
Poiché prima di lui un dipendente della fabbrica pubblicò il segreto della produzione della birra, per prevenire la fuga di informazioni la società impedì ai suoi dipendenti la pubblicazione di articoli contenente informazioni sensibili.
Per ovviare a questo problema Gosset pubblicò i suoi articoli con lo pseudonimo di Student. Ecco perché il test prende il nome di test di student!
	
	
</div>
Il test di student, noto anche come t-test, aiuta a capire se la differenza tra due medie è significativa o è dovuta al caso. Per fare ciò, va calcolato il valore <i>t</i> e confrontato con i valori riportati nelle [tavole](http://stat.unicas.it/vistoccoDownload/stat/materiale/tStudent.pdf).

Per fare ciò, importiamo <i>scipy</i>  eusiamo la funzione ttest_ind_from_stats del modulo <i>stats</i>. Useremo i valori medi e le variaziane dei goal segnati in casa e fuori casa calcolati precedentemente. Calcoliamo i gradi di libertà ([qui](https://it.wikipedia.org/wiki/Gradi_di_libert%C3%A0_(statistica) una breve speigazione) indicati dalla variabile <i>dof</i> e applichiamo il test.
```
from scipy.stats import ttest_ind_from_stats

dof = len(Partite_1718['FTHG'])+len(Partite_1718['FTAG'])-2	#Calcoliamo i gradi di libertà
t_stud, p_val = ttest_ind_from_stats(mean_home,var_home,dof,mean_away,var_away,dof)
print("===mean value Home/mean value Away===") #Stampiamo i valori
print(mean_home,"/t", mean_away)
print("===var value Home/var value Away===")
print(var_home,"/t", var_away)
print("===T-Student/P-Value===")
print(t_stud,"/t", p_val)
```
I valori ottenuti sono quindi:

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

In sostanza quello che stiamo dicendo è che i due valori medi sono <b>differenti</b>.

Come detto le motivazione alla base potrebbero essere molteplici e, in principio, dovremmo tenerne conto per sviluppare il nostro modello, ma per ora passeremo oltre e (forse) ci torneremo più in là.

Arrivati a questo punto iniziamo a complicare le cose introducendo la distribuzione di Poisson.

<div style="align: center; text-align:center;">
    <img src="https://imgs.xkcd.com/comics/poisson.jpg"  class="center">
	<div class="caption"><small>Toh!Un'altra immagine presa da <a href="https://xkcd.com/12/">xkcd</a></small></div>
	<div class="caption"><small>(Una breve spiegazione della vignetta <a href="http://leganerd.com/2011/04/15/xkcd-poisson/">qui</a>)</small></div>
</div>

La distribuzione di poisson viene utilizzata quando si vuole conoscere la probabilità di ottenere un certo numero di successi per unità di tempo, sapendo che <i>mediamente</i> se ne verifichi un numero $$\lambda$$. Questo tradotto in formule diventa:

$$
P\left( x \right) = \frac{e^{-\lambda} \lambda ^x }{x!}, \lambda>0
$$

Aspettate, c'è scritto mediamente? Ma allora è proprio quello che fa al caso nostro! Il problema è che ci sono alcune condizioni da rispettare (altrimenti il signor Siméon-Denis Poisson ce accappotta co na manata):

- <i>gli eventi devono essere indipendenti</i>;

- <i>la probabilità che in una determinata unità di tempo si verifichi un esito favorevole deve essere la stessa per tutte le unità</i>;

- <i>il numero medio di successi per unità di tempo deve essere costante</i>;

Queste condizioni ci stanno dicendo che:
- se considerassimo la partita come un insieme di istanti temporali composti da un minuto, per un totale di 90 minuti, dovremmo considerare che se un goal venisse segnato dalla squadra A in un istante qualunque della partita, la squadra B dovrebbe comportarsi coem se nulla fosse accaduto;
- la media goal dovrebbe mantenersi costante durante la partita (questo è valido se l'allenatore è Zeman!)

<div style="align: center; text-align:center;">
    <img src="https://i.kym-cdn.com/photos/images/original/000/471/439/321.gif"  class="center">
</div>

Diciamo che queste restrizioni non ci piacciono per niente! Però possiamo trovare una soluzione ragionandoci su...

Un'idea potrebbe essere quella di considerare una finestra temporale maggiore di un minuto: ad esempio possiamo supporre che tutta la partita sia composta da un'unità di tempo, ovvero che invece di considerare 90 minuti di istanti temporali, ne consideriamo uno da 90 minuti. Naturalmente la scelta di tale quantità minuti ricade nel fatto che le partite di calcio durano almeno novanta minuti (di più se consideriamo il recupero). Ciò ci permette di accettare le condizioni imposte e di poter procedere avanti con l'analisi!


<div style="align: center; text-align:center;">
    <img src="http://www.nov-art.eu/img/92MinutiDiApplausi.gif"  class="center">
	<div class="caption"><small>Qui ci vogliono 92 minuti di applausi! (meglio 90...)</small></div>
</div>

A questo punto siamo in grado di calcolare i valori della distribuzione di Poisson relativi ai due valori medi: per fare ciò, importiamo da <i>scipy.stats</i> la funzione <i>poisson</i>, quindi confrontiamo i valori teorici con quelli osservati tramite una rappresentazione grafica:

```
from scipy.stats import poisson		#importiamo la funzione poisson
frequency_home = Partite_1718['FTHG'].value_counts() #calcoliamo la frequenza dei goal e li rappresentiamo in un df
frequency_away = Partite_1718['FTAG'].value_counts()
df_home_FT = pd.DataFrame({'Home': frequency_home})
df_away_FT = pd.DataFrame({'Away':frequency_away})


df_FT = df_home_FT.combine_first(df_away_FT).fillna(0)
df_FT = df_FT[['Home','Away']]
for mean_to_use,state in zip([mean_home,mean_away],["Home","Away"]):
    df_FT = pd.DataFrame({state: Partite_1718['FTHG'].value_counts()})	#calcoliamo la frequenza dei goal e li rappresentiamo in un df
    obs = df_FT[state].values	#poniamo in un lista le frequenze osservate
    sum_tot = np.sum(obs)	#calcoliamo la somma totale dei goal
    exp = [poisson.pmf(i, mean_to_use)*sum_tot for i in range(0,len(obs))]	#calcoliamo i valori teorici

```


{% include /Articolo1/home_obs_teo_poisson.html %}


Effettivamente l'andamento tra i valori calcolati e quelli misurati sembrano essere simili...ma questo non basta, poiché per essere sicuri dobbiamo affidarci ancora una volta ad un test. 

Iniziamo con l'affermare, quindi, che

La distribuzione dei goal totali seguono approssimativamente una distribuzione di Poisson
{: .notice--success .text-center}

Per poter dire se questa affermazione sia vero o meno, dobbiamo ricorrere al test del chi quadro. 

<div class="dida_right">
  <h1>Info box</h1>
Il test del chi quadro è un test necessario nel caso si voglia verificare che le frequenze dei valori osservati abbiano un andamento simile alle frequenze teoriche di una distribuzione di probabilità scelta.
La formula per ottenere il valore &chi;<SUP>2</SUP> è
$$
\chi^{2}=\sum _{i=1}^{k}\frac{(o_{i}-e_{i})^{2}}{e_{i}}  
$$
dove o<SUB>i</SUB> e e<SUB>i</SUB> sono, rispettivamente, le frequenze osservate e teoriche. Questo test fu ideato da Karl Pearson, padre del Pearson citato prima!
	
	
</div>

Partendo dalle frequenze delle osservazioni misurate (cioè quante volte un certo numero di gol sono stati effettuati nelle partite), 
possiamo calcolare le frequenze teoriche e, confrontale, vedere quante queste differiscono tra loro. Il valore $$\chi ^{2}$$ quantifica questa informazione e, tramite le tavole [tavole della distribuzione](http://www00.unibg.it/dati/corsi/40025/74822-tavola_chi2.pdf)
è possibile determinare se l'ipotesi nulla sia verificata o meno.

Passando dalle parole ai fatti, vediamo lo script python usato per ottenere i risultati:
```
from scipy.stats import chisquare	#importiamo la funzione chisquare

for mean_to_use,state,name_col in zip([mean_home,mean_away],["Home","Away"],['FTHG','FTAG']):	#Questa parte è simile a quella di prima
    obs = df_FT[state].values									#ma l'ho inserita solo per una più
    sum_tot = np.sum(obs)									#chiara visualizzazione
    exp = [poisson.pmf(i, mean_to_use)*sum_tot for i in range(0,len(obs))]			#
    chi2_stat, p_val = chisquare(np.array(obs), np.array(exp))					#Questa è la parte nuova legata al
    print("==="+state+"===")    								#del valore chi e p (poi stampiamo i
    print("===Chi2 Stat===")									#valori ottenuti
    print(chi2_stat)
    print("===P-Value===")
    print(p_val)
	
```

I valori ottenuti sono

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
	

Questo ci dice che, per esempio, nel caso dei goal in casa i valori di $$\chi ^2$$ e p-value($$(P(>\chi^2)$$) sono rispettivametne 8.53308 e 0.28794), il ché significa che la probabilità del valore di chi quadro nelle [tavole della distribuzione del chi quadrato](http://www00.unibg.it/dati/corsi/40025/74822-tavola_chi2.pdf) di essere maggiore è del ≈ 29%. Perciò è possibile dire che c'è il 29% di possibilità che le due distribuzioni di valori siano uguali e che quindi l'ipotesi nulla può essere accettata!

<div style="align: center; text-align:center;">
    <img src="https://i.gifer.com/Fh5.gif"  class="center">
	<div class="caption"><small>Felicità!</small></div>
</div>

## Costruzione del modello
Procediamo e cerchiamo di costruire il modello. Quello che abbiamo è che che la distribuzione delle frequenze dei gol segue una distribuzione di Poisson.

Ma come ci può aiutare a predirre la futura vincitrice della serie A?

Possiamo calcolare la probabilità di vittoria di una squadra rispetto ad un'altra, costruendo un modello generalizzato di regressione lineare Poissoniano. 

Fermiamoci un attimo e respiriamo.

Il modello di regressione lineare è un modo generale per cercare di trovare funzione che rappresenti tutti punti in un grafico. Se i punti non sono distribuiti normalmente (cioè non seguono una distribuzione di Gauss), allora dobbiamo ricorrere al modello <b>generalizzato</b> di regressione lineare e, poiché nel nostro caso i punti seguono una distribuzione di Poisson, allora ecco che useremo il glm poissoniano!

Importiamo la libreria <i>statsmodels</i> e costruiamo il modello tramite la funzione glm.
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

Quello che otteniamo è

{% include /Articolo1/Tabella_modello_1.html %}
{% include /Articolo1/Tabella_modello_2.html %}

Dalla prima tabella possiamo estrarre qualche informazione: ad esempio sappiamo che la variabile dipendente sono i goal, che il numero di dati disponibili sono 760 e che la funzione usata, ovvero la fuzione link, è il logaritmo.

Nella seconda, invece, abbiamo diverse colonne che rappresentano:

- il coefficiente e l'errore associato
- il valore z, pari al rapporto tra il coefficiente e l'errore associato, e il relativo p-Value
- l'intervallo di confidenza

Di tutti questi valori quello che sicuramente ci interessa maggiormente è il p-value, poiché ci permette di capire quale variabile influisce maggiormente: infatti se fissiamo il livello di signicatifità del p-value al 5%, allora notiamo che sicuramente una variabile importante è la media gol della Juventus in casa (pari al 2%) o, anche, quello della Sampdoria fuori casa (pari al 3.6%).
I numeri presenti nella colonna <b>Coef</b> rappresentano i valori della funzione logaritmo (per un maggiore dettaglio vedere [qui](http://www.dima.unige.it/~rogantin/ModStat/Epidemiologia/teoria.pdf)): quelli positivi corispondenti alla riga con <i>Team[Squadra]</i> indicano le squadre con una maggiore predisposizione al gol, mentre quelli corrispondenti alla riga <i>Opponent[squadra]</i> indicano quelle squadre con una maggiore capacità di resistere al gol. Alla fine della colonna in questione troviamo il valore <i>home</i> il quale indica in generale la maggiore predisposizione delle squadre che giocano in casa a segnare maggiormente rispetto a quelle in trasferta.

Passiamo a un caso pratico, che tutte questi numeri e parole confondono.

<div class="tenor-gif-embed center" data-postid="5381579" data-share-method="host" data-width="50%" data-aspect-ratio="1.375"><a href="https://tenor.com/view/doc-brown-back-to-the-future-when-you-remember-when-you-cant-remember-if-christopher-lloyd-gif-5381579">Doc Brown Back To The Future GIF</a> from <a href="https://tenor.com/search/docbrown-gifs">Docbrown GIFs</a></div><script type="text/javascript" async src="https://tenor.com/embed.js"></script>

Prendiamo come esempio la Juventus: essa presenta come valore team 0.3966, mentre come valore opponent -0.4564. Possiamo fare un confronto con la Lazio la quale presenta una capacità di fare goal maggiore rispetto alla Vecchia Signora (infatti presenta un valore home di 0.4578), ma ha una capacità di difendersi nettamente peggiore (valore opponent 0.2630). Possiamo, inoltre, notare che le squadre con una migliore difesa sono quelle che risultano essere entrate in Champion's League l'anno scorso (quindi è vero il detto che "<i>In Serie A vince chi non subisce goal</i>"!)

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

Cerchiamo quindi di arrivare finalmente a quanto promesso da inizio articolo: la simulazione del campionato di Serie A.

Il primo step è simulare l'esito di una partita tra due squadre, partendo da quanto ottenuto fino ad ora.

Prendiamo la Juventus e il Napoli, rispettivamente, prima e seconda classificata del passato campionato italiano di Serie A.

Supponiamo che, per semplicità, i possibili risultati varino in un intervello che va dal 3 a 0 in casa allo 0 a 3 per la squadra in trasferta (ovvero 3-0,3-1,3-2,3-3,0-3,1-3,2-3).

Allora quello che avremo è una matrice di probabilità, un nome complicato per indicare una cosa molto semplice: 
lungo le righe abbiamo la probabilità che il Napoli possa segnare quel numero di gol entro la fine della partita, mentre, viceversa, sulle colonne abbiamo le probabilità che la Juventus possa segnare quel goal.

Incrociando la riga con la colonna abbiamo la probabilità che una partita finisca con un certo risultato.

Simulando abbiamo:
<table class="table table-bordered table-hover table-condensed">

<tbody>
<tr>
<td >Napoli\Juventus</td>
<td>0</td>
<td >1</td>
<td >2</td>
<td >3</td>
</tr>

<tr>
<td >0</td>
<td >0.10652766</td>
<td >0.12585058</td>
<td >0.07433923</td>
<td >0.02927451</td>
</tr>
<tr>
<td >1</td>
<td >0.11270221</td>
<td >0.13314511</td>
<td >0.07864807</td>
<td >0.03097131</td>
</tr>
<tr>
<td >2</td>
<td >0.05961732</td>
<td >0.07043123</td>
<td >0.04160333</td>
<td >0.01638323</td>
</tr>
<tr>
<td >3</td>
<td >0.02102428</td>
<td >0.02483785</td>
<td >0.01467158</td>
<td >0.00577761</td>
</tr>
</tbody></table>

Analizzando i valori, abbiamo che il risultato più probabile per Napoli Juventus è l'1 a 1 e andando a vedere il risultato della partita reale, questo è proprio il pareggio con un gol per parte ([qui](https://www.google.it/search?ei=dQPXW-ntCo2dgQaOoaHIDA&q=napoli+juventus+1+a+1&oq=napoli+juventus+1+a+1&gs_l=psy-ab.3..0i22i30k1l2.7854.9183.0.9345.6.6.0.0.0.0.116.585.5j1.6.0....0...1c.1.64.psy-ab..0.6.582...0j0i67k1.0.W5R_x9VFQpI#sie=m;/g/11ggr8ytzv;2;/m/03zv9;dt;fp;1!) i risultati della partita)!

Quindi il modello sulla singola partita sembra essere coerente!

Sommando i risultati lungo la diagonale abbiamo la probabilità totale che il risultato finale sia un pareggio, mentre sommando la parte superiore (o inferiore) della matrice abbiamo la probabilità che la squadra fuori casa (in casa)
vinca. In questo caso la probabilità che la partita finisca in pareggio è 0.28705, mentre che vinca la Juventus è 0.35547 o il Napoli è 0.30328. In sostanza quindi è più probabile che la <i>Vecchia Signora</i> vinca la partita.

Ed è qui che finalmente arriviamo alla parte finale del nostro viaggio....vediamo se riusciamo a predire la vincitrice del campionato!

L'idea è quella di far scontrare tutte le squadre in maniera casuale e simulare gli incontri con il modello ottenuto. 

```
dict_classifica = dict()
for squadra in Partite_1718['HomeTeam'].unique(): #Creiamo il dizionario dove le squadre sono le chiavi
    dict_classifica[squadra] = 0				  
for squadra_home,squadra_away in zip(Partite_1718['HomeTeam'],Partite_1718['AwayTeam']): 
    simulation = simulate_match(poisson_model, squadra_home,squadra_away, max_goals=15) #Effettuiamo la simulazione tramite
    prob_home = np.sum(np.tril(simulation, -1))						#il modello costruito e calcoliamo la 
    prob_par = np.sum(np.diag(simulation))						#probabilità che la squadra che giochi in casa 
    prob_away = np.sum(np.triu(simulation, 1))						#vinca sommando gli elementi del triangolo 
    result = a.index(max([prob_home,prob_par,prob_away]))				#superiore, la probabilità del pareggio sommando 											 #gli elementi sulla diagonale e, infine, la
    											#probabilità che la squadra che gioca fuori casa
											#vinca sommando gli elementi del triangolo
											#inferiore. 

    if result == 0:		#Riassumo i risultati e a seconda del risultato, diamo il punteggio per la costruzione della classifica.
        dict_classifica[squadra_home] = dict_classifica[squadra_home]+3
    if result == 1:
        dict_classifica[squadra_home] = dict_classifica[squadra_home]+1
        dict_classifica[squadra_away] = dict_classifica[squadra_away]+1
    if result == 2:
        dict_classifica[squadra_away] = dict_classifica[squadra_away]+3
```
La classifica finale è
{% include /Articolo1/classifica_teorica.html %}

Arrivati a questo punto facciamo un confronto tra la classifica <i>reale</i> e quella <i>teorica</i>: notiamo che le squadre dal primo al sesto posto risultano avere la stessa posizione ma un numero diversi di punti.
Addirittura per la Juventus si ha la vittoria in tutti i match (quindi per un totale di 114 punti!). Il motivo di questa differenza è data dal fatto che non sono stati presi in considerazione diversi fattori, come gli infortuni, la stanchezza o eventuali psicologici.
Di fatto si è ipotizzato che ogni squadra schierasse la formazione titolare, giocasse solo il campionato nazionale  e ogni giocatore fosse sempre in forma.

Procedendo verso la zona dell'Europa League notiamo una leggera imprecisione nella predizione. Qualche squadra risulta essere sotto o sopra rispetto alla posizione reale. La squadra con maggiore differenza in questi termini di classifica è il Sassuolo,
il quale secondo il modello sarebbe fovuto retrocedere, mentre nella realtà è arrivato sesto. Sempre perché a noi i piacciono i grafici, ecco qua rappresentazione di quanto detto:

{% include /Articolo1/graf_bubble.html %}

Come si vede abbiamo sia dei pallini grigi monocromatici posti lungo una retta, i quali rappresentano le posizioni ideali nella classifica, e quelli colorati, che rappresentano i valori incrociati della posizione di classifica reale e calcolata delle diverse squadre. 

Se il pallino colorato si sovrappone a quello grigio, allora le previsione teorica coincide con quella reale, altrimenti se è collocato sopra sopra la retta significa che la squadra è stata sottostimata dal punto di vista teorico e, viceversa, nel caso opposto. Ad esempio possiamo cosniderare il Sassuolo, il quale graficamente, come si vede, è la squadra più distante dalla retta in quanto l'anno scorso è arrivato undicesimo, mentre dal punto di vista teorico è stato ipotizzato diciottesimo. 

Conludiamo la descrizione del grafico dicendo che i pallini colorati differenziano anche nella grandezza: più il pallino è piccolo più la differenza di punti tra la classifica reale e teorica è minore mentre più è grande il raggio, maggiore è la distanza. Poiché abbiamo anche dei valori negativi, allora abbiamo riparametrizzato i valori tra 1, il quale rappresenta il cerchio con raggio minore (vedi Sassuolo), e 10, il quale, invece, rappresenta il cerchio con raggio maggiore (vedi Inter).
## Conclusione
Da dati del campionato 2017/2018 abbiamo effettuato un'analisi che ci ha permesso di estrarre informazioni statistiche sulle singole squadre. Abbiamo notato che la media dei goal in casa è differente rispetto a quella fuori casa e grazie a dei test statistici, abbiamo accertato questa ipotesi. La statistica, inoltre, ci ha anche dato una mano a capire se possiamo modellizare con sicurezza la frequenza dei goal nel campionato con una distribuzione di Poisson. Poiché anche questa ipotesi si è dimostrata corretta, abbiamo costruito un <i>simulatore</i> di partite da cui ricavare una classifica teorica: in questa stagione simulata il campionato è stato vintodalla Juventus, con il Napoli secondo e la Roma e l'Inter terze a pari punti. Andando a confrontare i posizionamenti con quelli reali, notiamo che il nostro modello risulta essere abbastanza preciso. Naturalmente la predizione per alcune squadre non è sufficientemente buona, ma possiamo accontentarci per questo livello di elaborazione.

Vorrei citare le seguenti fonti, fondamentali per l'estensione di questo primo articolo:
- https://dashee87.github.io/ :Ottimo, ottimo, ottimo blog sul data science. Ironico e divertente permette di comprendere bene quello che si sta leggendo.
- https://www1.maths.leeds.ac.uk/~voss/projects/2010-sports/JamesGardner.pdf : articolo da cui ho avuto l'ispirazione per questa idea.

Concludo ringraziandovi per essere arrivati fin qui e spero che possiamo ritrovarci nuovamente in qualche altro articolo!

<iframe src="https://giphy.com/embed/26BoCW1FA2980EaR2" width="480" height="205" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/26BoCW1FA2980EaR2">via GIPHY</a></p>


