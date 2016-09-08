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

===

### >>> Seleciona apenas arquivos que tenham exatamente a extensão “.tif” <<<
#### Se colocar apenas `pattern = ".tif"` selecionará também outros arquivos tipo “.tif.aux” ou “.tif.xml” etc.
````{r}
# Comando para listar APENAS os arquivos que contenham ".tif"
    files <- list.files(pattern = "\\.tif$")
````

===

### >>> Remover, deletar, apagar, copiar, renomear arquivos usando R <<<

````{r}

file.create(..., showWarnings = TRUE)
file.exists(...)
file.remove(...)
file.rename(from, to)
file.append(file1, file2)
file.copy(from, to, overwrite = recursive, recursive = FALSE, copy.mode = TRUE)
file.symlink(from, to)
file.link(from, to)

# lista os arquivos
arquivos <- list.files(path="sp_separadas2", full.names = T)

# Loop renomeia os arquivos
for (a in arquivos) {
  
  a_base <- basename(a)
  
  a_base_SEM <- gsub(pattern = "todas_nome3__",replacement = "",a_base)
  
  file.copy(from = a, to = paste0("renomeados/",a_base_SEM), recursive = FALSE, copy.mode = TRUE)
  }
````

===




