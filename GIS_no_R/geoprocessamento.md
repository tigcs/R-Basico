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

### >>> Cortar variaveis para uma mesma extensao <<<
````{r}
setwd("D:/2016/vulnerabilidade_uc/todas_var")

library(raster)

# Lista os arquivos das variaveis
 var_files <- list.files(path="./0_fonte/", pattern = "\\.tif$",full.names = TRUE,include.dirs = FALSE)

# carrega o raster modelo 
 raster_mod <- raster("./0_limite/fishnet_brasilzee.tif")
 
 
for (v in var_files) {
  
  # Carrega o raster
  var <- raster(v)
  cat(basename(v)," carregado","\n")
  
  # Crop pela extensao
  var_crop <- crop(x= var, y= raster_mod)
  cat(basename(v)," cropped","\n")
  
  # Corrige a extensao caso o crop tenha um extensao reduzida
  var_crop <- extend(var_crop,raster_mod)
  
  # Mask pela limite do raster
  var_mask <- mask(x= var_crop, mask= raster_mod)
  cat(basename(v)," masked","\n")
  
  # Separa o nome da camada
  nome <- basename(v)
  nome <- gsub(nome, pattern = ".tif", replacement = "")
  
  # Escreve o raster
  writeRaster(x=var_mask, filename=paste0("./1_cortadas/",nome), NAflag = -9999, format="GTiff", overwrite=TRUE)
  cat(basename(v)," escrito","\n")
  cat("######################################################","\n")
  
  }# Fecha o loop
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

#### A função `bind` do pacote `raster` foi a melhor opção que encontrei até o momento, pois consegue juntar shapefiles mesmo não tendo as mesmas colunas na tabela de atributos.

````{r}
merge <- bind(mpc,uf)
````

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

### >>> Rasterizar polígonos (Tipo Spatial Join) <<<

#### Ao usar a função `rasterize` dirtamente em um polígono, caso ele ocupe menos de 50% da célula, será gerado um raster vazio. Um forma de contornar este problema é "converter" o polígono para polígonos de tamanho equivalente ao da célula. Algo semelhante ao um "spatial join" ou "select by location".
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
===
### >>> Rasterizar Polígonos pelo número(posição da célula) <<<

####  Ao usar a função `rasterize` dirtamente em um polígono, caso ele ocupe menos de 50% da célula, será gerado um raster vazio. Um forma de contornar este problema é "converter" o polígono para polígonos de tamanho equivalente ao da célula. Algo semelhante ao um "spatial join" ou "select by location". Porém, este método pode se apresentar extremamente lento. Uma outra saída é extrair do raster o número(posição da célula) que tem sobreposição com o polígono, gerar os centróides destas células e em seguida cortar(`mask`) pelos centróides gerados. Desta forma todas as células que tiverem mais de 1% da sua área sobreposata pelo polígono será extraída.

````{r}
 # Carrega o shapefile da nova proposta de uc
 nova_proposta <- readShapePoly("nova_proposta.shp", proj4string=CRS("+proj=longlat +datum=WGS84"))
 
 # Lista de arquivos raster
 atv_anto_files <- list.files(path = "./atv_antropicas", pattern = "\\.tif$",full.names = T)
 remanescente_file <- list.files(path = "./remanescente",pattern = "\\.tif$",full.names = T)
 var_clim_file <- list.files(path = "./variacao_climatica",pattern = "\\.tif$",full.names = T)
 
 # Agrupa os raster em um unico objeto
 rasters <- stack(x=c(atv_anto_files,remanescente_file,var_clim_file))
 
 # Identifica quais celulas (numero de posicao) tem mais de 1% de sua area sobreposta pelo poligono.
 # O argumento "weights" deve estar como TRUE, caso contraio sera identificadas apenas celulas com mais de
 # 50% de sobreposicao.
 n_cel <- cellFromPolygon(rasters, nova_proposta, weights=T)
 n_cel <- as.data.frame(n_cel)
 n_cel <- n_cel$cell

 # Transforma o numero de posicao da celula em um shapefile de pontos
 pto_r <- xyFromCell(rasters_nova, n_cel, spatial=T)
 
 # Corta o raster pelo shapefile de pontos
 rasters_nova <- mask(rasters, mask = pto_r)

````
===
### >>> Reparando geometria <<<

#### Alguns problemas de geometria podem ser corrigidos fazendo um buffer de largura 0. Assim será criado um shapefile com limite igual ao original, mas com a geometria correta. 
````{r}
library(raster)
library(maptools)
library (rgeos)

# Carrega o shapefile das areas prioritarias MMA 2007 para criacao e ampliacao ed Uc
 priori_MMA <- readShapePoly("./area_prioritaria_MMA/area_prioritarias_2007.shp", proj4string=CRS("+proj=longlat +datum=WGS84"))
 
 # Checa se a geometria do shapefile tem algum problema # Havendo problema corrige.
   if (gIsValid(limite_shp_borda) == FALSE){
       limite_shp_borda <-gBuffer(limite_shp_borda, byid=TRUE,width=0.0)}
 # Confere se a geometria foi realmente corrigida.  
   if (gIsValid(limite_shp_borda) == TRUE) { print("geometria corrigida")}
````
#### Entretanto, este atrifício falha se houver polígonos com buracos, pois estes serão eliminados. Uma forma mais eficaz é usar a função `gSimplify`. Porém, esta função elimina a tabela de atributos, mas que pode ser inserida novamente se salva antes como um `data frame` em um objeto separado.
````{r}
library(raster)
library(maptools)
library (rgeos)

# Carrega o shapefile das areas prioritarias MMA 2007 para criacao e ampliacao ed Uc
 priori_MMA <- readShapePoly("./area_prioritaria_MMA/area_prioritarias_2007.shp", proj4string=CRS("+proj=longlat +datum=WGS84"))

 # Checa se a geometria do shapefile tem algum problema # Havendo problema corrige.
 if (gIsValid(priori_MMA) == FALSE){
   tab_atributos <- priori_MMA@data
   priori_MMA <- gSimplify(priori_MMA, tol=0, topologyPreserve=F)
  # Devolve a tabela de atributos transformando o Spatial Polygon em Spatial Polygon Data Frame
   priori_MMA <- SpatialPolygonsDataFrame(priori_MMA,data=tab_atributos)
   rm(tab_atributos)}
 
 # confere se realmente a geometria foi corrigida
 if (gIsValid(priori_MMA) == TRUE) {print("Geometria OK")}
 ````
===

### >>> Loop com Repeat e Break <<<

#### Não é necessário corrigir geometria com `repeat`, é apenas um exemplo de como `repeat`e `break`podem ser usados em um loop.
````{r}
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
### >>> Intersect e Dissolve de especies em UCs <<<
````{r}
setwd("D:/2016/vulnerabilidade_uc/especies_poligonos")

library(rgeos)
library(rgdal)
library(maptools)
library(raster)

# Carregar shapefile de ucs
ucs <- readShapePoly(fn= "./1_merge/ucs.shp",proj4string=CRS("+proj=longlat +datum=WGS84"))

# Lista arquivos das especies
sp_files <- list.files(path="./6_split_sp", pattern="\\.shp$",full.names=TRUE)

# Cria uma tabela para receber a lista da especie sem intersecao com ucs
sp_sem_inter <- data.frame(row.names=NULL)


for (shp in sp_files) {
  
# Carrega o shapefile da especie
sp_shp <- readShapePoly(shp,proj4string=CRS("+proj=longlat +datum=WGS84"))

# Nome especie
nome_sp <- gsub(sp_shp$nome_cient,pattern = " ", replacement = "_" )

cat("shapefile de ", nome_sp,"carregado", "\n")

# Especies com intersecao
if (gIntersects(sp_shp,ucs) == TRUE) {
  
  
 # Checa a geometria e rapara
  if (gIsValid(sp_shp) == FALSE){
    sp_shp <- gBuffer(sp_shp,width=0.0)
    }else{cat("Geometria OK","\n")}

# Faz o intersect entre UCs e shape especies
inter <- intersect(sp_shp,ucs)

cat("shapefile de ", nome_sp,": intersect feito", "\n")

# Faz o dissolve
disol <- gUnaryUnion(inter, id = inter@data$nome_cient)
disol$nome_cient <- nome_sp

cat("shapefile de ", nome_sp,": dissolve feito", "\n")

# Escreve o shapefile
shapefile(x= disol, filename=paste0("./7_R_inter/",nome_sp,".shp"))

cat("shapefile de ", nome_sp,": salvo", "\n")

} # Fecha condicao


# Especies sem intersecao
if (gIntersects(sp_shp,ucs) == FALSE) {
  
  cat("shapefile de ", nome_sp,": SEM interseção", "\n")
  
  # Adicina o nome da sp sem intersecao
  sp_sem_inter <- rbind(sp_sem_inter,nome_sp)
  
}# Fecha condicao
 cat ("#######################################################################", "\n")  

} # Fecha o for loop

# Troca o nome da coluna
names(sp_sem_inter) <- "taxon"

# Escreve a tabela de especies sem intersecao
write.table(x= sp_sem_inter, file = "./7_R_inter/_SP_SEM_INTER.txt",quote = FALSE,row.names = FALSE)

# Desliga o PC
system('shutdown -s')
````
===
### >>> Baixa registros de ocorrência de espécies armazenados no GBIF <<<
````{r}
library(dismo)

# Carrega lista de espécies
especies <- read.table(file = , header = T, sep="\t")

# Busca e baixa registro de ocorrência que tem coordenadas e sem problemas geoespaciais (args)
occ <- gbif(especies[1], download=T, geo = T, args=c("hasGeospatialIssue=FALSE", "hasCoordinate=TRUE"))

# Separa apenas as colunas de nome da espécies e coordenadas
occ <- occ[,c("species", "lon", "lat")]

# Conta quantos registros há
qtdade_pto <- length(occ$species)

# Insere na Tabela da espécie a coluna de quantidade de pontos
occ$qtdade_pto <- qtdade_pto

# Retira da lista de espécies a espécie já baixada
especies <- especies[-1]

# Cria data frame para registro de espécies sem registro no GBIF
especies_s_PTO <- data.frame("SP"=character(0))


for( sp in especies){
  
  cat( "Buscando regsitros de: ", sp, "\n")
  
  ## Busca e baixa registro de ocorrência que tem coordenadas e sem problemas geoespaciais
  occ1 <- gbif(sp, download=T, geo = T, args=c("hasGeospatialIssue=FALSE", "hasCoordinate=TRUE"))
  
  ## Armazena registros encontrados
  if (is.null(occ1)==FALSE){
  
  # Separa apenas as colunas de nome da espécies e coordenadas
  occ1 <- occ1[,c("species", "lon", "lat")]
  
  # Conta quantos registros há
  qtdade_pto <- length(occ1$species)
  
  # Insere na Tabela da espécie a coluna de quantidade de pontos
  occ1$qtdade_pto <- qtdade_pto
  
  # Junta as tabelas de todas as espécies em uma única tabela de registro de ocorrência
  occ <- rbind(occ,occ1)}
  
  ## Registra espécies sem registro
  if (is.null(occ1)==TRUE){especies_s_PTO <- rbind(especies_s_PTO, data.frame("SP"=sp))}
 
}

# Salva a tabela com todas as espécies
write.table(x=occ, file = , quote = F,sep = "\t",row.names = F)

````
===
### >>> Transformação de CRS e cálculo de Área <<<
#### Mais informações em: https://github.com/tigcs/R-Basico/blob/master/GIS_no_R/OverviewCoordinateReferenceSystems.pdf retirado de: https://www.nceas.ucsb.edu/~frazier/RSpatialGuides/OverviewCoordinateReferenceSystems.pdf
````{r}

# CRS WGS85
CRS("+init=epsg:4326")

# Cria o CRS South America Albers Equal Area. A tag "+towgs84" fornece os parâmetros de transformação(x,y,z).
albers <- CRS("+proj=aea +lat_1=-5 +lat_2=-42 +lat_0=-32 +lon_0=-60 +x_0=0 +y_0=0 +ellps=aust_SA +units=m +towgs84=-66.87,4.37,-38.52 +no_defs")

# Projeta o shapefile para CRS South America Albers Equal Area
t <- spTransform (grid_bl5, CRSobj = albers)

# Cálculo de área
t$Area_m2 <- area(t)

# Outra forma de calcular a área
t$Area_m21 <- gArea(t, byid=T)


````
===
