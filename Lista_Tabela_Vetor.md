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
