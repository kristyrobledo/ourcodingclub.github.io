---
layout: post
title: Efficient data manipulation
subtitle: Use pipes to streamline your code
date: 2017-01-06
updated: 2019-04-04
author: Sandra
updater: Sandra
meta: "Tutorials"
tags: data_manip 
---
<div class="block"> 
          <center>
          <img src="{{ site.baseurl }}/img/tutheader_DL_data_manip_2.png" alt="Img">
          </center>
        </div>

### Tutorial aims:

#### 1. Chain together multiple lines of codes with pipes `%>%`

#### 2. Use `dplyr` to its full potential

#### 3. Automate advanced tasks like plotting without writing a loop



### Steps:

#### <a href="#pipes"> An introduction to pipes

#### <a href="#dplyr"> Discover more functions of `dplyr` 

##### <a href="#filter"> - `summarise_all()`
##### <a href="#case"> - `case_when()`


#### <a href="#factors"> Rename and reorder factor levels or create categorical variables


#### <a href="#piping-graphs"> Advanced piping 


#### <a href="#challenge"> Challenge yourself!



Welcome to our second tutorial on data manipulation! In our (anything but) __basic tutorial__, we learned to subset and modify data to suit most of our coding needs, and to use a tidy data format. Today we dig deeper into the wonderful world of `dplyr` with one of our favourite feature, the pipe operator `%>%`. We also explore some extra `dplyr` functions and give some tips to recode and reclassify values.

#### __All the files you need to complete this tutorial can be downloaded from <a href="https://github.com/ourcodingclub/CC-data-manip-2" target="_blank">this repository</a>. Clone and download the repo as a zip file, then unzip it.__

We are working with a subset of a larger dataset\* of <a href="https://data.edinburghcouncilmaps.info/datasets/4dfc8f18a40346009b9fc32cbee34039_39" target="_blank">trees within the City of Edinburgh</a>. We subsetted this large dataset (over 50 thousand trees!) to the <a href= "https://data.edinburghcouncilmaps.info/datasets/33969ec66f9b46cf9617c40c023bb89e_35" target="_blank">Special Landscape Area</a> around Craigmillar Castle. Our __Spatial analysis tutorials__ could teach you how to do this yourself, but for now the file is all ready for you to use and is named `trees.csv`.


###### \*(Copyright City of Edinburgh Council, contains Ordnance Survey data © Crown copyright and database right 2019)
<br>

#### __Create a new, blank script, and add in some information at the top, for instance the title of the tutorial, your name, and the date (remember to use hasthags `#` to comment and annotate your script).__


<a name="pipes"></a>
### An introduction to pipes

The pipe operator `%>%` is a funny little thing that serves as a channel for the output of a command to be passed to another function seamlessly, i.e., without creating intermediary objects. It really makes your code flow, and avoids repetition. Let's first import the data, and then we'll see what pipes are all about.

<a id="Acode01" class="copy" name="copy_pre" href="#"> <i class="fa fa-clipboard"></i> Copy Contents </a><br>
<section id= "code01" markdown="1"> 
```r
# LIBRARIES
library(dplyr)     # for data manipulation
library(ggplot2)   # for making graphs; make sure you have it installed, or install it now

# Set your working directory
setwd("your-file-path")   # replace with the tutorial folder path on your computer
# If you're working in an R project, skip this step

# LOAD DATA
trees <- read.csv(file = "trees.csv", header = TRUE)

head(trees)  # make sure the data imported OK, familiarise yourself with the variables

```
</section>

Let's say we want to know how many trees of each species are found in the dataset. If you remember our first data manipulation tutorial, this is a task made for the functions `group_by()` and `summarise()`. So we could do this:

<a id="Acode02" class="copy" name="copy_pre" href="#"> <i class="fa fa-clipboard"></i> Copy Contents </a><br>
<section id= "code02" markdown="1"> 
```r
# Count the number of trees for each species

trees.grouped <- group_by(trees, CommonName)    # create an internal grouping structure, so that the next function acts on groups (here, species) separately. 

trees.summary <- summarise(trees.grouped, count = length(CommonName))   # here we use length to count the number of rows (trees) for each group (species). We could have used any row name.

# Alternatively, dplyr has a tally function that does the counts for you!
trees.summary <- tally(trees.grouped)
```
</section>

This works well, but notice how we had to create an extra data frame, `trees.grouped`, before achieving our desired output of `trees.summary`. For a larger, complex analysis, this would rapidly clutter your environment with lots of objects you don't really need!

This is where the pipe comes in to save the day. It takes the data frame created on its left side, and _passes it_ to the function on its right side. This saves you the need for creating intermediary objects, and also avoids repeating the object name in every function: the tidyverse functions "know" that the object that is passed through the pipe is the `data =` argument of that function. 

<a id="Acode03" class="copy" name="copy_pre" href="#"> <i class="fa fa-clipboard"></i> Copy Contents </a><br>
<section id= "code03" markdown="1">
```r 

# Count the number of trees for each species, with a pipe!

trees.summary <- trees %>%                   # the data frame object that will be passed in the pipe
                 group_by(CommonName) %>%    # see how we don't need to name the object, just the grouping variable?
                 tally()                     # and we don't need anything at all here, it has been passed through the pipe!

```
</section>

See how we go from `trees` to `trees.summary` while running one single chunk of code?


__Important notes:__ Pipes only work on data frame objects, and functions outside the tidyverse often require that you specify the data source with a full stop dot `.`. But as we will see later, you can still do advanced things while keeping these limitations in mind!


<div class="bs-callout-yellow" markdown="1">

__We're not lazy, but we love shortcuts!__ In RStudio, you can use `Ctrl + Shift + M` (or `Cmd + Shift + M` on a Mac) to create the `%>%` operator.

</div>


Let's use some more of our favourite `dplyr` functions in pipe chains. Can you guess what this does?

<a id="Acode04" class="copy" name="copy_pre" href="#"> <i class="fa fa-clipboard"></i> Copy Contents </a><br>
<section id= "code04" markdown="1"> 
```r
trees.subset <- trees %>%
                filter(CommonName %in% c("Common Ash", "Rowan", "Scots Pine")) %>% 
                group_by(CommonName, AgeGroup) %>% 
                tally()
```
</section>

Here we are first subsetting the data frame to only three species, and counting the number of trees for each species, but also breaking them down by age group. The intuitive names of `dplyr`'s actions make the code very readable for your colleagues, too. 
 
Neat, uh? Now let's play around with other functions that `dplyr` has to offer.


<a name="dplyr"></a>
### More functions of `dplyr`

An extension of the core `dplyr` functions is `summarise_all()`: you may have guessed, it will run a summary function of your choice over ALL the columns. Not meaningful here, but could be if all values were numeric, for instance. 

<a name="filter"></a>

<a id="Acode07" class="copy" name="copy_pre" href="#"> <i class="fa fa-clipboard"></i> Copy Contents </a><br>
<section id= "code07" markdown="1"> 
```r 
summ.all <- summarise_all(trees, mean)
```
</section>

As only two of the columns had numeric values over which a mean could be calculated, the other columns have missing values.


Now let's move on to a truly exciting function that not so many people know about.


<a name="case"></a>
#### 2. `case_when()` - a favourite for re-classifying values or factors

But first, it seems poor form to introduce this function without also introducing the simpler function upon which it builds, `ifelse()`. You give `ifelse()` a conditional statement which it will evaluate, and the values it should return when this statement is true or false. Let's do a very simple example to begin with:

<a id="Acode08" class="copy" name="copy_pre" href="#"> <i class="fa fa-clipboard"></i> Copy Contents </a><br>
<section id= "code08" markdown="1"> 
```r 
vector <- c(4, 13, 15, 6)      # create a vector to evaluate

ifelse(vector < 10, "A", "B")  # give the conditions: if inferior to 10, return A, if not, return B

# Congrats, you're a dancing queen! (Or king!)
``` 
</section>

The super useful `case_when()` is a generalisation of `ifelse()` that lets you assign more than two outcomes. All logical operators are available, and you assign the new value with a tilde `~`. For instance:

<a id="Acode09" class="copy" name="copy_pre" href="#"> <i class="fa fa-clipboard"></i> Copy Contents </a><br>
<section id= "code09" markdown="1">
```r
vector2 <- c("What am I?", "A", "B", "C", "D")

case_when(vector2 == "What am I?" ~ "I am the walrus",
          vector2 %in% c("A", "B") ~ "goo",
          vector2 == "C" ~ "ga",
          vector2 == "D" ~ "joob")
```
</section>

But enough singing, and let's see how we can use those functions in real life to reclassify our variables.

<a name="factors"></a>
### Changing factor levels or create categorical variables 

The use of `mutate()` together with `case_when()` is a great way to change the names of factor levels, or create a new variable based on existing ones. We see from the `LatinName` columns that there are many tree species belonging to some genera, like birches (Betula), or willows (Salix), for example. We may want to create a `Genus` column using `mutate()` that will hold that information.

We will do this using a character string search with the `grepl` function, which looks for patterns in the data, and specify what to return for each genus. Before we do that, we may want the full list of species occuring in the data!

<a id="Acode10" class="copy" name="copy_pre" href="#"> <i class="fa fa-clipboard"></i> Copy Contents </a><br>
<section id= "code10" markdown="1"> 
```r

unique(trees$LatinName)  # Shows all the species names

# Create a new column with the tree genera

trees.genus <- trees %>%
               mutate(Genus = case_when(               # creates the genus column and specifies conditions
                  grepl("Acer", LatinName) ~ "Acer",
                  grepl("Fraxinus", LatinName) ~ "Fraxinus",
                  grepl("Sorbus", LatinName) ~ "Sorbus",
                  grepl("Betula", LatinName) ~ "Betula",
                  grepl("Populus", LatinName) ~ "Populus",
                  grepl("Laburnum", LatinName) ~ "Laburnum",
                  grepl("Aesculus", LatinName) ~ "Aesculus", 
                  grepl("Fagus", LatinName) ~ "Fagus",
                  grepl("Prunus", LatinName) ~ "Prunus",
                  grepl("Pinus", LatinName) ~ "Pinus",
                  grepl("Sambucus", LatinName) ~ "Sambucus",
                  grepl("Crataegus", LatinName) ~ "Crataegus",
                  grepl("Ilex", LatinName) ~ "Ilex",
                  grepl("Quercus", LatinName) ~ "Quercus",
                  grepl("Larix", LatinName) ~ "Larix",
                  grepl("Salix", LatinName) ~ "Salix",
                  grepl("Alnus", LatinName) ~ "Alnus")
               )
```
</section>

We have searched through the `LatinName`column for each genus name, and specified a value to put in the new `Genus` column for each case. It's a lot of typing, but still quicker than specifying the genus individually for related trees (e.g. _Acer pseudoplatanus_, _Acer platanoides_, _Acer_ spp.). 

__BONUS FUNCTION!__ In our specific case, we could have achieved the same result much quicker. The genus is always the first word of the `LatinName` column, and always separated from the next word by a space. We could use the `separate()` function from the `tidyr` package to split the column into several new columns filled with the words making up the species names, and keep only the first one.

<a id="Acode11" class="copy" name="copy_pre" href="#"> <i class="fa fa-clipboard"></i> Copy Contents </a><br>
<section id= "code11" markdown="1"> 
```r
library(tidyr)
trees.genus.2 <- trees %>% 
                  tidyr::separate(LatinName, c("Genus", "Species"), sep = " ", remove = FALSE) %>%  
                  dplyr::select(-Species)
                  
# we're creating two new columns in a vector (genus name and species name), "sep" refers to the separator, here space between the words, and remove = FALSE means that we want to keep the original column LatinName in the data frame
```
</section>

Mind blowing! Of course, sometimes you have to be typing more, so here is another example of how we can reclassify a factor. The `Height` factor has 5 levels representing brackets of tree heights, but let's say three categories would be enough for our purposes. We create a new height category variable `Height.cat`:

<a id="Acode12" class="copy" name="copy_pre" href="#"> <i class="fa fa-clipboard"></i> Copy Contents </a><br>
<section id= "code12" markdown="1"> 
```r
trees.genus <- trees.genus %>%   # overwriting our data frame 
               mutate(Height.cat =   # creating our new column
                         case_when(Height %in% c("Up to 5 meters", "5 to 10 meters") ~ "Short",
                                   Height %in% c("10 to 15 meters", "15 to 20 meters") ~ "Medium",
                                   Height == "20 to 25 meters" ~ "Tall")
                      )
```
</section>

<div class="bs-callout-blue" markdown="1">

__Reordering factors levels__

We've seen how we can change the names of a factor's levels, but what if you want to change the order in which they display? R will always show them in alphabetical order, which is not very handy if you want them to appear in a more logical order. 

For instance, if we plot the number of trees in each of our new height categories, we may want the bars to read, from left to right: "Short", "Medium", "Tall". However, by default, R will order them "Medium", "Short", "Tall". 

To fix this, you can specify the order explicitly, and even add labels if you want to change the names of the factor levels. Here, we put them in all capitals to illustrate.

<a id="Acode13" class="copy" name="copy_pre" href="#"> <i class="fa fa-clipboard"></i> Copy Contents </a><br>
<section id= "code13" markdown="1"> 
```r
## Reordering a factor's levels

levels(trees.genus$Height.cat)  # shows the different factor levels in their default order

trees.genus$Height.cat <- factor(trees.genus$Height.cat,
                                 levels = c("Short", "Medium", "Tall"),   # whichever order you choose will be reflected in plots etc
                                 labels = c("SHORT", "MEDIUM", "TALL")    # Make sure you match the new names to the original levels!
                                 )   

levels(trees.genus$Height.cat)  # a new order and new names for the levels
```
</section>

Hence, plotting tree counts in each category before and after reordering the factors would look like this.

<center> <img src="{{ site.baseurl }}/img/DL_data-manip-2_treeheights.png" alt="Img" style="width: 850px;"/> </center>


__Reordering according to the value of another column__

Sometimes you might want to set your factor levels based on the numeric value *of another variable*. Imagine you have a dataset of the world's biomes and you want to plot the mean annual temperature in a bar plot. It would be nice for them to be ordered in increasing or decreasing order of temperature, and it makes no sense to have "Tundra" following "Tropical rainforest" just because they're alphabetically close!

The `reorder()` function comes to the rescue here. It takes the factor variable as the first argument, and the numeric variable by which you want to order as a second. So we would have:

```
world$biome <- reorder(world$biome, world$temperature)   # order factor levels in increasing order
world$biome <- reorder(world$biome, -world$temperature)   # order factor levels in decreasing order
```

</div>

Are you now itching to make graphs too? We've kept to base R plotting in our intro tutorials, but we are big fans of `ggplot2` and that's what we'll be using in the next section while we learn to make graphs as outputs of a pipe chain. If you haven't used `ggplot2` before, don't worry, we won't go far with it today. We have <a href="https://ourcodingclub.github.io/tutorials/" target="blank">two whole tutorials</a> dedicated to making pretty and informative plots with it. Install and load the package if you need to:

<a id="Acode14" class="copy" name="copy_pre" href="#"> <i class="fa fa-clipboard"></i> Copy Contents </a><br>
<section id= "code14" markdown="1"> 
```r
install.packages("ggplot2")
library(ggplot2)
```
</section>

And let's build up a plot-producing factory chain!


<a name="piping-graphs"></a>
### Advanced piping 

Earlier in the tutorial, we used pipes to gradually transform our dataframes by adding new columns or transforming the variables they contain. But sometimes you may want to use the really neat grouping functionalities of `dplyr` with non native `dplyr` functions, for instance to run series of models or produce plots. It can be tricky, but it's sometimes easier to write than a loop. (You can learn to write loops <a href="https://ourcodingclub.github.io/2017/02/08/funandloops.html" target="_blank">here</a>.)

First, we'll subset our dataset to just a few tree genera to keep things light. Pick your favourite five, or use those we have defined here! Then we'll map them to see how they are distributed. 

<a id="Acode15" class="copy" name="copy_pre" href="#"> <i class="fa fa-clipboard"></i> Copy Contents </a><br>
<section id= "code15" markdown="1"> 
```r

# Subset data frame to fewer genera

trees.five <- trees.genus %>%
               filter(Genus %in% c("Acer", "Fraxinus", "Salix", "Aesculus", "Pinus"))

# Map all the trees

(map.all <- ggplot(trees.five) +
            geom_point(aes(x = Easting, y = Northing, size = Height.cat, colour = Genus), alpha = 0.5) +
            theme_bw() +
            theme(panel.grid = element_blank(),
                  axis.text = element_text(size = 12),
                  legend.text = element_text(size = 12))
)
```
</section>

<center> <img src="{{ site.baseurl }}/img/DL_data-manip-2_treemap.jpeg" alt="Img" style="width: 800px;"/> </center>


Don't worry too much about all the arguments in the `ggplot` code, they are there to make the graph prettier. The interesting bits are the x and y axis, and the other two parameters we put in the `aes()` call: we're telling the plot to colour the dots according to genus, and to make them bigger or smaller according to our tree height factor. We'll explain everything else in our <a href="https://ourcodingclub.github.io/2017/01/29/datavis.html" target="_blank">data visualisation</a> tutorial. 

Now, let's say we want to save a separate map for each genus (so 5 maps in total). You could filter the data frame five times for each individual genus, and copy and paste the plotting code five times too, but imagine we kept all 17 genera! This is where pipes and `dplyr` come to the rescue again. (If you're savvy with `ggplot2`, you'll know that facetting is often a better option, but sometimes you do want to save things as separate files.) The `do()` function allows us to use pretty much any R function within a pipe chain, provided that we supply the data as `data = .` where the function requires it.

<a id="Acode16" class="copy" name="copy_pre" href="#"> <i class="fa fa-clipboard"></i> Copy Contents </a><br>
<section id= "code16" markdown="1"> 
```r
# Plotting a map for each genus

tree.plots <-  
   trees.five  %>%      # the data frame
   group_by(Genus) %>%  # grouping by genus
   do(plots =           # the plotting call within the do function
         ggplot(data = .) +
         geom_point(aes(x = Easting, y = Northing, size = Height.cat), alpha = 0.5) +
         labs(title = paste("Map of", .$Genus, "at Craigmillar Castle", sep = " ")) +
         theme_bw() +
         theme(panel.grid = element_blank(),
               axis.text = element_text(size = 14),
               legend.text = element_text(size = 12),
               plot.title = element_text(hjust = 0.5),
               legend.position = "bottom")
   ) 
   
# You can view the graphs before saving them
tree.plots$plots
   
# Saving the plots to file

tree.plots %>%              # the saving call within the do function
   do(., 
      ggsave(.$plots, filename = paste(getwd(), "/", "map-", .$Genus, ".png", sep = ""), device = "png", height = 12, width = 16, units = "cm"))
```
</section>

<br> <br> 
<center> <img src="{{ site.baseurl }}/img/DL_data-manip-2_treemaps.png" alt="Img" style="width: 900px;"/> </center><br>
<center> You should get five different plots looking something like this. </center>

Phew! This could even be chained in one long call without creating the `tree.plots` object, but take a moment to explore this object: the plots are saved as _lists_ within the `plots` column that we created. The `do()` function allows to use a lot of external functions within `dplyr` pipe chains. However, it is sometimes tricky to use and is becoming deprecated. <a href="https://www.brodrigues.co/blog/2017-03-29-make-ggplot2-purrr/" target="_blank">This page</a> shows an alternative solution using the `purr` package to save the files.



<div class="bs-callout-yellow" markdown="1">

__Sticking things together with `paste()`__

Did you notice how we used the `paste()` function to define the `filename=` argument of the last piece of code? (We did the same to define the titles that appear on the graphs.) It's a useful function that lets you combine text strings as well as outputs from functions or object names in the environment. Let's take apart that last piece of code here:

```r
paste(getwd(), "/", "map-", .$Genus, ".png", sep = "")
```

- `getwd()`: You are familiar with this call: try it in the console now! It writes the path to your working directory, i.e. the root folder where we want to save the plots.

- "/": we want to add a slash after the directory folder and before writing the name of the plot

- "map-": a custom text bit that will be shared by all the plots. We're drawing maps after all!

- `.$Genus`: accesses the Genus name of the tree.plots object, so each plot will bear a different name according to the tree genus.

- ".png": the file extension; we could also have chosen a pdf, jpg, etc.

- `sep = "": we want all the previous bits to be pasted together with nothing separating them

So, in the end, the whole string could read something like: "C:/Coding_Club/map-Acer.png".

</div>

We hope you've learned new hacks that will simplify your code and make it more efficient! Let's see if you can use what we learned today to accomplish a last data task. 


<a name="challenge"></a>
### Challenge yourself!

The Craigmillar Castle team would like a summary of the different species found within its grounds, but broken down in four quadrants (NE, NW, SE, SW). You can start from the `trees.genus` object created earlier.

1. Can you calculate the species richness (e.g. the number of different species) in each quadrant? 
2. They would also like to know how abundant the genus _Acer_ is (as a % of the total number of trees) in each quadrant.
3. Finally, they would like, _for each quadrant separately_, a bar plot showing counts of _Acer_ trees in the different age classes, ordered so they read from Young (lumping together juvenile and semi-mature trees), Middle Aged, and Mature.



<details>
   <summary markdown= "span"> Click this line to see the solution! </summary>
    <summary markdown= "block">
    
First of all, we need to create the four quadrants. This only requires simple maths and the use of mutate to create a new factor. 

```r
## Calculate the quadrants

# Find the center coordinates that will divide the data (adding half of the range in longitude and latitude to the smallest value)

lon <- (max(trees.genus$Easting) - min(trees.genus$Easting))/2 + min(trees.genus$Easting)
lat <- (max(trees.genus$Northing) - min(trees.genus$Northing))/2 + min(trees.genus$Northing)

# Create the column

trees.genus <- trees.genus %>%
   mutate(Quadrant = case_when(
                     Easting < lon & Northing > lat ~ "NW",
                     Easting < lon & Northing < lat ~ "SW",
                     Easting > lon & Northing > lat ~ "NE",
                     Easting > lon & Northing < lat ~ "SE")
   )

# We can check that it worked
ggplot(trees.genus) +
   geom_point(aes(x = Easting, y = Northing, colour = Quadrant)) +
   theme_bw()
```

<center> <img src="{{ site.baseurl }}/img/DL_data-manip-2_quadrants.png" alt="Img" style="width: 800px;"/> </center>


It did work, but there is a NA value (check the legend)! Probably this point has the exact integer value of middle Easting, and should be attributed to one side or the other (your choice). 


```r
trees.genus <- trees.genus %>%
   mutate(Quadrant = case_when(
      Easting <= lon & Northing > lat ~ "NW",  # using inferior OR EQUAL ensures that no point is forgotten
      Easting <= lon & Northing < lat ~ "SW",
      Easting > lon & Northing > lat ~ "NE",
      Easting > lon & Northing < lat ~ "SE")
   )
```
 
To answer the first question, a simple pipeline combining `group_by()` and `summarise()` is what we need.
   
```r
sp.richness <- trees.genus %>%
group_by(Quadrant) %>%
summarise(richness = length(unique(LatinName)))
```
There we are! We have 7, 15, 8 and 21 species for the NE, NW, SE, and SW corners respectively!

There are different ways to calculate the proportion of _Acer_ trees, here is one (maybe base R would have been less convoluted in this case!):

```r
acer.percent <- trees.genus %>%
   group_by(Quadrant, Genus) %>%
   tally() %>%                      # get the count of trees in each quadrant x genus
   group_by(Quadrant) %>%           # regroup only by quadrant 
   mutate(total = sum(n)) %>%       # sum the total of trees in a new column
   filter(Genus == "Acer") %>%      # keep only acer
   mutate(percent = n/total)        # calculate the proportion

# We can make a plot representing the %

ggplot(acer.percent) +
   geom_col(aes(x = Quadrant, y = percent)) +
   labs(x = "Quadrant", y = "Proportion of Acer") +
   theme_bw()
```

And finally, we can use our manipulation skills to subset the data frame to _Acer_ only and change the age factor, and then use our pipes to create the four plots. 

```r
# Create an Acer-only data frame 

acer <- trees.genus %>% 
   filter(Genus == "Acer")
   

# Rename and reorder age factor

acer$AgeGroup <- factor(acer$AgeGroup,
                        levels = c("Juvenile", "Semi-mature", "Middle Aged", "Mature"),
                        labels = c("Young", "Young", "Middle Aged", "Mature"))


# Plot the graphs for each quadrant

acer.plots <- acer %>%
   group_by(Quadrant) %>%
   do(plots =           # the plotting call within the do function
         ggplot(data = .) +
         geom_bar(aes(x = AgeGroup)) +
         labs(title = paste("Age distribution of Acer in ", .$Quadrant, " corner", sep = ""),
              x = "Age group", y = "Number of trees") +
         theme_bw() +
         theme(panel.grid = element_blank(),
               axis.title = element_text(size = 14),
               axis.text = element_text(size = 14),
               plot.title = element_text(hjust = 0.5))
   ) 

# View the plots (use the arrows on the Plots viewer)
acer.plots$plots
```
<center> <img src="{{ site.baseurl }}/img/DL_data-manip-2_challenge.png" alt="Img" style="width: 800px;"/> </center>

Well done for getting so far!
    
</summary>   
 </details>

<br>
<br>


We hope this was useful. Let's look back at what you can now do, and as always, get in touch if there is more content you would like to see!

### Tutorial Outcomes:

#### 1. You can streamline your code using pipes

#### 2. You know how to reclassify values or recode factors using logical statements

#### 3. You can tweak `dplyr`'s function `group_by()` to act as a loop without having to write one, following it by `do()`

<hr>
<hr>


This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/). <a href="https://creativecommons.org/licenses/by-sa/4.0/"><img src="https://licensebuttons.net/l/by-sa/4.0/80x15.png" alt="Img" style="width: 100px;"/></a>

<h3><a href="https://www.surveymonkey.co.uk/r/9QHFW33" target="_blank">&nbsp; We would love to hear your feedback, please fill out our survey!</a></h3>
<br>
<h3>&nbsp; You can contact us with any questions on <a href="mailto:ourcodingclub@gmail.com?Subject=Tutorial%20question" target = "_top">ourcodingclub@gmail.com</a></h3>
<br>
<h3>&nbsp; Related tutorials:</h3>

{% assign posts_thresh = 8 %}

<ul>
  {% assign related_post_count = 0 %}
  {% for post in site.posts %}
    {% if related_post_count == posts_thresh %}
      {% break %}
    {% endif %}
    {% for tag in post.tags %}
      {% if page.tags contains tag %}
        <li>
            <a href="{{ site.url }}{{ post.url }}">
	    &nbsp; - {{ post.title }}
            </a>
        </li>
        {% assign related_post_count = related_post_count | plus: 1 %}
        {% break %}
      {% endif %}
    {% endfor %}
  {% endfor %}
</ul>
<br>
<h3>&nbsp; Subscribe to our mailing list:</h3>
<div class="container">
	<div class="block">
        <!-- subscribe form start -->
		<div class="form-group">
			<form action="https://getsimpleform.com/messages?form_api_token=de1ba2f2f947822946fb6e835437ec78" method="post">
			<div class="form-group">
				<input type='text' class="form-control" name='Email' placeholder="Email" required/>
			</div>
			<div>
                        	<button class="btn btn-default" type='submit'>Subscribe</button>
                    	</div>
                	</form>
		</div>
	</div>
</div>

<ul class="social-icons">
	<li>
		<h3>
			<a href="https://twitter.com/our_codingclub" target="_blank">&nbsp;Follow our coding adventures on Twitter! <i class="fa fa-twitter"></i></a>
		</h3>
	</li>
</ul>




