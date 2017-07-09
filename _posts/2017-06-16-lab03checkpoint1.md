---
layout: post
title: "Filmes by Rodrigo Santoro"
date: 2017-07-09 15:32:17
published: true
tags: [htmlwidgets, r]
author: "Paulo Vinicius Soares"
output: 
  html_document:
        toc: true
        toc_float: true
        toc_depth: 4
---




{% highlight r %}
# Imports
library(tidyverse, warn.conflicts = F)
library(rvest)
library(plotly)
library(cluster)
library(ggdendro)
library(fpc)
source("plota_solucoes_hclust.R")
{% endhighlight %}
# Introdução
Olá! Este texto é uma continuação das análises referentes à disciplina de Análise de Dados I, na Universidade Federal de Campina Grande (UFCG). Na análise de hoje iremos abordar os filmes estrelados por **Rodrigo Santoro**, artista brasileiro de grande reconhecimento internacional atuando como galã de vários filmes e com um currículo de dar inveja em qualquer ator.

Nesta análise utilizaremos **duas variáveis** para traçar perfis dos filmes estrelados por Rodrigo. Estas variáveis são o **Box Office** e **Rating**, que apontam para a bilheteria que o filme arrecadou e avaliação pelo público, respectivamente.

Vamos visualizar abaixo os filmes representados por pontos em um gráfico que demonstra a relação entre a bilheteria do filme e sua avaliação pelo público.

# Análise do gráfico de clustering

{% highlight r %}
from_page <- read_html("https://www.rottentomatoes.com/celebrity/rodrigo_santoro/") %>% 
    html_node("#filmographyTbl") %>% # A sintaxe da expressão é de um seletor à lá JQuery: https://rdrr.io/cran/rvest/man/html_nodes.html 
    html_table(fill=TRUE) %>% # Faz parse
    as.tibble()

filmes = from_page %>% 
    filter(RATING != "No Score Yet", 
           `BOX OFFICE` != "—", 
           CREDIT != "Executive Producer") %>%
    mutate(RATING = as.numeric(gsub("%", "", RATING)), 
           `BOX OFFICE` = as.numeric(gsub("[$|M]", "", `BOX OFFICE`))) %>% 
    filter(`BOX OFFICE` >= 1) 
{% endhighlight %}


{% highlight r %}
# Utilizando o Ward Hierarchical Clustering

agrupamento_h_2d = filmes %>% 
    column_to_rownames("TITLE") %>%
    select(RATING, `BOX OFFICE`) %>%
    dist(method = "euclidean") %>% 
    hclust(method = "centroid")

plota_hclusts_2d(agrupamento_h_2d, 
                 filmes, 
                 c("RATING", "`BOX OFFICE`"), 
                 linkage_method = "average", ks = 1:6) + scale_y_log10()
{% endhighlight %}

![plot of chunk unnamed-chunk-3](/data-science-ad1/figure/source/lab03/checkpoint1/2017-06-16-lab03checkpoint1/unnamed-chunk-3-1.png)


Agora, utilizando uma técnica chamada **agrupamento** vamos criar blocos que englobam vários filmes atribuindo-lhe uma categoria. Primeiro, vamos esclarecer algumas coisas: O algoritmo de *clustering* que iremos utilizar é o **euclidiano**, que calcula a distância de um ponto para os demais pontos no gráfico. Dessa forma, pontos que estão muito distantes serão colocados em outro grupo e, consequentemente, pontos mais próximos serão inseridos no mesmo grupo.  
Acima, no gráfico, temos vários agrupamentos. O algoritmo vai agrupando os pontos em uma quantidade x de grupos, começando de 1 e indo até 6. Vamos utilizar o que consideramos a melhor quantidade de grupos para efetuarmos nossa análise.  
De acordo com o gráfico acima, podemos perceber que a melhor divisão ocorre quando há 6 grupos, onde o algoritmo considera com mais precisão o eixo x, o da avaliação pelo público. Vamos visualizar esse agrupamento.

# Análise dos grupos encontrados

{% highlight r %}
plota_hclusts_2d(agrupamento_h_2d, 
                 filmes, 
                 c("RATING", "`BOX OFFICE`"), 
                 linkage_method = "average", ks = 6) + scale_y_log10()
{% endhighlight %}

![plot of chunk unnamed-chunk-4](/data-science-ad1/figure/source/lab03/checkpoint1/2017-06-16-lab03checkpoint1/unnamed-chunk-4-1.png)

Podemos definir os 6 grupos da seguinte forma para o gráfico acima:  

- Grupo 1: Grupo de filmes que teve pior avaliação, entre 0-30 e com bilheteria bem abaixo de 100 milhões de dólares. Não há filmes muito conhecidos nessa categoria. Os mais conhecidos são **Ben-Hur**, onde Rodrigo interpreta o papel de Jesus e **Pelé: Birth of a Legend**  

- Grupo 2: Grupo de filmes que teve avaliação extremamente positiva, entre aproximadamente 40 e 80, porém a bilheteria muito baixa, mal passando dos 10 milhões de dólares. Filmes como **Redbelt** e **Che: Part One (The Argentine)** se encaixam nessa categoria.  

- Grupo 3: Grupo de filmes que teve avaliação mediana, aproximadamente 60, e bilheteria mediana, chegando quase aos 100 milhões de dólares. Filmes famosos que se encaixam nessa categoria foram **Rio 2**, por exemplo.  

- Grupo 4: Grupo de filmes que teve avaliação mediana, aproximadamente 50, porém lucrou em torno de 100 milhões de dólares. **Charlie's Angels - Full Throttle** é um exemplar dessa categoria.  

- Grupo 5: Grupo de filmes de maior bilheteria em relação à nota. Nesse grupo, o filme foi o segundo melhor avaliado e gerou a segunda maior receita. Nesse grupo encontramos o filme **Rio**, animação bastante famosa onde Rodrigo interpretou o papel de Túlio.  

- Grupo 6: Grupo de filmes de maior bilheteria dentre todos. Abrange o filme que conseguiu se destacar entre os demais nas vendas de ingressos. Nesse grupo está o filme **300**, talvez o mais conhecido deste ator, onde ele interpretou o papel de Xerxes, e que foi um sucesso aclamado pela crítica mundial.  

Essa foi a análise dessa semana, espero que tenham gostado. Até a próxima!


