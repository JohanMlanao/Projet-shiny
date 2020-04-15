Projet expertise
================
Romain Magrin-Chagnolleau et Johan Mlanao
24 avril

L’objectif de ce projet est de concevoir et déployer une application
Shiny permettant de suivre l’évolution des campagnes de financement
participatif du site Ulule.

Les données que nous avons concernent des campagnes ulule. Nous avons
plusieurs informations sur chaque campagne, comme par exemple si la
campagne a été financé ou non ou sa date de commencement. Nous devons
dans un premier temps trier ces données afin de pouvoir faire des
analyses dessus.

*Importation des packages*

``` r
library(tidyr)
library(lubridate)
```

    ## 
    ## Attaching package: 'lubridate'

    ## The following object is masked from 'package:base':
    ## 
    ##     date

``` r
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:lubridate':
    ## 
    ##     intersect, setdiff, union

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(ggplot2)
```

# Périmètre

*Extraction des données*

``` r
data <- read.csv2(file="data_ulule_2019.csv")
```

*Exclure du périmètre les campagnes annulées*

``` r
datanew <- subset(data,is_cancelled=="FALSE")
```

*Se restreindre au périmètre des 8 pays ayant le plus de campagnes au
total*

``` r
summary(data$country)
```

    ##      FR      BE              IT      CA      ES      CH      DE      GB      US 
    ##   39920    1806    1636     903     800     690     215     107      77      61 
    ##      RE      NL      LU      PT      TG      MG      BF      MQ      CM      BJ 
    ##      58      52      43      36      34      29      25      25      24      22 
    ##      GP      SN      MA      GR      KH      RO      DK      GF      IN      PE 
    ##      22      19      18      16      16      16      14      14      14      14 
    ##      HU      PL      BG      BR      HR      IE      SE      VN      CI      NC 
    ##      13      13      12      12      10      10       9       9       8       8 
    ##      AT      FI      NO      NP      CN      JP      MX      AR      CO      LA 
    ##       7       7       7       7       6       6       6       5       5       5 
    ##      LV      AU      BO      CF      CR      HT      LT      MD      NE      PH 
    ##       5       4       4       4       4       4       4       4       4       4 
    ##      SI      TN      TR      YT      ZA      CD      EC      IL      KE      MC 
    ##       4       4       4       4       4       3       3       3       3       3 
    ##      MN      PF      RU      TH      AM      BI      BL      CG      CL      CZ 
    ##       3       3       3       3       2       2       2       2       2       2 
    ##      EG      GT      GY      HN      ID      IS      KM      LK      ML      PM 
    ##       2       2       2       2       2       2       2       2       2       2 
    ##      RS      SM      TD      TZ      VE      AE      AF      BD      CU (Other) 
    ##       2       2       2       2       2       1       1       1       1      24

``` r
data2 <- subset(datanew,country=="FR"|country=="BE"|country==""|country=="IT"|country=="CA"|country=="ES"|country=="CH"|country=="DE")
```

*Eliminer les campagnes des tous derniers mois qui paraissent
incompletès*

``` r
data3 <- subset(data2,finished=="TRUE")
```

\#Manipulations de données

*Convertir les montants de toutes les devises en euro*

``` r
#Table de taux de conversion

exchange_rate <- (data3$currency=="AUD")*0.57+(data3$currency=="BRL")*0.19+(data3$currency=="CAD")*0.65+(data3$currency=="CHF")*0.94+(data3$currency=="DKK")*0.13+(data3$currency=="EUR")+(data3$currency=="GBP")*1.12+(data3$currency=="NOK")*0.09+(data3$currency=="SEK")*0.092+(data3$currency=="USD")*0.9

data3$amount_raised <- data3$amount_raised*exchange_rate

data3$currency <- "EUR"
```

Maintenant que nous avons trié nos données, nous allons pouvoir utiliser
Shiny afin de créer des graphiques dynamiques.

Voici des exemples de graphiques que l’on pourra retrouver sur
l’application:

``` r
data_temp <- filter(data3, year(date_start) >= 2013,month(date_start)==1)

value <- as.data.frame(table(year(data_temp$date_start)))
            
            ggplot(value,aes(x=Var1,y=Freq)) +  labs(title = "Evolution du nombres de campagnes au mois de Janvier")+ labs(x="Année",y="Montant moyen") +  geom_line(aes(group=1)) + geom_line(aes(group=1)) +
              geom_point(shape=21, color="black", fill="#69b3a2", size=6) 
```

![](Projet_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

``` r
 data_finance <- (filter(data3,year(date_start)>=2010,month(date_start)>=7,goal_raised==TRUE))
                value <- (filter(data3,year(date_start)>=2010,month(date_start)>=7))
                data_finance <- as.data.frame(table(year(data_finance$date_start)))
                value <- as.data.frame(table(year(value$date_start)))
                
                data_finance$Freq <- data_finance$Freq/value$Freq
                
                ggplot(data_finance,aes(x=Var1,y=Freq)) +  labs(title = "Taux de campagnes financées à partir de juillet")+ labs(x="Année",y="Taux") +  geom_line(aes(group=1)) + geom_line(aes(group=1)) +
                    geom_point(shape=21, color="black", fill="#69b3a2", size=6)
```

![](Projet_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

``` r
data_amount <- (filter(data3,year(date_start)>=2015,month(date_start)>=10,goal_raised==TRUE))
                montant_moyen <- rep(0,2019-2015+1)
                x<- seq(0,1,length.out=35)
                for(i in 2010:2019){
                    temp <- which (year(data_amount$date_start)== i)
                    montant_moyen[i+1-2015] <- mean(data_amount$amount_raised[temp])
                }
                annee <- seq(2015,2019)
                montant_moyen <- cbind(annee,montant_moyen)
                montant_moyen <- as.data.frame(montant_moyen)
                ggplot(montant_moyen,aes(x=annee,y=montant_moyen)) + labs(title = "Montant moyen des campagnes à partir d'Octobre")+ labs(x="Année",y="Montant moyen") +  geom_line(aes(group=1)) +
                    geom_point(shape=21, color="black", fill="#69b3a2", size=6) 
```

![](Projet_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->
