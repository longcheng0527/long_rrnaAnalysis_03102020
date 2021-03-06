Analyzing the sensitivity and specificity of ASVs for discriminating
between genomes
================
Pat Schloss
9/9/2020

    library(tidyverse)
    library(here)

### Need to determine the numbe of *rrn* operons across genomes

Our analysis will use full length sequences

    count_tibble <- read_tsv(here("data/processed/rrnDB.count_tibble"),
                                                     col_types = "cccd")

We want to count and plot the number of copies per genome

    count_tibble %>%
        filter(region == "v19") %>%
        group_by(genome) %>%
        summarize(n_rrn = sum(count), .groups="drop") %>%
        ggplot(aes(x=n_rrn)) + geom_histogram(binwidth=1)

![](2020-09-09-genome-sens-spec_files/figure-gfm/n_rrn-1.png)<!-- -->

    count_tibble %>%
        filter(region == "v19") %>%
        group_by(genome) %>%
        summarize(n_rrn = sum(count)) %>%
        count(n_rrn) %>%
        mutate(fraction = n / sum(n))

    ## `summarise()` ungrouping output (override with `.groups` argument)

    ## # A tibble: 20 x 3
    ##    n_rrn     n  fraction
    ##    <dbl> <int>     <dbl>
    ##  1     1  1566 0.102    
    ##  2     2  1740 0.113    
    ##  3     3  2143 0.139    
    ##  4     4  1769 0.115    
    ##  5     5  1232 0.0801   
    ##  6     6  2120 0.138    
    ##  7     7  2671 0.174    
    ##  8     8   964 0.0626   
    ##  9     9   363 0.0236   
    ## 10    10   322 0.0209   
    ## 11    11   140 0.00910  
    ## 12    12   157 0.0102   
    ## 13    13    66 0.00429  
    ## 14    14   101 0.00656  
    ## 15    15    23 0.00149  
    ## 16    16     5 0.000325 
    ## 17    17     4 0.000260 
    ## 18    18     1 0.0000650
    ## 19    19     1 0.0000650
    ## 20    21     1 0.0000650

We see that most genomes actually have more than one copy of the *rrn*
operon. I wonder whether those different copies are the same sequence /
ASV…

### Determine number of ASVs per genome

Considering most genomes have multiple copes of the *rrn* operon, we
need to know whether they all have the same ASV. Otherwise we run the
risk of splitting a single genome into multiple ASVs.

    count_tibble %>% 
        group_by(region, genome) %>%
        summarize(n_asv = n(), n_rrn = sum(count), .groups="drop") %>%
        group_by(region, n_rrn) %>%
        summarize(med_n_asv = median(n_asv),
                            mean_n_asv = mean(n_asv),
                            lq_n_asv = quantile(n_asv, prob=0.25),
                            uq_n_asv = quantile(n_asv, prob=0.75)) %>%
        filter(n_rrn == 7)

    ## `summarise()` regrouping output by 'region' (override with `.groups` argument)

    ## # A tibble: 4 x 6
    ## # Groups:   region [4]
    ##   region n_rrn med_n_asv mean_n_asv lq_n_asv uq_n_asv
    ##   <chr>  <dbl>     <dbl>      <dbl>    <dbl>    <dbl>
    ## 1 v19        7         5       4.51        3        6
    ## 2 v34        7         2       2.10        1        3
    ## 3 v4         7         1       1.48        1        2
    ## 4 v45        7         1       1.64        1        2

    count_tibble %>% 
        group_by(region, genome) %>%
        summarize(n_asv = n(), n_rrn = sum(count), .groups="drop") %>%
        ggplot(aes(x=n_rrn, y=n_asv, color=region)) + geom_smooth(method="lm")

    ## `geom_smooth()` using formula 'y ~ x'

![](2020-09-09-genome-sens-spec_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

Surprisingly (or not!) the number of ASVs increases at a rate of about 2
ASVs per 3 copies of *rrn* operon in the genome. The sub regions of the
16S rRNA region have few ASVs per *rrn* operon.

### Determine whether an ASV is unique to genomes they’re found in

Instead of looking at the number of ASVs per genome, we want to see the
number of genomes per ASV.

    count_tibble %>%
        group_by(region, asv) %>%
        summarize(n_genomes = n()) %>%
        count(n_genomes) %>%
        mutate(fraction = n/sum(n)) %>%
        filter(n_genomes == 1)

    ## `summarise()` regrouping output by 'region' (override with `.groups` argument)

    ## # A tibble: 4 x 4
    ## # Groups:   region [4]
    ##   region n_genomes     n fraction
    ##   <chr>      <int> <int>    <dbl>
    ## 1 v19            1 19246    0.824
    ## 2 v34            1  7246    0.779
    ## 3 v4             1  4592    0.759
    ## 4 v45            1  5717    0.778

We see that will full length sequences, that 82% of the ASVs were unique
to a genome. For the subregions, about 76% of the ASVs were unique to a
genome.

### To be determined…

-   Can correct for over representation?
-   Consider analysis at species, genus, family, etc. levels
-   Consider looking at more broad definition of an ASV
