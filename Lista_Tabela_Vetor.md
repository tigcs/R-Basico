### >>> Juntar várias tabelas CSV em um único data.frame <<<

````{r}

# Lista todos os aqruivos .csv da pasta chamda csv
tabelas_csv<- list.files(path="./csv",pattern ="\\.csv")

# Eliminar um arquivo indesejado
tabelas_csv<- tabelas_csv[-c(1,2,3)]

#Ler a primeira tabela da lista
sp1<- read.csv(file=paste0("./csv/",tabelas_csv[1]))

# Foreach Loop com rbind adicionando uma tabela por vez ao sp1.
foreach(i=tabelas_csv[-1]) %do% {
  spi<- read.csv(file=paste0("./csv/",i))
  sp1<- rbind(sp1,spi)
}

# Escreve a tabela final
write.csv(sp1,file="./csv/_sp_TODAS.csv")
````

#### Detalhes do uso do `foreach` em http://cran.r-project.org/web/packages/foreach/vignettes/foreach.pdf

===

### >>> Criar uma tabela vazia e adicionar linhas subsequentes <<<

````{r}
# Cria uma tabela com três colunas, a primeira de caracteres, e outras duas de números.
tabela <- data.frame(character(0),numeric(0),numeric(0))

# Nomeia as colunas das tabela vazia
names(tabela) <- c("tx", "minValue","maxValue")

for (rst in raster_files) {
  
  # Carrega o raster da especie
  r <- raster(rst)
  
  # Formata o nome da especie
  nome_sp <- basename(rst)
  nome_sp <- gsub(nome_sp, pattern = "\\.tif$",replacement = "")
  
  # Cria uma tabela com o nome da especie, minimo valor e maximo valor do raster
  tabela_r <- data.frame(tx= nome_sp, minValue= minValue(r), maxValue= maxValue(r))
  
  # Adiciona a tabela vazia a tabela criada na linha anterior
  tabela <- rbind(tabela,tabela_r)
  
  cat(nome_sp,"\n")
} 
````

### >>> Juntar várias tabelas xlsx excel em um único data.frame <<<

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

### >>> Como remover elementos de uma lista ou vetor<<<

#### Retirado de:
http://stackoverflow.com/questions/652136/how-can-i-remove-an-element-from-a-list

````{r}
x <- list("a", "b", "c", "d", "e"); # example list 
x[-2]; # without 2nd element 
x[-c(2, 3)]; # without 2nd and 3rd
````
===
### >>> Como selecionar itens de uma lista com um padrão (pattern) <<<

````{r}
# Lista 
   final_models_path_CAN <- list.files(path = "./2_final_model/CAN",pattern="\\.tif$",full.names = T)
   final_models_path_HAD <- list.files(path = "./2_final_model/HAD",pattern="\\.tif$",full.names = T)
   final_models_path <-c(final_models_path_CAN, final_models_path_HAD)

# Seleciona itens da lista que tem como padrão o “sp” usando a função grep, que dará as posições que 
se encontram os elementos que tenham o padrão “sp”.
    final_models_path[grep(pattern=sp, final_models_path)]
````
===

### >>> Busca dentro de uma lista (fichas) o termo (sp), e retorna o elemento da lista que contém o termo <<<
````{r}
grep(sp,fichas,value=T)
````
===

### >>> Seleciona a string que está entre dois caracteres, no caso duas barras "/", dentro do elemento “ficha_ori” <<<
````{r}
> ficha_ori
[1] "./_fichas/Fringilidae/Sporagra yarrellii.doc"
familia <- sub(".*/(.*)/.*","\\1",ficha_ori)
familia
> Fringilidae
````

===
### >>> Como remover ou substituir caracteres dos elementos de uma lista ou vetor<<<

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

### >>> Evitar row.names após fazer o subset de um data.frame <<<
````{r}
# Usar a função data.frame junto com subset e colocar o argumento row.names = NULL
pe_ja <- data.frame(subset(cemave,sp=="pe_ja",select=-taxon),row.names = NULL)
````

===

### >>> Join (merge) de tabelas pelos valores de colunas correspondentes <<<

````{r}
## Join entre tabela de registro de pontos e planilha mãe

library(XLConnect)

# Carregando tabela de registro de ocorrência
registros<- readWorksheetFromFile(file="./registros_ocorrencia_refine.xlsx", sheet=1)

# Carregando planilha mãe
pl<-read.csv("planilha_mae_utf8.csv",header=TRUE,sep=";")

# Join das tabelas pelo valor do campo "taxon". O campo taxon é em comum nas duas tabelas
join<-merge(registros,pl,by.x="taxon",by.y="taxon",all.x=TRUE)
````

===

### >>> Listar arquivos que tenham um string específica

https://stackoverflow.com/questions/30156904/using-pattern-to-select-files-that-has-xx-at-any-part-the-name-in-r/30157003#30157003

#### **Quetion:**
I have a folder full of files which names are like these, for example:
````{r}
"./final_model_pre/pre_pe_ja_bc_wm.tif" 
"./final_model_pre/pre_pe_ja_bc_wm.tif.aux.xml" 
"./final_model_pre/pre_pe_ja_bc_wm.tif.ovr" 
"./final_model_pre/pre_an_le_glm_wm.tif" 
"./final_model_pre/pre_an_le_glm_wm.tif.aux.xml" 
"./final_model_pre/pre_an_le_glm_wm.tif.ovr" 
"./final_model_pre/pre_an_bo_ma_wm.tif" 
"./final_model_pre/pre_an_bo_ma_wm.tif.aux.xml" 
"./final_model_pre/pre_an_bo_ma_wm.tif.ovr" 
"./final_model_pre/pre_pe_ja_mx_wm.tif" 
"./final_model_pre/pre_pe_ja_mx_wm.tif.aux.xml"
"./final_model_pre/pre_pe_ja_mx_wm.tif.ovr" 
"./final_model_pre/pre_pe_ja_rf1_wm.tif"
"./final_model_pre/pre_pe_ja_rf1_wm.tif.aux.xml"
"./final_model_pre/pre_pe_ja_rf1_wm.tif.ovr" 
"./final_model_pre/pre_pe_ja_svm_wm.tif"
"./final_model_pre/pre_pe_ja_svm_wm.tif.aux.xml"
"./final_model_pre/pre_pe_ja_svm_wm.tif.ovr"
````
I want to list every files that have "pe_ja" in the name and only with ".tif" extension, not ".tif.ovr" or ".tif.aux.xml" or any other extension. I'm trying to use list.files function, but I couldn't manage to use the pattern agrument properly. Could you help me doing that? 

#### **Answer:**
You can use a regular expression for that.
`files = list.files(pattern = '.*pe_ja.*\\.tif$')`
The `$` at the end of the regular expression indicates that that is the end of the string. The `\\.` is an escaped period, indicating that you want to match a period (not any character, which is what .normally matches).
The `.*` selects any character any number of times (including 0).

===

### >>> Primeira letra Maiuscula <<<

````{r}
simpleCap <- function(x) {
  s <- strsplit(x, " ")[[1]]
  paste(toupper(substring(s, 1, 1)), substring(s, 2),
        sep = "", collapse = " ") }
````
===
### >>> Sumarize, Count ou Frequência <<<

#### Calcular a frequência que um elemento aparece em um vetor
#### Função `table` associada à função `as.data.frame`:
````{r}
a <- c(1,1,1,1,2,2,2,3,3,4,4,4,4,4,4,4)
table(a)
a
1 2 3 4 
4 3 2 7 
 as.data.frame(table(a))
  a Freq
1 1    4
2 2    3
3 3    2
4 4    7
````
#### Ou usar a função `count`do pacote `plyr`:
````{r}
library(plyr)

> count(tab_num,vars ="ID_CEL")

  ID_CEL freq
1      1    4
2      2    4
3      3    4
````
===
### >>> Confere se elementos de um conujnto estão contidos em um outro <<<

````{r}
setwd("D:/2016/vulnerabilidade_uc/_R1")

# Lista de arquivos iniciais
especies <- list.files("./novos3",pattern="\\.shp$", full.names = TRUE)
especies <- basename(especies)
especies <- gsub(especies, pattern = "\\.shp$", replacement = "")


# Lista de arquivos finais
rasters <- list.files("./raster",pattern = "\\.tif$")
rasters <- basename(rasters)
rasters <- gsub(rasters,pattern = "\\.tif$",replacement = "")

# Cria tabela para receber o resultado da verificacao
tab_raster <- data.frame(row.names=NULL)

# Loop de conferencia
for ( sp in especies) {
  
  raster_sp <- sp %in% rasters
  tab_raster_sp <- data.frame (sp, raster_sp)
  tab_raster <- rbind(tab_raster,tab_raster_sp)
  cat (sp,raster_sp,"\n")
}

# Renomeia as colunas
names(tab_raster) <- c("taxon","raster")

# Especies sem raster
tab_sem_raster <- subset.data.frame(tab_raster,raster==FALSE)
````
===
### >>> Remover linhas duplicadas baseando-se em um única coluna <<<
````{r}
# Remove as linhas duplicadas baseando-se na 1ª coluna de tab_ucs
tab_ucs <- tab_ucs[!duplicated(tab_ucs[,1]),]
````
===
### >>> Como remover parênteses ( ) de uma string <<<
#### A princípio usando o caractere especial dentro de colchetes [ ] funciona com qualquer caractere.

````{r}
# Nome especie
nome_sp <- gsub(sp_shp$nome_cient,pattern = " ", replacement = "_" )
nome_sp <- gsub(nome_sp,pattern = "-", replacement = "_" )
nome_sp <- gsub(nome_sp,pattern = '[(]', replacement = "" )
nome_sp <- gsub(nome_sp,pattern = ")", replacement = "" )
````
===

### >>> Média e desvio padrão por colunas <<<

#### Retirado de : http://stackoverflow.com/questions/20794284/means-and-sd-for-columns-in-a-dataframe-with-na-values#
#### Explicação do funcionamento de apply,sapply, tapply:
http://www.dadosaleatorios.com.br/2014/01/apply-lapply-sapply-tapply-mapply-como.html
http://tech.queryhome.com/76799/r-difference-between-apply-vs-sapply-vs-lapply-vs-tapply

````[r}
sapply(df, function(cl) list(means=mean(cl,na.rm=TRUE), sds=sd(cl,na.rm=TRUE)))
      col1     col2     col3     col4     col5    
means 3        8        12.5     18.25    22.5    
sds   1.581139 1.581139 1.290994 1.707825 1.290994

as.data.frame( t(sapply(df, function(cl) list(means=mean(cl,na.rm=TRUE), 
                                              sds=sd(cl,na.rm=TRUE))) ))
     means      sds
col1     3 1.581139
col2     8 1.581139
col3  12.5 1.290994
col4 18.25 1.707825
col5  22.5 1.290994
````
===
### >>> Arrendondar números de uma tabela <<<
````{r}
is.num <- sapply(DF, is.numeric)
DF[is.num] <- lapply(DF[is.num], round, 8)
````
===
### >>> Transformar tipo de coluna <<<

````{r}
> d
  char fake_char fac char_fac num
1    a         1   1        a   1
2    b         2   2        b   2
3    c         3   3        c   3
4    d         4   4        d   4
5    e         5   5        e   5

> transform(d, fake_char = as.numeric(fake_char), 
               char_fac = as.numeric(char_fac))

  char fake_char fac char_fac num
1    a         1   1        1   1
2    b         2   2        2   2
3    c         3   3        3   3
4    d         4   4        4   4
5    e         5   5        5   5
````
===
### >>> Ordenar Tabela por uma coluna <<<
````{r}
 # Ordena a tabela por ordem alfabetica pela coluna variaveis
 var_biomas <- var_biomas[order(var_biomas$variaveis),]
 ````
 ===
### >>> Trasnforma a tabela de fator ou caractere em numérica usando sapply <<<
````{r}
#
tab_p <- as.data.frame(sapply(tab_p, FUN=as.numeric))
````
