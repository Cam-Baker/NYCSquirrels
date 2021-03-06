Central Park Squirrels
================

# Can we Pinpoint Squirrel Behaviour to Specific Regions of Central Park?

``` r
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
# Reading in the data

squirrel_data <- read.csv("2018_Central_Park_Squirrel_Census_-_Squirrel_Data.csv", stringsAsFactors = F)

convertField <- function(data, field){
  data[which(grepl('false',squirrel_data[,field])),field] = 0
  data[which(grepl('true',squirrel_data[,field])),field] = 1
  data[,field] <- as.numeric(data[,field])
  return(data)
}

# Converting fields of interest to 1 and 0

squirrel_data <- convertField(squirrel_data,"Kuks")
squirrel_data <- convertField(squirrel_data,"Quaas")
squirrel_data <- convertField(squirrel_data,"Moans")
squirrel_data <- convertField(squirrel_data,"Tail.flags")
squirrel_data <- convertField(squirrel_data,"Tail.twitches")
squirrel_data <- convertField(squirrel_data,"Approaches")
squirrel_data <- convertField(squirrel_data,"Indifferent")
squirrel_data <- convertField(squirrel_data,"Runs.from")

# Accumulate scores for similar behaviour

squirrel_data$Scared <- rowSums(squirrel_data[,c("Kuks","Quaas","Moans","Tail.flags","Runs.from")])
squirrel_data$Curious <- rowSums(squirrel_data[,c("Tail.twitches","Approaches")]) + abs(squirrel_data$Indifferent - 1)
squirrel_data$Brave <- rowSums(squirrel_data[,c("Tail.twitches","Approaches")]) + abs(squirrel_data$Runs.from - 1)

knitr::kable(head(squirrel_data) %>% select(-Color.notes))
```

|          X |        Y | Unique.Squirrel.ID | Hectare | Shift |     Date | Hectare.Squirrel.Number | Age   | Primary.Fur.Color | Highlight.Fur.Color | Combination.of.Primary.and.Highlight.Color | Location     | Above.Ground.Sighter.Measurement | Specific.Location | Running | Chasing | Climbing | Eating | Foraging | Other.Activities | Kuks | Quaas | Moans | Tail.flags | Tail.twitches | Approaches | Indifferent | Runs.from | Other.Interactions | Lat.Long                                   | Scared | Curious | Brave |
| ---------: | -------: | :----------------- | :------ | :---- | -------: | ----------------------: | :---- | :---------------- | :------------------ | :----------------------------------------- | :----------- | :------------------------------- | :---------------- | :------ | :------ | :------- | :----- | :------- | :--------------- | ---: | ----: | ----: | ---------: | ------------: | ---------: | ----------: | --------: | :----------------- | :----------------------------------------- | -----: | ------: | ----: |
| \-73.95613 | 40.79408 | 37F-PM-1014-03     | 37F     | PM    | 10142018 |                       3 |       |                   |                     | \+                                         |              |                                  |                   | false   | false   | false    | false  | false    |                  |    0 |     0 |     0 |          0 |             0 |          0 |           0 |         0 |                    | POINT (-73.9561344937861 40.7940823884086) |      0 |       1 |     1 |
| \-73.96886 | 40.78378 | 21B-AM-1019-04     | 21B     | AM    | 10192018 |                       4 |       |                   |                     | \+                                         |              |                                  |                   | false   | false   | false    | false  | false    |                  |    0 |     0 |     0 |          0 |             0 |          0 |           0 |         0 |                    | POINT (-73.9688574691102 40.7837825208444) |      0 |       1 |     1 |
| \-73.97428 | 40.77553 | 11B-PM-1014-08     | 11B     | PM    | 10142018 |                       8 |       | Gray              |                     | Gray+                                      | Above Ground | 10                               |                   | false   | true    | false    | false  | false    |                  |    0 |     0 |     0 |          0 |             0 |          0 |           0 |         0 |                    | POINT (-73.97428114848522 40.775533619083) |      0 |       1 |     1 |
| \-73.95964 | 40.79031 | 32E-PM-1017-14     | 32E     | PM    | 10172018 |                      14 | Adult | Gray              |                     | Gray+                                      |              |                                  |                   | false   | false   | false    | true   | true     |                  |    0 |     0 |     0 |          0 |             0 |          0 |           0 |         1 |                    | POINT (-73.9596413903948 40.7903128889029) |      1 |       1 |     0 |
| \-73.97027 | 40.77621 | 13E-AM-1017-05     | 13E     | AM    | 10172018 |                       5 | Adult | Gray              | Cinnamon            | Gray+Cinnamon                              | Above Ground |                                  | on tree stump     | false   | false   | false    | false  | true     |                  |    0 |     0 |     0 |          0 |             0 |          0 |           0 |         0 |                    | POINT (-73.9702676472613 40.7762126854894) |      0 |       1 |     1 |
| \-73.96836 | 40.77259 | 11H-AM-1010-03     | 11H     | AM    | 10102018 |                       3 | Adult | Cinnamon          | White               | Cinnamon+White                             |              |                                  |                   | false   | false   | false    | false  | true     |                  |    0 |     0 |     0 |          0 |             1 |          0 |           1 |         0 |                    | POINT (-73.9683613516225 40.7725908847499) |      0 |       1 |     2 |

``` r
# Group squirrels by hectare, normalizing the behaviour metrics and averaging the coordinates

library(pillar)
```

    ## 
    ## Attaching package: 'pillar'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     dim_desc

``` r
options(pillar.sigfig = 10)

normalize <- function(x){
  return((x - min(x, na.rm = T)) / (max(x, na.rm = T) + 0.0000001 - min(x, na.rm = T)))
}

squirrel_data_grouped_data <- squirrel_data %>% summarise(Hectare = Hectare, norm_scared = normalize(Scared), norm_curious = normalize(Curious), norm_brave = normalize(Brave)) %>% group_by(Hectare) %>% summarize(scared_avg = mean(norm_scared), curious_avg = mean(norm_curious), brave_avg = mean(norm_brave))
squirrel_data_grouped_coords <- squirrel_data %>% group_by(Hectare) %>% summarize(X_avg = mean(X), Y_avg = mean(Y))
squirrel_data_grouped_count <- squirrel_data %>% group_by(Hectare)%>% tally()
squirrel_data_grouped <- left_join(squirrel_data_grouped_data, squirrel_data_grouped_coords, by = "Hectare") %>% left_join(squirrel_data_grouped_count, by = "Hectare")

knitr::kable(head(squirrel_data_grouped))
```

| Hectare | scared\_avg | curious\_avg | brave\_avg |     X\_avg |   Y\_avg |  n |
| :------ | ----------: | -----------: | ---------: | ---------: | -------: | -: |
| 01A     |   0.1212121 |    0.2424242 |  0.3333333 | \-73.98089 | 40.76820 | 11 |
| 01B     |   0.0123457 |    0.0864198 |  0.3703704 | \-73.98024 | 40.76789 | 27 |
| 01C     |   0.0833333 |    0.3611111 |  0.3333333 | \-73.97940 | 40.76756 | 12 |
| 01D     |   0.0000000 |    0.2708333 |  0.4583333 | \-73.97821 | 40.76693 | 16 |
| 01E     |   0.0000000 |    0.2916667 |  0.4583333 | \-73.97736 | 40.76652 |  8 |
| 01F     |   0.0416667 |    0.1666667 |  0.3333333 | \-73.97654 | 40.76619 |  8 |

``` r
library(ggplot2)
library(reshape2)

sorted_scared <- squirrel_data_grouped$scared_avg[order(squirrel_data_grouped$scared_avg)]
sorted_curious <- squirrel_data_grouped$curious_avg[order(squirrel_data_grouped$curious_avg)]
sorted_brave <- squirrel_data_grouped$brave_avg[order(squirrel_data_grouped$brave_avg)]
index = 1:length(sorted_brave)

sorted_data <- data.frame(index,sorted_scared,sorted_curious,sorted_brave)

ggplot(melt(sorted_data, id = "index"), aes(x=index, y = value, color = variable)) + geom_line()
```

![](NYCSquirrels_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

``` r
library(ggmap)
```

    ## Google's Terms of Service: https://cloud.google.com/maps-platform/terms/.

    ## Please cite ggmap if you use it! See citation("ggmap") for details.

``` r
library(grid)
library(gridExtra)
```

    ## 
    ## Attaching package: 'gridExtra'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     combine

``` r
#park.map <- get_map(c(left = min(squirrel_data$X)-0.001, bottom = min(squirrel_data$Y)-0.001, right = max(squirrel_data$X)+0.001, top = max(squirrel_data$Y)+0.001))
#save(park.map, file = "park.map.Robj")
load(file="park.map.Robj")

p1 <- ggmap(park.map) + 
  geom_point(data = squirrel_data_grouped, aes(x = X_avg, y = Y_avg, color = n)) + 
  ggtitle("Number of Squirrels Recorded per Hectare") + 
  theme(plot.title = element_text(hjust = 0.5), plot.margin = unit(c(0,0,0,0),"cm"))

p2 <- ggmap(park.map) + 
  geom_point(data = squirrel_data_grouped, aes(x = X_avg, y = Y_avg, color = scared_avg)) + 
  ggtitle("Average Scared Score per Hectare") + 
  theme(plot.title = element_text(hjust = 0.5), plot.margin = unit(c(0,0,0,0),"cm"))

p3 <- ggmap(park.map) + 
  geom_point(data = squirrel_data_grouped, aes(x = X_avg, y = Y_avg, color = curious_avg)) + 
  ggtitle("Average Curiosity Score per Hectare") + 
  theme(plot.title = element_text(hjust = 0.5), plot.margin = unit(c(0,0,0,0),"cm"))

p4 <- ggmap(park.map) + 
  geom_point(data = squirrel_data_grouped, aes(x = X_avg, y = Y_avg, color = brave_avg)) + 
  ggtitle("Average Bravery Score per Hectare") + 
  theme(plot.title = element_text(hjust = 0.5), plot.margin = unit(c(0,0,0,0),"cm"))

grid.arrange(p1, p2, p3, p4, ncol = 2)
```

![](NYCSquirrels_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->
