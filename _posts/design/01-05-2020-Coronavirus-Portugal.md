---
layout: page
subheadline:  "Headers With Style"
title:  "No Header but Article Image"
teaser: "Feeling Responsive enables you to get the attention of visitors. If you don't want to use a big header, use an image for the article instead."
categories:
    - design
tags:
    - design
    - background color
    - header
header: no
image:
    title: sars_cov_2.jpg
    caption: This is a caption for the header image with link
    caption_url: https://unsplash.com/
---


```{r setup, include=FALSE}
version <- "1.8"
version_date <- lubridate::ymd("2020-02-28")

knitr::opts_chunk$set(echo = FALSE, cache=TRUE,
                      tidy.opts=list(width.cutoff=60),
                      tidy=TRUE)
library(tidyverse)
library(magrittr)
library(lubridate)
library(tibble)
library(ggplot2)
library(ggthemes)
library(hrbrthemes)
library(rvest)
library(gt)
library(deSolve)
library(EpiEstim)
library(incidence)
library(distcrete)
library(epitrix)
library(projections)
library(formatR)

```

## Introdu��o

Este post foi feito para fins didacticos e pretende ajudar a entender melhor como � poss�vel construir um modelo matem�tico para estudar a evolu��o da doen�a Covid19, provocada pelo v�rus SARS-CoV2, e aplicar esse modelo ao caso portugu�s. Surgiu na sequ�ncia de alguma pesquisa e da leitura do seguinte post https://blog.ephorie.de/epidemiology-how-contagious-is-novel-coronavirus-2019-ncov?fbclid=IwAR35_eyO1Ry6Bru04WKKPNv7mxt5rhNT_liU6QlEqJ8u-BrOZoVHxxJ0 . 
Antes de mais nada, a minha forma��o acad�mica � em Qu�mica e Biotecnologia, por isso tenho algumas bases cient�ficas, contudo n�o sou especialista em epidemiologia. Talvez devido a esse facto, resolvi aprender a trabalhar com o software R e aprofundar os meus conhecimentos sobre esta doen�a. Este trabalho n�o se destina a fazer nenhuma previs�o, nem ser uma ferramenta de tomada de decis�es, sendo apenas uma primeira abordagem (imperfeita) de aproxima��o � realidade, destinando-se apenas a ilustrar alguns conceitos cient�ficos de estat�stica, epidemiologia e modela��o.
Para a metodologia escolhi usar o software R para os c�lculos estat�sticos e fazer modela��o dos dados.
Portanto, a primeira pergunta �: 

## "Como � poss�vel aos epidemiologistas estimarem o grau de cont�gio do v�rus e como a epidemia evolui?"
Para responder a esta quest�o a abordagem cl�ssica � atrav�s de modelos que tentam simular a realidade e assim prever a evolu��o de uma determinada epidemia, ajudando dessa forma na tomada de decis�es informadas sobre estrat�gias de sa�de p�blica, ou outras, de combate � sua propaga��o (medidas de quarentena, isolamento social, higieniza��o, vacina��o, etc).

Existem diversos modelos dispon�veis, por�m aqui vamos usar o modelo SIR, talvez um dos mais populares (poder� consultar mais informa��o sobre diversos modelos existentes nesta p�gina https://en.wikipedia.org/wiki/Compartmental_models_in_epidemiology). 
Os dados estat�sticos relativos ao n�mero de pessoas infectadas, que usaremos prov�m das autoridades oficiais portuguesas, a Direc��o Geral de Sa�de (DGS), e podem ser consultados aqui (https://covid19.min-saude.pt/relatorio-de-situacao/). As an�lises adicionais ser�o minhas.

## "Por que � t�o importante criarmos modelos?" 

A sa�de e a economia s�o duas faces de qualquer epidemia. Por um lado, precisamos salvar vidas e, por outro, precisamos manter nossos empregos, servi�os sociais e todas as atividades que precisamos para sobreviver (agricultura, ind�stria, servi�os e outras). Precisamos de ter estrat�gias para alcan�armos um equil�brio entre a sa�de e economia. 
Na minha opini�o, numa primeira fase deveremos fazer um esfor�o para salvar vidas. Portanto, as medidas de isolamento social devem ser t�o restritivas quanto as necess�rias para evitar o colapso do sistema de sa�de, mas tamb�m n�o as poderemos prolongar indefinidamente no tempo ou provocaremos uma crise econ�mica com consequ�ncias tamb�m graves para a sociedade.
Numa segunda fase, deveremos retomar as actividades econ�micas numa situa��o de maior controlo das vari�veis que influenciam a transmiss�o do v�rus (nomeadamente o R0). Uma vez que poderemos ter novamente um aumento de casos quando as medidas de quarentena s�o levantadas. 
Quanto � imunidade de grupo, estudos estimam que esta se atinge quando, pelo menos, cerca de 60% da popula��o fica imunizada contra um agente infeccioso (mas ainda decorrem investiga��es para elucidar esta quest�o quanto ao caso da Covid19). Enquanto n�o existir uma vacina ou adquirirmos a imunidade de grupo necessitamos de continuar vigilantes quanto � progress�o da doen�a, da� a necessidade destes modelos, para tomarmos decis�es informadas. Surge ent�o a quest�o seguinte: 

## "Em que fase da epidemia estamos em Portugal agora? "

O primeiro passo ser� analisarmos os casos confirmados acumulados no tempo e para tal precisamos encontrar uma fonte confi�vel dispon�vel. Como os dados da DGS est�o em arquivos PDF no seu website, precisamos de uma solu��o melhor para adquirir rapidamente os dados de maneira autom�tica. A solu��o que podemos usar � recorrer a uma tabela que contenha os mesmos dados mas que est� em c�digo html, pelo que recorri � seguinte fonte: https://pt.wikipedia.org/wiki/Pandemia_de_COVID-19_em_Portugal#Evolu%C3%A7%C3%A3o_dos_casos ).
Com algum c�digo em R podemos selecionar os dados de interesse (c�digo � fornecido em anexo).  Outra solu��o, ser� adquirimos os dados em um arquivo (formato csv ou txt) ou inserirmos os mesmos manualmente no c�digo, mas n�o vou desenvolver este tema mais aqui pois existem diversos tutoriais sobre o software R para quem quiser aprofundar estes conhecimentos. Ter em aten��o que caso existam altera��es nas p�ginas de onde retiramos on-line a informa��o poderemos ter de reajustar o c�digo.
Ent�o visualizando os dados graficamente: 
```{r Scraping HTML Tables, include=FALSE}

# download the wikipedia web page

# we use a specific version of the template page directly
# version of the wikipedia page that is used by this version of this document
portuguese_wikipedia_data_url <- "https://pt.wikipedia.org/wiki/Pandemia_de_COVID-19_em_Portugal#Evolu%C3%A7%C3%A3o_dos_casos"

# unversioned page
# portuguese_wikipedia_data_url <- "https://pt.wikipedia.org/wiki/Pandemia_de_COVID-19_em_Portugal#Evolu%C3%A7%C3%A3o_dos_casos"
portuguese_outbreak_webpage <- read_html(portuguese_wikipedia_data_url)

# read tables
tbls <- html_nodes(portuguese_outbreak_webpage, "table")

head(tbls)

tbls_ls <- portuguese_outbreak_webpage %>%
  html_nodes("table") %>%
  .[5:6] %>%
  html_table(fill = TRUE)

# verification of the parameters of the table
str(tbls_ls)
head (tbls_ls)


# remove row 1 that includes part of the headings
tbls_ls[[1]] <- tbls_ls[[1]][-2]

# remove row 1 that includes part of the headings

tbls_ls[[1]]<- tbls_ls[[2]][-1,]
tbls_ls[[1]]

#last_date_j <- ymd("2020-03-01")

# download the wikipedia web page

# we use a specific version of the template page directly
# version of the wikipedia page that is used by this version of this document
portuguese_wikipedia_data_url <- "https://pt.wikipedia.org/wiki/Pandemia_de_COVID-19_em_Portugal#Evolu%C3%A7%C3%A3o_dos_casos"

# unversioned page
# portuguese_wikipedia_data_url <- "https://pt.wikipedia.org/wiki/Pandemia_de_COVID-19_em_Portugal#Evolu%C3%A7%C3%A3o_dos_casos"
portuguese_outbreak_webpage <- read_html(portuguese_wikipedia_data_url)

# read tables
tbls <- html_nodes(portuguese_outbreak_webpage, "table")

#head(tbls)

tbls_ls <- portuguese_outbreak_webpage %>%
  html_nodes("table") %>%
  .[5:6] %>%
  html_table(fill = TRUE)

# verification of the parameters of the table
#str(tbls_ls)
#head (tbls_ls)

# remove row 1 that includes part of the headings
tbls_ls[[1]] <- tbls_ls[[1]][-2]

# remove row 1 that includes part of the headings

tbls_ls[[1]]<- tbls_ls[[2]][-1,]
tbls_ls[[1]]

# rename table headings
colnames(tbls_ls[[1]]) <- c("Date", "Confirmed Cases")

#atribute values
Infected<-tbls_ls[[1]]$`Confirmed Cases`

#convert date in Char to date format

Date <-tbls_ls[[1]]$`Date`

Date<-seq(as.Date('2020-03-03'), as.Date('2020-12-31'), by = 'days') 

#Date <- as_tibble(Date)
```

```{r infected}
#Inseri os valores de Portugal at� dia 17-04-2020:

Day <- 1:(length(Infected))
N <- 10000000 # popula��o de Portugal
 
old <- par(mfrow = c(1, 2))
plot(Day, Infected, type ="b")
plot(Day, Infected, log = "y")
#abline(lm(log10(Infected) ~ Dia))
title("Cumulative confirmed cases 2020-Portugal", outer = TRUE, line = -2)
```


O gr�fico � esquerda � o total acumulado de pessoas infectadas ao longo do tempo (em dias desde o in�cio do primeiro caso detectado) e � direita o mesmo gr�fico, mas com uma escala logar�tmica no eixo y (um gr�fico log-linear).
No segundo gr�fico parece bastante claro que a curva est� a "aplanar", mostrando que em Portugal no momento actual terminamos o crescimento exponencial, ou seja, a taxa de crescimento � muito mais lenta agora. Veremos mais detalhadamente esta quest�o adiante. 

# Modela��o dos dados
Chegamos ent�o � modela��o dos dados com o modelo SIR, cuja ideia b�sica � bastante simples. Existem tr�s grupos de pessoas: aqueles que s�o saud�veis, mas suscept�veis � doen�a (S), os infectados (I) e as pessoas que se recuperaram (R):
Para modelar a din�mica do surto, precisamos de tr�s equa��es diferenciais, uma para a mudan�a em cada grupo, onde \beta � o par�metro que controla a transi��o entre S e I e \gama que controla a transi��o entre I e R:

![Modelo SIR](https://www.lewuathe.com/assets/img/posts/2020-03-11-covid-19-dynamics-with-sir-model/sir.png "Modelo SIR")


Source: wikipedia

O modelo pode ser representado por:


  \[\frac{dS}{dt} = - \frac{\beta I S}{N}\]

  \[\frac{dI}{dt} = \frac{\beta I S}{N}- \gamma I\]

  \[\frac{dR}{dt} = \gamma I\]

Inserindo as equa��es no modelo temos:
```{r SIR MODEL, include=TRUE}
SIR <- function(time, state, parameters) {
  par <- as.list(c(state, parameters))
  with(par, {
    dS <- -beta/N * I * S
    dI <- beta/N * I * S - gamma * I
    dR <- gamma * I
    list(c(dS, dI, dR))
    })
}
```
Inserimos as equa��es anteriores no modelo e de seguida precisamos de duas fun��es: uma para resolver as equa��es e outra para optmizar. Para a primeira usaremos a fun��o "Ode" do pacote "deSolve" (CRAN) e para a optimiza��o usaremos a ferramenta de base do R, ou seja, um m�todo de minimiza��o da soma da diferen�a quadr�tica entre o n�mero de infectados e o n�mero de casos previstos pelo modelo, ao longo do tempo (t):

  \[RSS(\beta, \gamma) = \sum_{t} \left( I(t)-\^{I}(t) \right)^2\]

Resolvendo, obt�m-se converg�ncia (indicada pelo software) e os seguintes par�metros \beta e \gama. 


Ap�s uma pr�via an�lise explorat�ria dos dados escolhemos apenas os pontos que pertencem a uma fase de crescimento exponencial at� ao dia 20-03-2010.  
Escolhemos apenas estes dias porque sabemos da literatura que este modelo n�o integra as vari�veis de distanciamento social que tomamos em Portugal em 16-03-2020, com o fecho das escolas, e depois em 18-03-2020, foi decretado o "Estado de Emerg�ncia", al�m das medidas de higieniza��o.
Assim vamos estimar por excesso (obviamente) numa situa��o hipot�tica em que nada teria sido feito para evitar a progress�o dos cont�gios. 
Podemos ent�o fazer a representa��o gr�fica:


```{r optimization2, include=FALSE}

#select data until day 20-03-2020, exponecial growth rate


# download the wikipedia web page

# we use a specific version of the template page directly
# version of the wikipedia page that is used by this version of this document
portuguese_wikipedia_data_url <- "https://pt.wikipedia.org/wiki/Pandemia_de_COVID-19_em_Portugal#Evolu%C3%A7%C3%A3o_dos_casos"

# unversioned page
# portuguese_wikipedia_data_url <- "https://pt.wikipedia.org/wiki/Pandemia_de_COVID-19_em_Portugal#Evolu%C3%A7%C3%A3o_dos_casos"
portuguese_outbreak_webpage <- read_html(portuguese_wikipedia_data_url)

# read tables
tbls <- html_nodes(portuguese_outbreak_webpage, "table")

head(tbls)

tbls_ls <- portuguese_outbreak_webpage %>%
  html_nodes("table") %>%
  .[5:6] %>%
  html_table(fill = TRUE)

# verification of the parameters of the table
str(tbls_ls)
head (tbls_ls)
#at� aqui funciona!!

# remove row 1 that includes part of the headings
tbls_ls[[1]] <- tbls_ls[[1]][-2]

# remove row 1 that includes part of the headings

tbls_ls[[1]]<- tbls_ls[[2]][-1,]
tbls_ls[[1]]

# download the wikipedia web page

# we use a specific version of the template page directly
# version of the wikipedia page that is used by this version of this document
portuguese_wikipedia_data_url <- "https://pt.wikipedia.org/wiki/Pandemia_de_COVID-19_em_Portugal#Evolu%C3%A7%C3%A3o_dos_casos"

# unversioned page
# portuguese_wikipedia_data_url <- "https://pt.wikipedia.org/wiki/Pandemia_de_COVID-19_em_Portugal#Evolu%C3%A7%C3%A3o_dos_casos"
portuguese_outbreak_webpage <- read_html(portuguese_wikipedia_data_url)

# read tables
tbls <- html_nodes(portuguese_outbreak_webpage, "table")

#head(tbls)

tbls_ls <- portuguese_outbreak_webpage %>%
  html_nodes("table") %>%
  .[5:6] %>%
  html_table(fill = TRUE)

# verification of the parameters of the table
#str(tbls_ls)
#head (tbls_ls)

# remove row 1 that includes part of the headings
tbls_ls[[1]] <- tbls_ls[[1]][-2]

# remove row 1 that includes part of the headings

tbls_ls[[1]]<- tbls_ls[[2]][-1,]


# rename table headings
colnames(tbls_ls[[1]]) <- c("Date", "Confirmed Cases")

#atribute values
Infected<-tbls_ls[[1]]$`Confirmed Cases`

#convert date in Char to date format

Date <-tbls_ls[[1]]$`Date`

Date<-seq(as.Date('2020-03-03'), as.Date('2020-03-20'), by = 'days') 

Infected<-Infected[1:18]

#tbls_ls[[1]]$`Date`<- Date

#select data until day 20-03-2020, exponecial growth rate
data.frame<-data.frame(Date,Infected)

Infected<-data.frame[data.frame$Infected <=1220, ]

Date<-data.frame$Date

Infected<-data.frame$Infected

#atribute values without intervention or social distancing
Infected_exp<-Infected

#Inseri os valores de Portugal at� dia 20-03-2020:

Day <- 1:(length(Infected))
N <- 10000000 # popula��o de Portugal
 
old <- par(mfrow = c(1, 2))
plot(Day, Infected, type ="b")
plot(Day, Infected, log = "y")
#abline(lm(log10(Infected) ~ Dia))
title("Cumulative confirmed cases 2020-Portugal", outer = TRUE, line = -2)

SIR <- function(time, state, parameters) {
  par <- as.list(c(state, parameters))
  with(par, {
    dS <- -beta/N * I * S
    dI <- beta/N * I * S - gamma * I
    dR <- gamma * I
    list(c(dS, dI, dR))
    })
}



#library(deSolve)
init <- c(S = N-Infected_exp[1], I = Infected_exp[1], R = 0)
RSS <- function(parameters) {
  names(parameters) <- c("beta", "gamma")
  out <- ode(y = init, times = Day, func = SIR, parms = parameters)
  fit <- out[ , 3]
  sum((Infected_exp - fit)^2)
}
 
Opt <- optim(c(0.5, 0.5), RSS, method = "L-BFGS-B", lower = c(0, 0), upper = c(1, 1)) # optimize with some sensible conditions
Opt$message
#[1] "CONVERGENCE: REL_REDUCTION_OF_F <= FACTR*EPSMCH"
 
Opt_par <- setNames(Opt$par, c("beta", "gamma"))
Opt_par
#beta     gamma 
## 0.6746089 0.3253912
 
```
```{r pressure, echo=FALSE}
t <- 1:100 # time in days
fit <- data.frame(ode(y = init, times = t, func = SIR, parms = Opt_par))
col <- 1:3 # colour
 
matplot(fit$time, fit[ , 2:4], type = "l", xlab = "Day", ylab = "Number of subjects", lwd = 2, lty = 1, col = col)
matplot(fit$time, fit[ , 2:4], type = "l", xlab = "Day", ylab = "Number of subjects", lwd = 2, lty = 1, col = col, log = "y")
## Warning in xy.coords(x, y, xlabel, ylabel, log = log): 1 y value <= 0
## omitted from logarithmic plot
 
points(Day, Infected_exp)
legend("bottomright", c("Susceptibles", "Infecteds", "Recovereds"), lty = 1, lwd = 2, col = col, inset = 0.05)
title("SIR model COVID19 Portugal", outer = TRUE, line = -2)
```


Observando os gr�ficos tal como previsto temos um bom ajuste dos dados reais � curva prevista pelo modelo.
Assim, temos uma primeira aproxima��o � realidade e permite estimar alguns coeficientes. Num pr�ximo post vamos tentar usar um modelo mais realista. 
Devemos tamb�m ter em conta que uma "curva epid�mica" deve ser feita com as "datas de in�cio dos sintomas" (tal como podemos verificar nos boletins oficiais) e aqui estamos a usar os casos confirmados por n�o termos outros dados. Outro factor � que a qualidade dos dados � muito importante para a qualidade do modelo.

# Estimativa de R0
Agora poderemos extrair algumas estat�sticas importantes. Um dos coeficientes � o chamado n�mero b�sico de reprodu��o ou taxa b�sica de reprodu��o, R0 ("R nought" em ingl�s) que mostra basicamente quantas pessoas saud�veis s�o infectadas por uma pessoa doente em m�dia (n�mero m�dio de cont�gios):


```{r Estimation R0, include=TRUE}
par(old)
 
R0 <- setNames(Opt_par["beta"] / Opt_par["gamma"], "R0")
R0
##       R0 
## 1.98384
 
fit[fit$I == max(fit$I), "I", drop = FALSE] # height of pandemic
##            I
## 37 616443.4
 
max(fit$I) * 0.02 # max deaths with supposed 2% fatality rate
## [1] 12328.87

```



Assim, o R0 � estimado em 1,9 na fase inicial da epidemia no pa�s, o que � consistente com o n�mero que muitos pesquisadores e a OMS estimaram sendo aproximado do valor do SARS, Influenza ou �bola. Existem diversos valores estimados que v�o desde 1,4 at� 3,5 para este par�metro (https://www.worldometers.info/coronavirus/#repro). De notar ainda que este R0 vem diminuindo ao longo do tempo passando a designar-se Rt ou Re (R efectivo), senbdo neste momento inferior a 1 segundo a DGS (pode variar de regi�o para regi�o).
Al�m disso, de acordo com este modelo, o pico da epidemia seria alcan�ado em torno de 16-04-2020 (45 dias ap�s o in�cio). Como j� referido a curva epid�mica dever� ser constru�da com a data de in�cio de sintomas e n�o de casos laboratoriais confirmados, pelo que podemos considerar uma m�dia de 7 dias entre o in�cio de sintomas e detec��o laboratorial, logo se retirarmos 7 dias, ter� sido na primeira semana de Abril, tamb�m n�o muito longe de algumas informa��es oficiais.
Neste modelo hipot�tico a extens�o da epidemia seria cerca de 1,5 milh�es de pessoas infectadas e cerca de 30000 �bitos (assumindo taxa de mortalidade de 2%), o que claramente est� sobrestimado. 
Como discutido anteriormente, este n�mero est� estimado para o pior cen�rio poss�vel, sem quaisquer medidas, assumindo um modelo deterministico aleat�rio de transmiss�o. 
Isto pode dever-se ao modelo ser demasiado simplista por n�o incluir as vari�veis de distanciamento social e/ou de outras medidas tomadas (portanto, estes n�meros s�o muito altos). Outro factor que podemos ter � que existem muitos casos assintom�ticos e estes nunca foram testados. S� saberemos quantas pessoas estiveram realmente expostas ao v�rus atrav�s de testes serol�gicos � popula��o.
 Portanto, n�o entremos em p�nico, vamos tentar criar um modelo melhor para o caso portugu�s, num pr�ximo post.
 
# Bibliografia consultada

https://www.nytimes.com/2020/04/23/world/europe/coronavirus-R0-explainer.html
https://covid19.min-saude.pt/relatorio-de-situacao/
https://wikiciencias.casadasciencias.org/wiki/index.php/Modelo_SIR_em_epidemiologia
https://wwwnc.cdc.gov/eid/article/26/7/20-0282_article
https://www.worldometers.info/coronavirus/#repro

