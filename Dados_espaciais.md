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
#### Cortando variáveis pela extensão da América do Sul
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

### >>>

