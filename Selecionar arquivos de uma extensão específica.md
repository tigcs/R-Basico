### Seleciona apenas arquivos que tenham exatamente a extensão “.tif”
#### Se colocar apenas `pattern = ".tif"` selecionará também outros arquivos tipo “.tif.aux” ou “.tif.xml” etc.
````{r}
    files <- list.files(pattern = "\\.tif$")
