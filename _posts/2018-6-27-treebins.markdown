---
layout: post
comments: true
title:  "tree.bins 0.1.1"
subtitle: "Recategorization by Decision Tree Method"
date:   2018-06-27 12:05:19 -0500
authors: piro_polo
tags: R
header-img: "img/tree.bins/tree.jpg"
permalink: /:title
---

## Overview
I created `tree.bins` to provide users the ability to recategorize categorical variables, dependent on a response variable, by iteratively creating a decision tree for each of the categorical variables (class factor) and the selected response variable. The decision tree is created using `rpart::rpart`. You often encounter variables with several levels when conducting data analysis or using machine learning algorithms. Decision trees can be used to decide how to best collapse these categorical variables into more manageable factors.  The rules from the leaves of the decision tree are extracted, and used to recategorize (bin) the appropriate categorical variable (predictor). Only variables containing more than two factor levels will be considered by the function. The final output generates a dataset containing the recategorized variables and/or a list containing a mapping table for each of the candidate variables. For more details see [Decision tree methods: applications for classification and prediction](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4466856/) by Yan-yan Song and Ying Lu  or T. Hastie et al (2009, ISBN: 978-0-387-84857-0).

## Introduction
At FI Consulting, the Data Science Book Club explores cutting-edge topics over a Friday beer. We occasionally challenge each other to a friendly competition practicing new predictive and machine learning models. Last meeting's challenge used the Ames housing dataset to predict housing prices in Ames, Iowa. It contained a large number of categorical and continuous variables. Altogether, the dataset consisted of over 70 variables. To truly understand the relationship among the variables and the response, in particular the categorical variables, I needed to visualize their relationship with the average house price of homes sold in Ames, Iowa. My goal was to recategorize the current levels into bins that displayed similar relationship with the sale price. Performing such a task manually would have taken extensive effort.

When working with large datasets, there may be a need to recategorize candidate variables by some criterion. `tree.bins` allows you to recategorize these variables through a decision tree method derived from `rpart::rpart`. `tree.bins` is especially useful if the dataset contains several factor class variables with an abnormal number of levels. The intended purpose of the package is to recategorize predictors in order to reduce the number of dummy variables created when applying a statistical method to model a response. This can result in more parsimonious and/or accurate modeling results. The first half of this post illustrates data analysis procedures to identify a typical problem that contains a variable with several levels, and the latter half covers `tree.bins` functionality and usage.

## Pre-Categorization: Typical Variable for Consideration
This section illustrates a typical variable that could be considered for recategorization.

### Visualization of Candidate Variable
Using a subset of the Ames dataset, the below chunk illustrates the average home sale price of each Neighborhood.

{% highlight r %}
AmesSubset %>%
  select(SalePrice, Neighborhood) %>%
  group_by(Neighborhood) %>%
  summarise(AvgPrice = mean(SalePrice)/1000) %>%
  ggplot(aes(x = reorder(Neighborhood, -AvgPrice), y = AvgPrice)) +
  geom_bar(stat = "identity", fill = "#389135") +
  labs(x = "Neighborhoods", y = "Avg Price (in thousands)",
       title = paste0("Average Home Prices of Neighborhoods")) +
  theme_economist() +
  theme(legend.position = "none",
	axis.text.x = element_text(angle = 90, hjust = 1, size = 8),
        axis.title.x = element_text(size = 12), axis.text.y = element_text(size = 9),
        axis.title.y = element_text(size = 12))
{% endhighlight %}

![AmesAvgPrices]({{site.baseurl}}/img/tree.bins/AmesAvgPrices.png){:class="img-responsive"}

Notice that many neighborhoods observe the same average sale price. This indicates that we could combine and recategorize the Neighborhoods variable into fewer levels.

### Statistical Method Implementation of Candidate Variable
The following illustrates the results of using a statistical learning method without using the `tree.bins` function – linear regression for this example – with the Neighborhoods categorical variable.

{% highlight r %}
fit <- lm(formula = SalePrice ~ Neighborhood, data = AmesSubset)
summary(fit)
{% endhighlight %}

![LM_Stats]({{ site.baseurl}}/img/tree.bins/LM_Stats.png){:class="img-responsive"}

Notice that there are multiple dummy variables being created to capture the different levels found within the Neighborhoods variable.

### Visualizing the Leaves Created by a Decision Tree
The below steps illustrate how `rpart` categorizes the different levels of Neighborhoods into separate leaves. These leaves are used to generate the mappings that are extracted and applied within `tree.bins` to recategorize the current data.

{% highlight r %}
d.tree = rpart(formula = SalePrice ~ Neighborhood, data = AmesSubset)
rpart.plot::rpart.plot(d.tree)
{% endhighlight %}

![rpartplot]({{ site.baseurl}}/img/tree.bins/rpartplot.png){:class="img-responsive"}

These 5 categories are what `tree.bins` will use to recategorize the variable Neighborhood.

## Post-Categorization: Typical Variable for Consideration
This section illustrates the result of using `tree.bins` to recategorize a typical variable.

### Recategorization of Candidate Variable
There are many neighborhoods with similar average home sale prices. To create fewer dummy variables, we could group the neighborhoods with similar sale prices into one bin. We could alternatively create visualizations to identify these similarities in levels for each variable, but it would remain an extremely tedious task not to mention subjective to the analyst.

A better method would be to use the rules that are generated from a decision tree. Using `rpart::rpart`, the task remains tedious, especially when there are numerous factor class variables to be considered. The `tree.bins` function automates this process by iteratively recategorizing each factor level variable for a given dataset.

{% highlight r %}
sample.df <- AmesSubset %>% select(Neighborhood, MS.Zoning, SalePrice)
binned.df <- tree.bins(data = sample.df, y = SalePrice,
		       bin.nm = "bin#.", return = "new.fctrs")
levels(sample.df$Neighborhood) #current levels of Neighborhood
unique(binned.df$Neighborhood) #new levels of Neighborhood
{% endhighlight %}

|        Old Levels ||
---------|---------|---------|---------|---------|
 Blmngtn | Blueste | BrDale  | BrkSide | ClearCr |
 CollgCr | Crawfor | Edwards | Gilbert | Greens  |
 GrnHill | IDOTRR  | Landmrk | MeadowV | Mitchel |
 OldTown | Sawyer  | SawyerW | Somerst | StoneBr |
 SWISU   | Timber  | Veenker |         |         |

| New Levels |||||
|--------|---------|--------|--------|--------|
| bin#.4 | bin#.3  | bin#.5 | bin#.2 | bin#.1 |

The control parameter in the `tree.bins` function serves the same purpose as the control parameter in the `rpart` function. If you specify a value for this parameter, that value will be used to prune the tree for each variable passed in to the data parameter. Remember, that a decision tree is being built to refactor each variable into new levels.

{% highlight r %}
binned.df2 <- tree.bins(data = sample.df, y = SalePrice,
		       bin.nm = "bin#.", control = rpart.control(cp = .001),
	               return = "new.fctrs") #unique(binned.df2$Neighborhood)
		       #displays new levels of Neighborhood
{% endhighlight %}

You can also create a two-dimensinal `data.frame` and pass this object into the control parameter. The first column must contain the variable name(s) that are contained in the `data.frame` specified in the data parameter. The second column must contain the cp values of the respective variable name(s). Any variable name(s) not included in your `data.frame` will use the generated cp value within the `rpart` function. Lastly, the column names identified in your created `data.frame` are irrelevant, only the elements are important.

{% highlight r %}
cp.df <- data.frame(Variables = c("Neighborhood", "MS.Zoning"), CP = c(.001, .1))
binned.df3 <- tree.bins(data = sample.df, y = SalePrice, bin.nm = "bin#.",
		        control = cp.df, return = "new.fctrs")
unique(binned.df3$Neighborhood) #new levels of Neighborhood
unique(binned.df3$MS.Zoning) #new levels of MS.Zoning
{% endhighlight %}

| New Levels | Neighborhood | MS.Zoning |
|		1	 |bin#.7		| bin#.1 	|
|		2	 |bin#.3		| bin#.2 	|
|		3	 |bin#.9		|       	|
|		4	 |bin#.4		|       	|
|		5	 |bin#.1		|       	|
|		6	 |bin#.2		|       	|
|		7	 |bin#.8		|       	|
|		8	 |bin#.5		|       	|
|		9	 |bin#.6		|       	|
|	    10	 |bin#.10		|       	|


## The Different Return Options of tree.bins
Depending on what you want, `tree.bins` can return either the recategorized data.frame or a list of lookup tables. The lookup tables contain the old-to-new value mappings for each recategorized variable generated by `tree.bins`.

"new.fctrs" returns the recategorized data.frame.

{% highlight r %}
head(
  tree.bins(data = sample.df, y = SalePrice, bin.nm = "bin#.",
		        control = cp.df, return = "new.fctrs"
  )
)
{% endhighlight %}

| SalePrice | Neighborhood | MS.Zoning |
|-----------|--------------|-----------|
| 105000    | bin#.4       | bin#.1    |
| 244000    | bin#.4       | bin#.2    |
| 189900    | bin#.3       | bin#.2    |
| 195500    | bin#.3       | bin#.2    |
| 191500    | bin#.5       | bin#.2    |
| 236500    | bin#.5       | bin#.2    |

"lkup.list" returns a list of the lookup tables.

{% highlight r %}
head(
  tree.bins(data = sample.df, y = SalePrice, bin.nm = "bin#.",
 		         control = rpart.control(cp = .01), return = "lkup.list"
  )[[1]] # displaying the first object in the list
)
{% endhighlight %}

| Neighborhood | Categories |
|--------------|------------|
| BrDale       | bin#.1     |
| BrkSide      | bin#.1     |
| IDOTRR       | bin#.1     |
| MeadowV      | bin#.1     |
| OldTown      | bin#.1     |
| Somerst      | bin#.2     |

"both" returns an object containing both the `new.fctrs` and `lkup.list` outputs. These can be returned by using the "$" notation.

{% highlight r %}
both <- tree.bins(data = sample.df, y = SalePrice, bin.nm = "bin#.",
 		  control = rpart.control(cp = .01), return = "both")
head(both$new.fctrs)
head(both$lkup.list[2])
{% endhighlight %}

| SalePrice | Neighborhood | MS.Zoning |
|-----------|--------------|-----------|
| 105000    | bin#.4       | bin#.1    |
| 244000    | bin#.4       | bin#.2    |
| 189900    | bin#.3       | bin#.2    |
| 195500    | bin#.3       | bin#.2    |
| 191500    | bin#.5       | bin#.2    |
| 236500    | bin#.5       | bin#.2    |

| MS.Zoning | Categories |
|-----------|------------|
| A (agr)   | bin#.1     |
| C (all)   | bin#.1     |
| I (all)   | bin#.1     |
| RH        | bin#.1     |
| RM        | bin#.1     |
| FV        | bin#.2     |
| RL        | bin#.2     |

## Using the bin.oth Function

`tree.bins` recategorizes factor class variables of any data.frame. Assuming that similar data will continue to be collected, or perhaps used in testing the performance of the model, you may want to recategorize this new data.frame by the same lookup tables that were generated from the first data.frame. In this case, being able to bin other data.frames with the same lookup table would be quite useful. The example below takes in a subset of the `AmesSubset` data and returns a data.frame recategorized by the lookup list generated from the `tree.bins` function.

{% highlight r %}
head(bin.oth(list = lookup.list, data = sample.df))
{% endhighlight %}

| SalePrice | Neighborhood | MS.Zoning |
|-----------|--------------|-----------|
| 105000    | bin#.4       | bin#.1    |
| 244000    | bin#.4       | bin#.2    |
| 189900    | bin#.3       | bin#.2    |
| 195500    | bin#.3       | bin#.2    |
| 191500    | bin#.5       | bin#.2    |
| 236500    | bin#.5       | bin#.2    |

## Closing Remarks
I hope `tree.bins` will help you solve similar problems. Please feel free to post any issues that you encounter or simply keep up with the latest changes and updates on [GitHub](https://github.com/pikos90/tree.bins).
