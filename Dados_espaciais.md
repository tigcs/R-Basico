### >>> Criar um grid <<<
#### Nunca testei.
http://rfunctions.blogspot.com.br/2014/12/how-to-create-grid-and-intersect-it.html

===

### >>> Ler e escrever um shapefile <<<


#### Pacote `raster`
https://cran.r-project.org/web/packages/raster/raster.pdf
##### Costuma ser mais lento do que o pacote `maptools`

````{r}
# Ler shapefile
biomas<- shapefile ("./biomas/editado/BIOMAS.shp")

# Escreve shapefile
shapefile(x=intersection, filename="./teste_clip.shp")
````

#### Pacote `dismo`
http://www.inside-r.org/packages/cran/dismo/docs/shapefile

````{r}
# filename => Character. Full filename of a ESRI shapefile
shapefile(filename)
````

#### Pacote `maptools`
##### Há funções diferentes para cada tipo de feição (pontos, linhas e polígonos)
http://cran.r-project.org/web/packages/maptools/maptools.pdf
https://www.nceas.ucsb.edu/scicomp/usecases/ReadWriteESRIShapeFiles
````{r}
# Ler shapefile
sp_biomas <- readShapePoly(fn= "./sp_biomasEdit.shp",proj4string=CRS("+proj=longlat +datum=WGS84")) 

# Escreve um shapefile com e sem prj
writePolyShape(sp_biomas,"sp_biomas_test")  
````

===

### >>> Cortar um raster <<<

````{r}
### Cortando variáveis pela extensão da América do Sul

library(maptools)
library(raster)
library(dismo)

## Carregando o shapefile da América do Sul (obs: foram retiradas as ilhas)

# Criando o objeto de projeção
projecao <- CRS("+proj=longlat +datum=wgs84")

# Carregando o shapefile
am_sul <- readShapePoly (fn="countries_shp/Am_SUl_edit",proj4string = projecao)

# Carregando um único raster 
bio_1 <- raster(x="bio_5m_esri_ASC/bio_1.asc", crs=projecao)

# Carregando vários raster
vars_files <- list.files("./hadgem2_es/mohc_hadgem2_es_rcp8_5_2080s_bio_5min_r1i1p1_no_tile_asc",
pattern="\\.asc$",full.names=TRUE)
vars <- stack(vars_files)

# Cortando pela extensão
cortado <- crop(x=vars,y=am_sul)

# Refinando o corte pela máscara
masked <- mask(cortado,mask=am_sul)

# Escrevendo os raster cortados
writeRaster(cortado,filename= paste0("./AM_SUL/hadgem2_es/2080/rcp85/",names(cortado)),
format="ascii",bylayer=TRUE,NAflag=-9999)
````

===

### >>> Transferindo os shapes de espécies distribuídos em várias pasta  para uma única pasta <<<

#### Esta operação também pode ser feita utilizando a função `file.copy`, que inclusive de ser mais rápida.

````{r}
library(rgdal)
library(maptools)
library(foreach)

# Listando todos os arquivos shp dentro dos subdiretórios
files<-list.files(pattern = "\\.shp$",full.names=TRUE,recursive=TRUE,include.dirs=TRUE)

# Removendo o path dos elementos da lista
file_path<-basename(files)

# Substituindo a extensão "shp" do final dos elementos da lista
files_shp<- gsub(pattern=".shp",replacement="",file_path)

# Exportando cada shape para outra pasta.
foreach(i=files, j=files_shp) %do% {
shape<-readShapePoints(i)
writeOGR(shape, "./_TODAS", j, driver="ESRI Shapefile")
}
````
===

### >>> Adicionando o nome da espécie na tabela de atributos de shp de polígonos <<<

#### Comando `@data` é usado para acessar a tabela de atributos do shapefile
````{r} 
library(rgeos)
library(rgdal)
library(maptools)
library(raster)

# Cabeçalho e primeiras linhas da tabela de atributos do shapefile a1
head(a1@data)
                     nome_cient    bioma marinho fauna
1 Aguarunichthys_tocantinsensis amazonia       0     1
2            Aiouea_benthamiana amazonia       0     0
3              Aiouea_lehmannii amazonia       0     0
4          Albizia_glabripetala amazonia       0     0
5             Alouatta_belzebul amazonia       0     1
6             Alouatta_discolor amazonia       0     1

# Adiciona uma nova coluna  e a preenche com o valor "a" em todas as linhas
a1$nova_coluna <- "a"

# Cabeçalho e primeiras linhas da tabela de atributos do shapefile a1 após adição de coluna
head(a1@data)
                     nome_cient    bioma marinho fauna nova_coluna
1 Aguarunichthys_tocantinsensis amazonia       0     1           a
2            Aiouea_benthamiana amazonia       0     0           a
3              Aiouea_lehmannii amazonia       0     0           a
4          Albizia_glabripetala amazonia       0     0           a
5             Alouatta_belzebul amazonia       0     1           a
6             Alouatta_discolor amazonia       0     1           a

# Também é possível criar colunas com valores numéricos
a1$nova_coluna <- 0

# Cabeçalho e primeiras linhas da tabela de atributos do shapefile a1 após adição de coluna
head(a1@data)
                     nome_cient    bioma marinho fauna nova_coluna
1 Aguarunichthys_tocantinsensis amazonia       0     1           0
2            Aiouea_benthamiana amazonia       0     0           0
3              Aiouea_lehmannii amazonia       0     0           0
4          Albizia_glabripetala amazonia       0     0           0
5             Alouatta_belzebul amazonia       0     1           0
6             Alouatta_discolor amazonia       0     1           0

# Remove coluna
a1@data <- a1@data [,-5]
head(a1@data)
                     nome_cient    bioma marinho fauna
1 Aguarunichthys_tocantinsensis amazonia       0     1
2            Aiouea_benthamiana amazonia       0     0
3              Aiouea_lehmannii amazonia       0     0
4          Albizia_glabripetala amazonia       0     0
5             Alouatta_belzebul amazonia       0     1
6             Alouatta_discolor amazonia       0     1

# Remove coluna (outra opacao)
a1 <- a1[,-5]
head(a1@data)
                     nome_cient    bioma marinho fauna
1 Aguarunichthys_tocantinsensis amazonia       0     1
2            Aiouea_benthamiana amazonia       0     0
3              Aiouea_lehmannii amazonia       0     0
4          Albizia_glabripetala amazonia       0     0
5             Alouatta_belzebul amazonia       0     1
6             Alouatta_discolor amazonia       0     1
````
===

### >>> Merge de shapefile de especies <<<

#### Merge só funciona se os shapefiles tiverem as mesmas colunas, para isso pode se deletar ou criar colunas para que haja correspondência.

````{r}
library(rgeos)
library(rgdal)
library(maptools)
library(raster)

# Lista arquivos das especies
sp_files <- list.files(path="./7_intersect_biomas_arc_copia", pattern="\\.shp$",full.names=TRUE)

# Carrega o primeiro shapefile
sp_shp <- readShapePoly(sp_files[1],proj4string=CRS("+proj=longlat +datum=WGS84"))

# Formata o nome especie retirando caracteres indesejaveis
nome_sp <- gsub(sp_shp$nome_cient,pattern = " ", replacement = "_" )
nome_sp <- gsub(nome_sp,pattern = "-", replacement = "_" )
nome_sp <- gsub(nome_sp,pattern = '[(]', replacement = "" )
nome_sp <- gsub(nome_sp,pattern = ")", replacement = "" )
sp_shp@data$nome_cient <- nome_sp

# Remove colunas nao coincicentes entre os shapefiles
sp_shp@data <- sp_shp@data[,c(2,4)]

# Retira o primeiro shapefile da lista 
sp_files <- sp_files[-1]

for (shp in sp_files){

    # Carrega o shapefile da especie
    sp_shp_2 <- readShapePoly(shp,proj4string=CRS("+proj=longlat +datum=WGS84"))

    # Formata o nome especie retirando caracteres indesejaveis
    nome_sp_2 <- gsub(sp_shp_2$nome_cient,pattern = " ", replacement = "_" )
    nome_sp_2 <- gsub(nome_sp_2,pattern = "-", replacement = "_" )
    nome_sp_2 <- gsub(nome_sp_2,pattern = '[(]', replacement = "" )
    nome_sp_2 <- gsub(nome_sp_2,pattern = ")", replacement = "" )
    sp_shp_2@data$nome_cient <- nome_sp_2

    # Remove colunas nao coincicentes entre os shapefiles
    sp_shp_2@data <- sp_shp_2@data[,c(2,4)]
    cat(shp,"\n")

    # Merge do shapefile
    sp_shp <- rbind.SpatialPolygonsDataFrame(sp_shp,sp_shp_2, makeUniqueIDs = TRUE)
    cat(shp,"merged","\n")
}
````

===

### >>> Dissolve por mais de um campo <<<

#### Se usar a função `gUnaryUnion` mesmo indicando mais de um campo no atributo `id`, será feito um dissolve tudo.
#### Para fazer um `dissolve` por mais de um campo use a função `aggregate`.
#### ATENÇÃO: Valores `NA` ocosionam erro no `aggregate`

````{r}
library(rgeos)
library(rgdal)
library(maptools)
library(raster)

# Carregar shapefile 
sp_biomas <- readShapePoly(fn= "./sp_biomasEdit.shp",proj4string=CRS("+proj=longlat +datum=WGS84"))

# Faz o dissolve
a = aggregate(sp_biomas, by = list(sp_biomas@data$nome_cient,sp_biomas@data$bioma,sp_biomas@data$marinho),
dissolve = TRUE, FUN= mean) 
# disol <- gUnaryUnion(sp_biomas, id = c(sp_biomas@data$nome_cient,sp_biomas@data$bioma,sp_biomas@data$marinho))
 
# Escreve o shapefile
shapefile(x= a, filename=("sp_biomas_disol_R.shp"))
````
===
### >>> Gerando raster <<<

####  ATENÇÃO: ao usar `rasterize` se a área do polígono for muito pequena ele será ignorado. É melhor transferir o valor do pixel para um grid de pontos e criar o raster a partir deste grid, ou realizar um operação do tipo "spatial join" com um fishnet e criar um raster a partir do fishnet. 

````{r} 
library(raster)

# Carrega a tabela que contem todos os valores das proporcoes
emp_planejado <- read.table("./planejado/7_wgs/emp_planejado.txt",sep=",",header = T)

#Carrega o shapefile de ptos
ptos<-shapefile("./_GRID/grid_caatinga_ptos.shp")

# Carrega o raster do bioma
r_bioma<- raster("./_GRID/grid_caatinga_disol_64bit.tif")

# Lista tipologias
tipologia<-unique(emp_planejado$tipologia)

for (tp in tipologia){
    
# Selecionado shape com base nos atributos (select by attributes)
tipo<- emp_planejado[emp_planejado$tipologia==tp,]

# Merge: equivale a um join de tabelas
join <- merge(ptos,tipo,by="Id",all.x=F)

# Rasterizar: transforma o poligono do empreendimento em raster
r_pol <- rasterize(join,r_bioma,field="propEmp",background=NA,mask=F)

# Merge com o raster do bioma, para que os valores NA no r_pol sejam transformados em 0.
raster_merge <- merge(r_pol,r_bioma)

# Escreve o raster
writeRaster(raster_merge,filename= paste0("./planejado/8_raster/",tp,"_planejado"),format="GTiff",NAflag=-9999,overwrite=TRUE)  

rm(join)
}
````
===

### >>> RASTERIZAR POLIGONOS (TIPO SPATIAL JOIN) <<<

#### Ao usar a função `rasterize` dirtamente em um polígono, caso ele seja muito menor que a célula, pode ocorrer de ser gerado um raster vazio. Um forma de contornar este problema é "converter" o polígono para polígonos de tamanho equivalente ao da célula. Algo semelhante ao um "spatial join" ou "select by location".
````{r}
library(rgeos)
library(rgdal)
library(maptools)
library(raster)
library(snowfall)

# Lista arquivos das especies
sp_files <- list.files(path="./1_split_sp_nome", pattern="\\.shp$",full.names=TRUE)

# Carrega o fishnet
fishnet <- readShapePoly("./1_fishnet/_BIOMAS.shp",proj4string=CRS("+proj=longlat +datum=WGS84"))

# Carrega o raster modelo
raster_modelo <- raster("raster_modelo/fishnet_brasilzee_1.tif")

# Carrega o limite o fishnet
limite_fish <- readShapePoly ("./1_fishnet/fish_barsilzee_disol.shp",proj4string=CRS("+proj=longlat +datum=WGS84"))

# Adiciona a coluna pixel preenchendo com 0
limite_fish$pixel <- 0 


## Funcao Rasterizar
rasterizar_pol <- function( sp,
                            fishnet,
                            limite_fish,
                            raster_modelo)  {
  
  
  # Carrega o shapefile da especie
  sp_shp <- readShapePoly(sp,proj4string=CRS("+proj=longlat +datum=WGS84"))
  
  # Adiciona coluna pixel no shape especie
  sp_shp@data$pixel <- 1
  
  # Nome especie formatado
  nome_sp <- gsub(sp_shp$nome_cient,pattern = " ", replacement = "_" )
  nome_sp <- gsub(nome_sp,pattern = "-", replacement = "_" )
  nome_sp <- gsub(nome_sp,pattern = '[(]', replacement = "" )
  nome_sp <- gsub(nome_sp,pattern = ")", replacement = "" )
  nome_sp <- gsub(nome_sp,pattern = '[.]', replacement = "" )
  nome_sp <- gsub(nome_sp,pattern = "subsp._", replacement = "" )
  nome_sp <- gsub(nome_sp,pattern = "var._", replacement = "" )
  
  cat("Shapefile de ", nome_sp, "carregado." ,"\n")
  
  # Identifica as celulas que tem sobreposicao
  col_nome_cient <- over(y=sp_shp, x=fishnet)
  
  cat("Sobreposicao com fishnet feita." ,"\n")
  
  # Leva para o fishnet a coluna com as celula com sobreposicao indicadas
  fishnet@data <- cbind(fishnet@data,col_nome_cient)
  
  cat("Adicao da coluna de sobreposicao ao fishnet feita." ,"\n")
  
  # Subset do fishnet retirando apenas aquelas celulas que tem sobreposicao com a especie.
  fishnet_sp <- subset (fishnet, pixel==1 )
  
  cat("Subset do fishnet feito." ,"\n")
  
  # Remove outras colunas deixando apenas a coluna pixel
  fishnet_sp <- fishnet_sp[,4]
  
  # Merge com o limite do fishnet dissolvido
  merge <- rbind.SpatialPolygonsDataFrame(fishnet_sp,limite_fish, makeUniqueIDs = TRUE)
  
  cat("Merge do fishnet com o limite dissolvido feito." ,"\n")
  
  # substituir os NA da coluna pixel por 0
  #fishnet1@data$pixel[is.na(fishnet1@data$pixel)] <- 0
    
  # Faz dissolve pelo campo pixel
  #disol <- gUnaryUnion(merge, id = merge@data$pixel)
  disol <-  aggregate(merge, by = list(merge@data$pixel), dissolve = TRUE, FUN= mean) 
  
  # Transforma o poligono em raster
  r <- rasterize(disol,raster_modelo,field="pixel",NAflag=-9999)
  
  cat("Shapefile rasterizado." ,"\n")

  # Escreve o raster
  writeRaster(x= r, filename = paste0("./3_raster_R/",nome_sp),format="GTiff", overwrite=TRUE, NAflag=-9999)
  
  cat("Raster salvo." ,"\n")
  
  # Remove a coluna nome_cient e pixel do fishnet retornando para condicao inicial
  fishnet <- fishnet[,-c(4,3)]
  
  cat("Fishnet retorna ao estado original." ,"\n")
  
  cat("###############################################################" ,"\n")
  
} # Fecha a funcao

rasterizar_pol(sp= sp_files, fishnet=fishnet, limite_fish = limite_fish, raster_modelo= raster_modelo)

#################### Processamento em paralelo ####################

# Inicia o processamento em paralelo
sfInit(parallel=T,cpus=2)

# Carrega os pacotes nos cluster(nucleos)
sfLibrary(raster)  
sfLibrary(maptools) 
sfLibrary(rgeos)  
sfLibrary(rgdal)

# Roda a funcao em paralelo
sfClusterApplyLB(sp_files,rasterizar_pol, fishnet=fishnet, limite_fish = limite_fish, raster_modelo= raster_modelo)

# Para o processamento em paralelo
sfStop()

# Desligar o PC
system('shutdown -s')
````

### >>> Reparando geometria usando `repeat` e `break` <<<

#### Alguns problemas de geometria podem ser corrigidos fazendo um buffer de largura 0. Assim será criado um shapefile com limite igual ao original, mas com a geometria correta.
````{r}

library(raster)
library(maptools)
library (rgeos)

# Ler o shapefile do limite da borda da America do Sul
limite_shp_borda  <- readShapePoly ("0_limites/am_sul_borda.shp",proj4string=CRS("+proj=longlat +datum=WGS84"))

# Loop repetido para corrigir geometrias
 repeat {
       
   # Checa se a geometria do shapefile tem algum problema # Havendo problema corrige.
   if (gIsValid(limite_shp_borda) == FALSE){
       limite_shp_borda <-gBuffer(limite_shp_borda, byid=TRUE,width=0.0)}
   
   if (gIsValid(limite_shp_borda) == TRUE) {
       print("geometria corrigida")
   break} # Sai do loop
   
} # fecha o repeat
````
===

### >>> “Clip” Intersection entre dois shapefile de polígonos <<<

https://cran.r-project.org/web/packages/rgeos/rgeos.pdf
````{r}
library(rgeos)
# intereseção entre sp_pol e bioma
intersection <- gIntersection(sp_pol,bioma)
````
===
### >>> Split layer by attributes <<<

#### Separa as feições de um shapefile de acordo com uma coluna de atributos.

````{r}

library(raster)
library(maptools)

# Carrega nome do arquivo
arquivo <- list.files( path = ".", pattern = "\\.shp$",full.names = T)

# Carrega o shapefile
shp <- shapefile(arquivo)

# Lista as especies
taxons <- unique(shp$taxon)

# Split layer by attributes
for (t in taxons) {
  
 sp1<-subset(shp,shp$taxon==t)

 t <- gsub(pattern = " ",replacement = "_",t)
 
 writeSpatialShape(x=sp1,fn=paste0("./split/",t))
}
````
===
### >>> Normaliza valores de um raster <<<

````{r}
setwd("I:/Gp-A-COAPRO-bsa/projetos/vulnerabilidade_uc/Rodrigo")

library(raster)

# Carregar o raster

r <- raster("dist_lenco1.tif")

# Normaliza o raster para vairar de 0 a 1
r_01 <- (r - minValue(r)) / (maxValue(r) - minValue(r))
````
===

