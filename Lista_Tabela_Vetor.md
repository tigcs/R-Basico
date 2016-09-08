### >>> Como juntar várias tabelas xlsx excel em um único data.frame <<<

#### Retirado de:
http://stackoverflow.com/questions/22394234/loop-for-read-and-merge-multiple-excel-sheets-in-r

````{r}
# This will give you a vector of the names of files in your current directory 
# (where I've assumed the directory contains only the files you want to read)
data.files = list.files()

# Read the first file
df = readWorksheetFromFile(file=data.files[1], sheet=1)

# Loop through the remaining files and merge them to the existing data frame
for (file in data.files[-1]) {
    newFile = readWorksheetFromFile(file=file, sheet=1)
    df = merge(df, newFile, all=TRUE)
}
````
===

### >>> Como remover elementos de uma lista <<<

#### Retirado de:
http://stackoverflow.com/questions/652136/how-can-i-remove-an-element-from-a-list

````{r}
x <- list("a", "b", "c", "d", "e"); # example list 
x[-2]; # without 2nd element 
x[-c(2, 3)]; # without 2nd and 3rd
````
===

### >>> Como remover ou substituir caracteres dos elementos de uma lista <<<

#### Retirado de:
http://stackoverflow.com/questions/19667234/removing-a-group-of-words-from-a-character-vector

````{r}
# Sua lista especies
> especies
  [1] "Aburria jacutinga.xlsx"                              
  [2] "Acrobatornis fonsecai.xlsx"                          
  [3] "Alectrurus tricolor.xlsx" 

# Retirar a extensão “.xlsx”
nomes<-gsub(pattern=".xlsx",replacement="",especies)
> nomes
  [1] "Aburria jacutinga"                               "Acrobatornis fonsecai"                          
  [3] "Alectrurus tricolor"        

# Substituir os espaços por “_”
nomes_<-gsub(pattern=" ",replacement="_",nomes)
> nomes_
  [1] "Aburria_jacutinga"                               "Acrobatornis_fonsecai"                          
  [3] "Alectrurus_tricolor"
````
===

### >>> Como converter vários XSLX em vários CSV usando FOREACH LOOP <<<

#### Detalhes do uso do `foreach` em http://cran.r-project.org/web/packages/foreach/vignettes/foreach.pdf

````{r}
library('XLConnect')
library('foreach')

# Cria uma lista com todos os arquivos em xlsx
especies<-list.files(pattern = "\\.xlsx$")

# Eliminar algum arquivo indesajado que esteja na pasta
especies<-especies[-c(1:41)]

## Cria um vetor que contenha os nomes desejados para serem os nomes dos arquivos finais.
# Retira a extensão ".xlsx"
nomes<-gsub(pattern=".xlsx",replacement="",especies)

# Substitui todo espaço por "_"
nomes_<-gsub(pattern=" ",replacement="_",nomes)

# Retira todo "." Atenção se quiser substituir pontos ".", estes devem estar entre colchetes assim: "[.]"
nomes_<-gsub(pattern="[.]",replacement="",nomes_) 

# Cria um csv com a lista dos nomes dos arquivos.(Opcional)
write.csv(nomes_,file="./csv/_lista_especies.csv", quote=FALSE)

## Usando FOREACH LOOP para converter cada arquivo xlsx em um csv que terão como nome o elementos da 
# lista criada acima.

foreach(file = especies, j = nomes_) %do% {
  xlsx_objeto<- readWorksheetFromFile(file=file, sheet=1, keep=c(2,3,4,6,7,8,9,10,13,18))
  write.csv(xlsx_objeto,file=paste0("./csv/",j,".csv"))
}
````
===

### >>>
