# MGI-search-R-
To query MouseMine and add constraints


Load intermine and list mines.

```R
library(InterMineR)
listMines()
```
Mpuse mine will be selected here
```R
im<-initInterMine(mine = listMines()["MouseMine"])
```
Various queries can be prepared, below phenotype_genotype is being used.
```R
queryPhen = getTemplateQuery(
  im = im, ### mine to search, mousemine in this case
  name = "MPhenotype_MGenotype" ### type of query
)
```

To investigate alternative queries, code below creates a dataframe which alternative queries can be selected.
The above name = "MPhenotype_MGenotype" was reviewing the template below.
```R
template=getTemplates(im)
```

To actually run the prepared query, the mine and query are used as arguments in runQuery
```R
resPhen<-runQuery(im,queryPhen) ## runs query
```




