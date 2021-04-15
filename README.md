# MGI-search-R-
To query MouseMine and add constraints


Load intermine and list mines.

```R
library(InterMineR)
listMines()
```
Mouse mine will be selected here
```R
im<-initInterMine(mine = listMines()["MouseMine"])
```

Get template. Use mousemine if looking for mammalian phenotypes becuae they are candidate animals.
Also views the mammalian templates
```R
template=getTemplates(im)
## want to search templates for: phenotypes
template[grep("mammalian", template$title, ignore.case=TRUE),] ## search title column

```
Various queries can be prepared, below phenotype_genotype is being used as a query.
```R
queryPhen = getTemplateQuery(
  im = im, ### mine to search, mousemine in this case
  name = "MPhenotype_MGenotype" ### type of query
)
```

To investigate alternative queries, code below creates a dataframe which alternative queries can be selected.
The above name = "MPhenotype_MGenotype" was found by reviewing the template below.
```R
template=getTemplates(im)
```

To actually run the prepared query, the mine and query are used as arguments in runQuery
```R
resPhen<-runQuery(im,queryPhen) ## runs query
```

To add a constraint to my query, mouse phenotype/genotype, arguments will be passed to setConstraints index 5.
resPhen from above has a default constraint in index 5 we are not concerned about therefore this will be overwritted
by 'Perinatal lethality', the phenotype of interest. This can be verified by inspecting running query phen before and
after this constraint modification, position 5 changes from default to 'perinatal lethality'.

```R
queryPhen$where = setConstraints(
  modifyQueryConstraints = queryPhen,
  m.index = 5,
  values = list("perinatal lethality")
)
resPhen1<-runQuery(im,queryPhen) ## runs query
```
An inspection of ontology term in resPhen and resPhen1 demonstrated a change in query.

Additional terms can be added to the search by filling in the next index i.e. 6 in this case. Below, zygosity was
selected which will now return perinatal lethality when homozygous.

```R
newConstraint<-list(
  path=c("OntologyAnnotation.evidence.baseAnnotations.subject.zygosity"), ## path where constraint put on
  op=c("="), ## operation
  value=c("hm"), ## what value filtering on
  code=c("B") ## use a code not in use
)

queryPhen$where[[6]]<-newConstraint ## added constraint to query, used 6 which was next free index

resPhen<-runQuery(im,queryPhen) ## runs query
```

Now constraints have been added, additional information available in MGI might be desired and can be included in the query
to be requested. Below, supporting evidence is requested by way of publications. Below, the previous query is inherited
whilst requesting (selecting) new information. get select is used which return the data to be retrieved.


```R
model<- getModel(im) # Retrieve base model (mousemine)

## Adding publication data to the query for it to be returned
queryPhen.IntermineR = setQuery(
  inheritQuery = queryPhen,
  select = c(queryPhen$select,
             "OntologyAnnotation.evidence.publications.year",
             "OntologyAnnotation.evidence.publications.citation")
)

### running the line below shows the columns to be returned from a query
getSelect(queryPhen.IntermineR) ## to check the newly created and added columns

res.queryPhenIntermineR <-runQuery(im, queryPhen.IntermineR) ### run query with new requests included
```

