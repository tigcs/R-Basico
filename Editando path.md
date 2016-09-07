###Remover todo caminho (path) deixando apenas o nome do último arquivo

####Comando para listar os arquivos dentro do diretório
````{r}
    files<-list.files(pattern ="\\.shp$",full.names=TRUE,recursive=TRUE,include.dirs=TRUE)
````

####Resultado
````{r}
    "./Xolmis dominicanus/Xolmis_dominicanus.shp" 
```{r}

####Comando para retirar o path
 ````{r}   
    basename(files)

####Resultado
    "Xolmis_dominicanus.shp"
````{r}
