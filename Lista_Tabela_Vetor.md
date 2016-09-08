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
````{r}
===

### >>>
