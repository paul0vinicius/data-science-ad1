---
layout: post
title: "Qual a melhor série da atualidade?"
date: 2017-07-09 14:44:30
published: true
tags: [htmlwidgets, r]
author: "Paulo Vinícius Soares"
output: html_document
---

Primeiro, vamos importar as bibliotecas e configurar o RStudio.

{% highlight r %}
knitr::opts_chunk$set(echo = TRUE,warning = FALSE,message = FALSE,fig.width = 10, fig.align = "center")

library(readr)
library(dplyr)
{% endhighlight %}



{% highlight text %}
## 
## Attaching package: 'dplyr'
{% endhighlight %}



{% highlight text %}
## The following objects are masked from 'package:stats':
## 
##     filter, lag
{% endhighlight %}



{% highlight text %}
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
{% endhighlight %}



{% highlight r %}
library(ggplot2)
{% endhighlight %}

Agora, vamos importar a nossa base de dados!
Após importar a base de dados do IMDB, vou escolher 6 séries para avaliar.

As séries escolhidas são:

  * How To Get Away With Murder
  * 13 Reasons Why
  * Friends
  * Modern Family
  * Black Mirror
  * Grey's Anatomy
  

{% highlight r %}
series_from_imdb <- read_csv("https://raw.githubusercontent.com/nazareno/imdb-series/master/data/series_from_imdb.csv")
dataSeries<- series_from_imdb %>% filter(series_name %in% c("How to Get Away with Murder", "13 Reasons Why", "Friends","Modern Family", "Black Mirror", "Grey’s Anatomy"))
{% endhighlight %}

A nossa base de dados fornece inúmeras variáveis, incluindo nome e número do episódio, temporada, avaliação do usuário (nota), link, entre outras. Para responder às duas perguntas, utilizaremos os nomes das séries e a avaliação do usuário por episódio (UserRating), em suma.

**1. Qual das séries que eu escolhi é mais bem avaliada no IMDB? A diferença é grande? Pequena?**

Os dados estão agrupados da seguinte forma: Cada episódio da série tem uma nota, então podemos analisar a média ou mediana das notas dos episódios a fim de saber qual possui a maior nota e, dessa forma, descobrir qual a série mais bem avaliada. Nesse caso utilizaremos a mediana.    
Por que não utilizar a média? Bem, a média é afetada por notas extremas, que podem distorcer onde a maioria das notas se concentram, de forma que podem alterar a percepção final do leitor dos dados. Nós estamos mais interessados na **constância das notas**, de forma que uma nota ou outra que sejam pontos fora da curva podem ser desconsiderados.

Para termos uma análise mais completa dos nossos dados, vamos utilizar um gráfico de caixa, o boxplot. Vou te explicar o que podemos extrair a partir dele.

<img src="/data-science-ad1/figure/source/lab01/images/howtoreadboxplot.png" style="display: block; margin: auto;" />

A parte que daremos mais enfoque no nosso gráfico de caixa são os conceitos de **mediana**, **terceiro quartil** e **primeiro quartil**. O boxplot agrupa, dentro da caixinha, metade dos dados de forma ordenada. A mediana é o **valor do meio**, ou **valor central**, que separa os maiores valores dos menores, agrupando 50% da amostra acima e 50% da amostra abaixo de sua linha. O primeiro quartil agrupa 25% das notas abaixo e 75% das notas acima, enquanto o terceiro quartil agrupa 75% das observações abaixo e 25% acima.


Vamos ordenar os boxplots abaixo de forma decrescente, da série com maior mediana para menor mediana.

{% highlight r %}
dataSeriesBP <- ggplot(dataSeries, aes(x=reorder(series_name, -UserRating, FUN=median), y = UserRating)) + geom_boxplot(aes(color=series_name), show.legend=FALSE) +  stat_summary(aes(label=round(..y..,2)), fun.y=median, geom="text", size=2.5, vjust = -0.5) + ggtitle("\t\t\t\t\t Série mais bem avaliada (de acordo com dados do IMDB)") + xlab("Nome da série") + ylab("Nota dos episódios") + theme(axis.text.x = element_text(angle = 90, hjust = 1))
dataSeriesBP
{% endhighlight %}

<img src="/data-science-ad1/figure/source/lab01/checkpoint1/2017-19-05-LAB01Markdown/unnamed-chunk-2-1.png" title="plot of chunk unnamed-chunk-2" alt="plot of chunk unnamed-chunk-2" style="display: block; margin: auto;" />

De acordo com o boxplot, podemos ver que a série mais bem avaliada foi How To Get Away With Murder, entre as 6 séries escolhidas. Mas a diferença entre as notas das séries é bem pequena. A mediana de How To Get Away With Murder é 8.7, enquanto 13 Reasons Why possui nota 8.5, junto com Friends. A série com avaliação "mais baixa" é Modern Family, com mediana 8, uma diferença de 7 décimos para a melhor série. No geral, todas as séries possuem nota acima ou igual à 8, o que são avaliações extremamente positivas.    



**2. Qual das séries que eu escolhi tem episódios de qualidade mais irregular segundo o IMDB? A diferença é grande? Pequena?**

Para efetuar essa análise, podemos utilizar duas ferramentas: O **desvio padrão** ou o **IQR**. Vamos utilizar as duas ferramentas e comparar os resultados. Mas antes, vamos relembrar o conceito de cada uma.

O desvio padrão é uma medida utilizada considerando a média das notas de todos os episódios da série. Quanto maior o desvio padrão, mais distante as notas estão da média, ou seja, mais espalhados os dados estão.

Abaixo, temos um gráfico de pontos onde as séries estão ordenadas de forma crescente pelo desvio padrão. Obtemos o seguinte resultado:


{% highlight r %}
ggplot(dataSeries, aes(x=reorder(series_name, UserRating, FUN=sd), y = UserRating)) + geom_jitter(aes(color=series_name), show.legend=FALSE) + stat_summary(aes(x = series_name, y = UserRating), fun.y = mean, fun.ymin = mean, fun.ymax = mean, geom = "crossbar", width = 0.5, inherit.aes = FALSE) + stat_summary(aes(label=round(..y..,2)), fun.y=sd, geom="text", size=2.5, vjust = -0.5) + ggtitle("\t\t\t\t Séries com episódios de qualidade mais irregular (Usando DP)") + xlab("Nome da série e desvio padrão") + ylab("Nota dos episódios") + theme(axis.text.x = element_text(angle = 90, hjust = 1))
{% endhighlight %}

<img src="/data-science-ad1/figure/source/lab01/checkpoint1/2017-19-05-LAB01Markdown/unnamed-chunk-3-1.png" title="plot of chunk unnamed-chunk-3" alt="plot of chunk unnamed-chunk-3" style="display: block; margin: auto;" />

A barrinha no meio representa a média das notas dos episódios. Quanto mais os episódios (pontos) estão distantes dessa barrinha central, maior o desvio padrão e, consequentemente, maior a irregularidade das notas.

Para uma precisão maior, podemos aliar o uso do desvio padrão ao IQR (Intervalo Inter Quartil), que é uma medida que considera a diferença entre o terceiro e primeiro quartil. Quanto maior a largura do boxplot (a diferença entre os quartis), mais os dados estão espalhados/dispersos.

Vamos utilizar mais uma vez o gráfico de caixa ordenando as séries do menor para o maior IQR. Obtemos o seguinte resultado:


{% highlight r %}
ggplot(dataSeries, aes(x=reorder(series_name, UserRating, FUN=IQR), y = UserRating)) + geom_boxplot(aes(color=series_name), show.legend=FALSE) + stat_summary(aes(label=round(..y..,2)), fun.y=IQR, geom="text", size=2.5, vjust = -0.5) + ggtitle("\t\t\t\t Séries com episódios de qualidade mais irregular (Usando IQR)") + xlab("Nome da série e IQR") + ylab("Nota dos episódios") + theme(axis.text.x = element_text(angle = 90, hjust = 1))
{% endhighlight %}

<img src="/data-science-ad1/figure/source/lab01/checkpoint1/2017-19-05-LAB01Markdown/unnamed-chunk-4-1.png" title="plot of chunk unnamed-chunk-4" alt="plot of chunk unnamed-chunk-4" style="display: block; margin: auto;" />

Embora o desvio padrão seja muito utilizado, este ainda é afetado por notas extremas e não foca onde a maioria dos valores estão concentrados.  Aliando os dois gráficos acima podemos perceber que as duas séries que variam mais são Black Mirror e 13 Reasons Why, utilizando as duas medidas. Ou seja, as duas apresentam dados irregulares ao considerar o desvio padrão das notas dos episódios e o intervalo inter-quartil.     




