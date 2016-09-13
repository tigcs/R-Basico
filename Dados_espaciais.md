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

#### >>>

