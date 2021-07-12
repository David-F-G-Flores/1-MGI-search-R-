# MGI-search-R-
To query the MouseMine for mouse genes associated with phenotype ```prenatal lethality``` by adding constraints to the query.

Load intermine and list all available mines.
```R
library(InterMineR)
listMines()
```
Mouse mine was selected here. Alternative model organisms can be selected via searching the above mentioned ```listMines()```.
```R
im<-initInterMine(mine = listMines()["MouseMine"])
```
To retrieve all template queries. 
Searching through templates for mammalian.
```R
template=getTemplates(im)
## want to search templates for: phenotypes
template[grep("mammalian", template$title, ignore.case=TRUE),] ## search title column
```
Various queries can be prepared, below ```"MPhenotype_MGenotype"``` is being queried in the mousemine, obtained from above search.
```R
queryPhen = getTemplateQuery(
  im = im, ### mine to search, mousemine in this case
  name = "MPhenotype_MGenotype" ### type of query
)
```
To run the prepared query.
```R
resPhen<-runQuery(im,queryPhen) ## runs query
```
To add a constraint, arguments will be set in ```setConstraints``` at ```m.index 5```.
```resPhen``` from above had a default constraint in index 5, "*circulating glucose*", which can be verified by calling queryPhen$where[5] and to confirm this is the last populated index, queryPhen$where[6] can be called to confirm empty, this can be used for more complex queries. We are not concerned with "*circulating glucose*"  therefore this was be populated with ```"prenatal lethality"```, the phenotype of interest. This can be verified by inspecting queryPhen$where[5].
```R
queryPhen$where = setConstraints(
  modifyQueryConstraints = queryPhen,
  m.index = 5,
  values = list("prenatal lethality")
)
resPhen1<-runQuery(im,queryPhen) ## runs query
```
Alternative constraints can be added. The example above ```"prenatal lethality"``` was identified by viewing the phenotype tree at http://www.informatics.jax.org/vocab/mp_ontology/ 

Additional constraints can be prepared and included in the search by populating the next index i.e. 6 in this case. 
Below, a constraint on zygosity was made by preparing a new constraint, and inserting this into 6th index. This will return prenatal lethality when homozygous "hm". If heterozygous was desired, "ht" could be used. 
This newConstraint was added to the 6th index, and ```runQuery``` was used. 
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
Now constraints have been added. Additional information available in MGI but not in the ```resPhen``` object can be included in the query. Below, supporting evidence is requested i.e. publications. The previous query was inherited and new information, "...publications.year" and "...publications.citation" saved into ```queryPhen``` object.
```R

## Adding publication data to the query for it to be returned
queryPhen.IntermineR = setQuery(
  inheritQuery = queryPhen,
  select = c(queryPhen$select,
             "OntologyAnnotation.evidence.publications.year",
             "OntologyAnnotation.evidence.publications.citation")
)

### running the line below shows the columns which will return in a query before running
getSelect(queryPhen.IntermineR) ## to check the newly created and added columns

mousegenes <-runQuery(im, queryPhen.IntermineR) ### run query with new requests included
```

