<!-- README.md is generated from README.Rmd. Please edit that file -->
[![Build Status](https://travis-ci.org/ropensci/rfishbase.svg?branch=rfishbase2.0)](https://travis-ci.org/ropensci/rfishbase) [![Coverage Status](https://coveralls.io/repos/ropensci/rfishbase/badge.svg?branch=rfishbase2.0)](https://coveralls.io/r/ropensci/rfishbase?branch=rfishbase2.0) [![Downloads](http://cranlogs.r-pkg.org/badges/rfishbase)](https://github.com/metacran/cranlogs.app) [![Onbaroding](https://badges.ropensci.org/137_status.svg)](https://github.com/ropensci/onboarding/issues/137)

<details> <summary><strong>FishBase NEEDS YOUR HELP!</strong></summary>
Dear FishBase Users,

FishBase needs help and I am writing to you because you either have at one point or another requested data to be extracted from FishBase for your own research purposes or have contributed your own data to FishBase.

One of the FishBase funders has had to reduce its commitment and as a result, there is a US$200,000 gap in the FishBase 2018/2019 budget, which will result in forced unpaid leave of key staff with direct consequences for the constant updating and growth of FishBase, a resource on which many of us rely.

Many of us use FishBase regularly in our work given it provides important data on distribution, traits etc. Indeed, these data are so valuable that FishBase receives over 700,000 unique visits per month and underpins key scientific breakthroughs such as the Nature paper on rates of evolution [it’s slower in the tropics!] (see Nature (see https://www.nature.com/articles/d41586-018-05575-2 and https://www.facebook.com/FishBase/posts/1885134558216592).

Key about FishBase is that it is free to everyone in the world, regardless of whether their institutions can afford journal subscriptions.

FishBase co-founder Daniel Pauly once said “sending a bibliography is like providing cookbooks in a famine” and it has been the underpinning ethos of FishBase to make information available equally, regardless of where one works.

So for nearly 30 years, FishBase (www.fishbase.org), with its team of biologists and programmers has done just that, while constantly improving and expanding this valued resource.

But FishBase needs our help now. So, when you are next online on FishBase and see the donate request pop up, please donate at least $25. That’s one bottle of good VQA wine in Canada or half a carton of decent beer in Australia, 3 packs of organic Hess avocados from Loblaws or 6 latte grandes at Starbucks. If you drink more beer, use more avocado on your toast or are caffeine dependent, please consider a larger donation. IF EVERY MARINE RESEARCHER GETS ON BOARD, we can make a major contribution to FishBase. Imagine if you had to pay to access this type of information.

It’s time to pay it forward!

Thank you for your consideration and we all look forward to a flood of world-wide support to FishBase.
</details>

<br>

Welcome to `rfishbase 3.0`. This package is the third rewrite of the original `rfishbase` package described in [Boettiger et al. (2012)](http://www.carlboettiger.info/assets/files/pubs/10.1111/j.1095-8649.2012.03464.x.pdf), and is not backwards compatible with the original. The first version of `rfishbase` relied on the XML summary pages provided by FishBase, which contained relatively incomplete data and have since been deprecated. The package later added functions that relied on HTML scraping of fishbase.org, which was always slow, subject to server instabilities, and carried a greater risk of errors.

To address all of these issues, we created `rfishbase 2.0` accompanied by a stand-alone FishBase API with the blessing of the FishBase.org team, who have kindly provided copies of the backend SQL database to our team for this purpose. At this time the API does not cover all tables provided by the SQL backend, but does access the largest and most commonly used. A list of all tables available from the API (and from rfishbase) can be seen using the `heartbeat()` function.

`rfishbase` 3.0 queries pre-compressed tables from a static server and employs local caching (through memoization) to provide much greater performance and stability, particularly for dealing with large queries involving 10s of thousands of species. The user is never expected to deal with pagination or curl headers and timeouts.

`rfishbase` 3.0 tries to maintain as much backwards compatibility as possible with rfishbase 2.0. However, there are cases in which the rfishbase 2.0 behavior was not desirable -- such as throwing errors when a introducing simple `NA`s for missing data would be more appropriate, or returning vectors where `data.frame`s were needed to include all the context.

We welcome any feedback, issues or questions that users may encounter through our issues tracker on GitHub: <https://github.com/ropensci/rfishbase/issues>

Installation
------------

``` r
remotes::install_github("ropensci/rfishbase")
```

``` r
library("rfishbase")
```

Getting started
---------------

[FishBase](http://fishbase.org) makes it relatively easy to look up a lot of information on most known species of fish. However, looking up a single bit of data, such as the estimated trophic level, for many different species becomes tedious very soon. This is a common reason for using `rfishbase`. As such, our first step is to assemble a good list of species we are interested in.

### Building a species list

Almost all functions in `rfishbase` take a list (character vector) of species scientific names, for example:

``` r
fish <- c("Oreochromis niloticus", "Salmo trutta")
```

You can also read in a list of names from any existing data you are working with. When providing your own species list, you should always begin by validating the names. Taxonomy is a moving target, and this well help align the scientific names you are using with the names used by FishBase, and alert you to any potential issues:

``` r
fish <- validate_names(c("Oreochromis niloticus", "Salmo trutta"))
```

Another typical use case is in wanting to collect information about all species in a particular taxonomic group, such as a Genus, Family or Order. The function `species_list` recognizes six taxonomic levels, and can help you generate a list of names of all species in a given group:

``` r
fish <- species_list(Genus = "Labroides")
fish
```

    [1] "Labroides dimidiatus"    "Labroides bicolor"      
    [3] "Labroides pectoralis"    "Labroides phthirophagus"
    [5] "Labroides rubrolabiatus"

`rfishbase` also recognizes common names. When a common name refers to multiple species, all matching species are returned:

``` r
trout <- common_to_sci("trout")
trout
```

    # A tibble: 118 x 4
       Species                   ComName              Language SpecCode
       <chr>                     <chr>                <chr>       <int>
     1 Salmo obtusirostris       Adriatic trout       English      6210
     2 Schizothorax richardsonii Alawan snowtrout     English      8705
     3 Schizopyge niger          Alghad snowtrout     English     24454
     4 Salvelinus fontinalis     American brook trout English       246
     5 Salmo trutta              Amu-Darya trout      English       238
     6 Salmo kottelati           Antalya trout        English     67602
     7 Oncorhynchus apache       Apache Trout         English      2687
     8 Oncorhynchus apache       Apache trout         English      2687
     9 Plectropomus areolatus    Apricot trout        English      6082
    10 Salmo trutta              Aral Sea Trout       English       238
    # ... with 108 more rows

Note that there is no need to validate names coming from `common_to_sci` or `species_list`, as these will always return valid names.

### Getting data

With a species list in place, we are ready to query fishbase for data. Note that if you have a very long list of species, it is always a good idea to try out your intended functions with a subset of that list first to make sure everything is working.

The `species()` function returns a table containing much (but not all) of the information found on the summary or homepage for a species on [fishbase.org](http://fishbase.org). `rfishbase` functions always return [tidy](http://www.jstatsoft.org/v59/i10/paper) data tables: rows are observations (e.g. a species, individual samples from a species) and columns are variables (fields).

``` r
species(trout$Species)
```

    # A tibble: 118 x 98
       SpecCode Species     SpeciesRefNo Author      FBname   PicPreferredName
          <int> <chr>              <int> <chr>       <chr>    <chr>           
     1     6210 Salmo obtu…        59043 (Heckel, 1… Adriati… Saobt_u0.jpg    
     2     8705 Schizothor…         4832 (Gray, 183… Snowtro… Scric_u1.jpg    
     3    24454 Schizopyge…         4832 (Heckel, 1… Alghad … <NA>            
     4      246 Salvelinus…        86798 (Mitchill,… Brook t… Safon_u4.jpg    
     5      238 Salmo trut…         4779 Linnaeus, … Sea tro… Satru_u2.jpg    
     6    67602 Salmo kott…        99540 Turan, Do?… Antalya… Sakot_m0.jpg    
     7     2687 Oncorhynch…         5723 (Miller, 1… Apache … Onapa_u0.jpg    
     8     2687 Oncorhynch…         5723 (Miller, 1… Apache … Onapa_u0.jpg    
     9     6082 Plectropom…         5222 (R<fc>ppel… Squaret… Plare_u4.jpg    
    10      238 Salmo trut…         4779 Linnaeus, … Sea tro… Satru_u2.jpg    
    # ... with 108 more rows, and 92 more variables: PicPreferredNameM <chr>,
    #   PicPreferredNameF <chr>, PicPreferredNameJ <chr>, FamCode <int>,
    #   Subfamily <chr>, GenCode <int>, SubGenCode <int>, BodyShapeI <chr>,
    #   Source <chr>, AuthorRef <chr>, Remark <chr>, TaxIssue <int>,
    #   Fresh <int>, Brack <int>, Saltwater <int>, DemersPelag <chr>,
    #   AnaCat <chr>, MigratRef <int>, DepthRangeShallow <int>,
    #   DepthRangeDeep <int>, DepthRangeRef <int>, DepthRangeComShallow <int>,
    #   DepthRangeComDeep <int>, DepthComRef <int>, LongevityWild <int>,
    #   LongevityWildRef <int>, LongevityCaptive <dbl>, LongevityCapRef <int>,
    #   Vulnerability <dbl>, Length <dbl>, LTypeMaxM <chr>,
    #   LengthFemale <dbl>, LTypeMaxF <chr>, MaxLengthRef <int>,
    #   CommonLength <dbl>, LTypeComM <chr>, CommonLengthF <int>,
    #   LTypeComF <chr>, CommonLengthRef <int>, Weight <dbl>,
    #   WeightFemale <dbl>, MaxWeightRef <int>, Pic <chr>,
    #   PictureFemale <chr>, LarvaPic <chr>, EggPic <chr>,
    #   ImportanceRef <int>, Importance <chr>, PriceCateg <chr>,
    #   PriceReliability <chr>, Remarks7 <chr>, LandingStatistics <chr>,
    #   Landings <chr>, MainCatchingMethod <chr>, II <chr>, MSeines <int>,
    #   MGillnets <int>, MCastnets <int>, MTraps <int>, MSpears <int>,
    #   MTrawls <int>, MDredges <int>, MLiftnets <int>, MHooksLines <int>,
    #   MOther <int>, UsedforAquaculture <chr>, LifeCycle <chr>,
    #   AquacultureRef <int>, UsedasBait <chr>, BaitRef <int>, Aquarium <chr>,
    #   AquariumFishII <chr>, AquariumRef <int>, GameFish <int>,
    #   GameRef <int>, Dangerous <chr>, DangerousRef <int>,
    #   Electrogenic <chr>, ElectroRef <int>, Complete <chr>,
    #   GoogleImage <int>, Comments <chr>, Profile <chr>, PD50 <dbl>,
    #   Emblematic <int>, Entered <int>, DateEntered <dttm>, Modified <int>,
    #   DateModified <dttm>, Expert <int>, DateChecked <dttm>, TS <chr>

Most tables contain many fields. To avoid overly cluttering the screen, `rfishbase` displays tables as `data_frame` objects from the `dplyr` package. These act just like the familiar `data.frames` of base R except that they print to the screen in a more tidy fashion. Note that columns that cannot fit easily in the display are summarized below the table. This gives us an easy way to see what fields are available in a given table. For instance, from this table we may only be interested in the `PriceCateg` (Price category) and the `Vulnerability` of the species. We can repeat the query for our full species list, asking for only these fields to be returned:

``` r
dat <- species(trout$Species, fields=c("Species", "PriceCateg", "Vulnerability"))
dat
```

    # A tibble: 118 x 3
       Species                   PriceCateg Vulnerability
       <chr>                     <chr>              <dbl>
     1 Salmo obtusirostris       very high           47.0
     2 Schizothorax richardsonii unknown             34.8
     3 Schizopyge niger          unknown             46.8
     4 Salvelinus fontinalis     very high           43.4
     5 Salmo trutta              very high           60.0
     6 Salmo kottelati           <NA>                33.7
     7 Oncorhynchus apache       very high           53.8
     8 Oncorhynchus apache       very high           53.8
     9 Plectropomus areolatus    very high           57.0
    10 Salmo trutta              very high           60.0
    # ... with 108 more rows

### FishBase Docs: Discovering data

Unfortunately identifying what fields come from which tables is often a challenge. Each summary page on fishbase.org includes a list of additional tables with more information about species ecology, diet, occurrences, and many other things. `rfishbase` provides functions that correspond to most of these tables.

Because `rfishbase` accesses the back end database, it does not always line up with the web display. Frequently `rfishbase` functions will return more information than is available on the web versions of the these tables. Some information found on the summary homepage for a species is not available from the `species` summary function, but must be extracted from a different table. For instance, the species `Resilience` information is not one of the fields in the `species` summary table, despite appearing on the species homepage of fishbase.org. To discover which table this information is in, we can use the special `rfishbase` function `list_fields`, which will list all tables with a field matching the query string:

``` r
list_fields("Resilience")
```

    # A tibble: 1 x 1
      table 
      <chr> 
    1 stocks

This shows us that this information appears on the `stocks` table. Working in R, it is easy to query this additional table and combine the results with the data we have collected so far:

``` r
stocks(trout$Species, fields=c("Species", "Resilience", "StockDefs"))
```

    # A tibble: 160 x 3
       Species                   Resilience StockDefs                         
       <chr>                     <chr>      <chr>                             
     1 Salmo obtusirostris       Medium     Europe:  Adriatic basin in Krka, …
     2 Schizothorax richardsonii Medium     Asia:  Himalayan region of India,…
     3 Schizopyge niger          Medium     Asia:  Kashmir Valley in India an…
     4 Salvelinus fontinalis     Medium     North America:  native to most of…
     5 Salmo trutta              High       Europe and Asia:  Atlantic, North…
     6 Salmo trutta              <NA>       <i>Salmo trutta aralensis</i>:  A…
     7 Salmo trutta              Medium     <i>Salmo trutta fario</i>:  North…
     8 Salmo trutta              Low        "<i>Salmo trutta lacustris</i>\t:…
     9 Salmo trutta              <NA>       "<i>Salmo trutta oxianus</i>\t:  …
    10 Salmo trutta              <NA>       <i>Salmo trutta aralensis</i>:  A…
    # ... with 150 more rows

Sometimes it is more useful to search for a broad description of the tables.

SeaLifeBase
-----------

Support for SeaLifeBase not yet implemented in `3.0`. Should be coming soon!

------------------------------------------------------------------------

Please note that this project is released with a [Contributor Code of Conduct](CONDUCT.md). By participating in this project you agree to abide by its terms.

[![ropensci\_footer](http://ropensci.org/public_images/github_footer.png)](http://ropensci.org)
