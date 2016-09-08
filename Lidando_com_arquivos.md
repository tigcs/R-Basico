### >>> Remover todo caminho (path) deixando apenas o nome do arquivo <<<

````{r}
# Comando para listar os arquivos dentro do diretório
    files <- list.files(pattern ="\\.shp$",full.names=TRUE,recursive=TRUE,include.dirs=TRUE)

# Resultado
    "./Especies/Xolmis_dominicanus.shp" 

# Comando para retirar o path
      basename(files)

# Resultado
    "Xolmis_dominicanus.shp"
````

=

### >>> Seleciona apenas arquivos que tenham exatamente a extensão “.tif” <<<
#### Se colocar apenas `pattern = ".tif"` selecionará também outros arquivos tipo “.tif.aux” ou “.tif.xml” etc.
````{r}
# Comando para listar APENAS os arquivos que contenham ".tif"
    files <- list.files(pattern = "\\.tif$")
````

