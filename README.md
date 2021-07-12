# MGI-search-R-
To query MouseMine for mouse genes associated with phenotype ```prenatal lethality```  and add constraints


Load intermine and list mines.

```R
library(InterMineR)
listMines()
```
Mouse mine will be selected here. Alternative model organisms can be selected via searching listMines().
```R
im<-initInterMine(mine = listMines()["MouseMine"])
```
Get all template queries. 
Searching through templates for mammalian.
```R
template=getTemplates(im)
## want to search templates for: phenotypes
template[grep("mammalian", template$title, ignore.case=TRUE),] ## search title column
```
Various queries can be prepared, below "MPhenotype_MGenotype" is being used as a query in the mousemine, obtained from above search.
```R
queryPhen = getTemplateQuery(
  im = im, ### mine to search, mousemine in this case
  name = "MPhenotype_MGenotype" ### type of query
)
```
To actually run the prepared query.
```R
resPhen<-runQuery(im,queryPhen) ## runs query
```
To add a constraint to my prepared query, arguments will be passed to setConstraints index 5.
resPhen from above has a default constraint in index 5, "*circulating glucose*", which can be verified by calling queryPhen$where[5] and to confirm this is the last filled index, queryPhen$where[6] can be called to confirm this. We are not concerned about "*circulating glucose*"  therefore this will be overwritten by 'prenatal lethality', the phenotype of interest. This can be verified by inspecting queryPhen$where[5].
```R
queryPhen$where = setConstraints(
  modifyQueryConstraints = queryPhen,
  m.index = 5,
  values = list("prenatal lethality")
)
resPhen1<-runQuery(im,queryPhen) ## runs query
```
Alternative constraints can be added. The above example 'prenatal lethality' was identified by viewing the phenotype tree at http://www.informatics.jax.org/vocab/mp_ontology/ 

Additional terms can be added to the search by filling in the next index i.e. 6 in this case. 
Below, a constraint on zygosity was made by preparing a new constraint, inserting this into 6th index. This will now return prenatal lethality when homozygous "hm". If heterozygous was desired, "ht" could be used. 
This newConstraint was added to the 6th index, and runQuery which will return 
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
to be requested. Below, supporting evidence is requested by way of publications. The previous query is inherited
whilst requesting (selecting) new information, "...publications.year" and "...publications.citation".
```R

## Adding publication data to the query for it to be returned
queryPhen.IntermineR = setQuery(
  inheritQuery = queryPhen,
  select = c(queryPhen$select,
             "OntologyAnnotation.evidence.publications.year",
             "OntologyAnnotation.evidence.publications.citation")
)

### running the line below shows the columns to be returned from a query
getSelect(queryPhen.IntermineR) ## to check the newly created and added columns

mousegenes <-runQuery(im, queryPhen.IntermineR) ### run query with new requests included
```

