### >>> Decisao de qual UC sera representada pela celula <<<

#### Uma forma de decidir qual será a uc representada pela célula do grid, quando mais de uma UCs está presente na mesma célula.
#### Os critérios de decisão foram escolhidos em reuniões com outras coordenações.

````{r}
#library(plyr)
setwd("D:/2016/vulnerabilidade_uc")

# Ler tabela de entrada
tab <- read.table("union.txt",header=T,sep="\t")
names(tab) <- c("ID_CEL","COD_UC","PIUS","ESFERA","TIPO","CATEGORIA","etapa_peso", "apoio_soci",
                "sobrep","govern_pad","area_ha","anoCriacao")


# Converte os caracteres da tabela em numeros

# PIUS
PIUS <- as.vector(tab$PIUS)
PIUS <- gsub(pattern = "PI",replacement = 2, x=PIUS)
PIUS <- gsub(pattern = "US",replacement = 1, x=PIUS)
PIUS <- as.numeric(PIUS)
unique(PIUS) # Confere se todos os valores foram transformados
# ESFERA
ESFERA <- as.vector(tab$ESFERA)
ESFERA <- gsub(pattern = "Federal",replacement = 3, x=ESFERA)
ESFERA <- gsub(pattern = "estadual",replacement = 2, x=ESFERA)
ESFERA <- gsub(pattern = "municipal",replacement = 1, x=ESFERA)
ESFERA <- as.numeric(ESFERA)
unique(ESFERA)# Confere se todos os valores foram transformados
# TIPO
TIPO <- as.vector(tab$TIPO)
TIPO <- gsub(pattern = "existente",replacement = 2, x=TIPO)
TIPO <- gsub(pattern = "planejada",replacement = 1, x=TIPO)
TIPO <- as.numeric(TIPO)
unique(TIPO) # Confere se todos os valores foram transformados
# CATEGORIA
CATEGORIA <- as.vector(tab$CATEGORIA)
CATEGORIA <- gsub(pattern = "REBIO",replacement = 11, x=CATEGORIA)
CATEGORIA <- gsub(pattern = "ESEC",replacement = 10, x=CATEGORIA)
CATEGORIA <- gsub(pattern = "PARQUE",replacement = 9, x=CATEGORIA)
CATEGORIA <- gsub(pattern = "RVS",replacement = 8, x=CATEGORIA)
CATEGORIA <- gsub(pattern = "MONA",replacement = 7, x=CATEGORIA)
CATEGORIA <- gsub(pattern = "UCPI",replacement = 6, x=CATEGORIA)
CATEGORIA <- gsub(pattern = "RPPN",replacement = 5, x=CATEGORIA)
CATEGORIA <- gsub(pattern = "RESEX",replacement = 4, x=CATEGORIA)
CATEGORIA <- gsub(pattern = "RDS",replacement = 4, x=CATEGORIA)
CATEGORIA <- gsub(pattern = "FLORESTA",replacement = 4, x=CATEGORIA)
CATEGORIA <- gsub(pattern = "ARIE",replacement = 3, x=CATEGORIA)
CATEGORIA <- gsub(pattern = "APA",replacement = 2, x=CATEGORIA)
CATEGORIA <- gsub(pattern = "ND",replacement = 1, x=CATEGORIA)
CATEGORIA <- as.numeric(CATEGORIA)
unique(CATEGORIA) # Confere se todos os valores foram transformados

# AREA
AREA <- tab$area_ha

# Ano Criacao
ANO <- tab$anoCriacao

# Tabela numerica
tab_num <- cbind(tab[,1:2],PIUS,ESFERA,TIPO,CATEGORIA,AREA,ANO)
tab_num <- subset(tab_num,COD_UC!=" ") # retira as linhas que não são UC

# Tabela de UCs
UCs <-as.data.frame(unique(tab_num$COD_UC))
names(UCs) <- "COD_UC"
tab_ucs <- tab_num[,c(-1,-7)]
tab_ucs <- unique(tab_ucs, nmax=1)

# Tabela de frequencia para uc. Conta em quantas celulas a UC esta contida.
tab_freq <- as.data.frame(table(tab_num$COD_UC))
names(tab_freq)<- c("COD_UC","Frequencia")
tab_freq <- tab_freq[-1,]

# UCs que aparecem em apenas uma celula
#UC_1 <- subset(tab_freq, Frequencia == 1)

# Merge da tab_num com a Frequencia das UCs
tab_num <- merge(tab_num,tab_freq, by="COD_UC", all = TRUE)

# ID das celulas
id_cel <- unique(tab_num$ID_CEL)

# Cria uma tabela para receber as linhas escolhidas
tab_final <- data.frame(row.names=NULL)

# Cria uma tabela para receber casos de ID repetidos
#tab_repetidos <- data.frame(row.names=NULL)

# For loop para cada valor de ID_CEL

for (i in id_cel) {
  
# Separa as linhas da tabela de um celula
tab_id <- data.frame(subset(tab_num,ID_CEL==i))

# Imprimi no console
cat("decidindo célula", i,"\n")

# Verificar se existem UCs existentes e planejadas

# Somente UC existente
if (sum(unique(tab_id$TIPO)) == 2) {
  
  # Prioriza as UCs q estao somente em uma celula
  if (1 %in% tab_id$Frequencia == TRUE){
  tab_id_1 <- data.frame(subset(tab_id,Frequencia == 1))
  } else { tab_id_1 <- tab_id }
  
  # Separa as linhas que tem o maior valor de ESFERA seguindo a ordem decrescente federal>estadual>municipal
  tab_esfera <- data.frame(subset(tab_id_1,ESFERA == max(tab_id_1$ESFERA)))
  
  # Separa as linhas que tem o maior valor de CATEGORIA seguindo a ordem decrescente 
  # 8)REBIO    7) ESEC    6) PARNA   5)REVIS    4)MONA    3)RPPN  2)RESEX/RDS/FLONA   1) ARIE 0)APA
  tab_categoria <- data.frame(subset(tab_esfera,CATEGORIA == max(tab_esfera$CATEGORIA)))
  
  # Separa a linha que tiver maior area
  tab_area <- data.frame(subset(tab_categoria,AREA == max(tab_categoria$AREA)))
  
  # Separa a linha que tiver menor ano
  tab_ano <- data.frame(subset(tab_area,ANO == min(tab_area$ANO)))
  
}


# Somente UC planejadas
if (sum(unique(tab_id$TIPO)) == 1) {
  
  # Prioriza as UCs q estao somente em uma celula
  if (1 %in% tab_id$Frequencia == TRUE){
    tab_id_1 <- data.frame(subset(tab_id,Frequencia == 1))
  } else { tab_id_1 <- tab_id }

  # Separa as linhas que tem o maior valor de ESFERA seguindo a ordem decrescente federal>estadual>municipal
  tab_esfera <- data.frame(subset(tab_id_1,ESFERA == max(tab_id_1$ESFERA)))
  
  # Separa as linhas que tem o maior valor de CATEGORIA seguindo a ordem decrescente 
  # 8)REBIO    7) ESEC    6) PARNA   5)REVIS    4)MONA    3)RPPN  2)RESEX/RDS/FLONA   1) ARIE 0)APA
  tab_categoria <- data.frame(subset(tab_esfera,CATEGORIA == max(tab_esfera$CATEGORIA)))
  
  # Separa a linha que tiver maior area
  tab_area <- data.frame(subset(tab_categoria,AREA == max(tab_categoria$AREA)))
  
  # Separa a linha que tiver menor ano
  tab_ano <- data.frame(subset(tab_area,ANO == min(tab_area$ANO)))
  
}

# UCs existentes + planejadas
if (sum(unique(tab_id$TIPO)) == 3) {
  
  # Prioriza as UCs q estao somente em uma celula
  if (1 %in% tab_id$Frequencia == TRUE){
    tab_id_1 <- data.frame(subset(tab_id,Frequencia == 1))
  } else { tab_id_1 <- tab_id }
  
  # Separa as linhas que tem o maior valor de ESFERA seguindo a ordem decrescente federal>estadual>municipal
  tab_esfera <- data.frame(subset(tab_id_1,ESFERA == max(tab_id_1$ESFERA)))
  
  # Separa as linhas que tem o maior valor de CATEGORIA seguindo a ordem decrescente 
  # 8)REBIO    7) ESEC    6) PARNA   5)REVIS    4)MONA    3)RPPN  2)RESEX/RDS/FLONA   1) ARIE 0)APA
  tab_categoria <- data.frame(subset(tab_esfera,CATEGORIA == max(tab_esfera$CATEGORIA)))
  
  # Separa a linha 
  # tab_pius <- data.frame(subset(tab_categoria,PIUS == max(tab_categoria$PIUS)))
  
  # Separa a linha que for existente em detrimento da planejada
  tab_tipo <- data.frame(subset(tab_categoria,TIPO == max(tab_categoria$TIPO)))
  
  # Separa a linha que tiver maior area
  tab_area <- data.frame(subset(tab_tipo,AREA == max(tab_tipo$AREA)))
  
  # Separa a linha que tiver menor ano
  tab_ano <- data.frame(subset(tab_area,ANO == min(tab_area$ANO)))
  
}


# Cria um data frame para armazenar os ID_CEL repetidos se ainda restar
#if (i %in% tab_final$ID_CEL == TRUE){
#    tab_repetidos <- rbind(i)}

# Adiciona a linha escolhida na tabela final
tab_final <- rbind(tab_final,tab_ano)  

} # Fecha For Loop


  
# Escreve a tabela final
write.table(tab_final,file="tab_final_1.txt",quote=FALSE,sep = "\t",row.names = FALSE)

write.table(uc_taltantes_Plan,file="tab_faltantes_Planejadas.txt",quote=FALSE,sep = "\t",row.names = FALSE)



#### Ucs faltantes ####
ucs_final <- as.data.frame(unique(tab_final$COD_UC))
ucs_final <- cbind(ucs_final, as.vector(rep(1,1308)))
names(ucs_final) <- c("COD_UC", "PRESENTE")
uc_faltantes <- merge(x=tab_ucs,y =ucs_final,by="COD_UC",all.x=TRUE)
uc_faltantes1 <- subset(uc_faltantes, is.na(PRESENTE))
uc_faltantes_Fed <- subset(uc_faltantes1, ESFERA==3)
uc_faltantes_Est <- subset(uc_faltantes1, ESFERA==2)
uc_faltantes_Mun <- subset(uc_faltantes1, ESFERA==1)
uc_faltantes_Plan <- subset(uc_faltantes1, TIPO ==1)

#### Checar id_cel repetidos ####
# Ler a tabela final
tab_final <- read.csv("tab_final_1.txt",header=T,sep="\t")
# Checa quais celulas estao repetidas
repetidos <- as.character(duplicated(tab_final$ID_CEL))
"TRUE" %in% repetidos
tab_final_repetidos <- cbind(tab_final,repetidos)
tab_final_repetidos <- subset(tab_final_repetidos, repetidos=="TRUE")
tab_final_repetidos$ID_CEL

# Cria uma tabela com as celulas repetidas
tab_repetidos_all <- merge(tab_final,tab_final_repetidos, by="ID_CEL",all=FALSE)
#### ####

#### Checar se todas as UCs estão presentes ####

# Lista dos codigos CNUC
COD_UC <- unique(tab_num$COD_UC)

# Cria uma tabela para receber a checagem
tab_final_faltantes <- data.frame(COD_UC= numeric(0),REPETE=character(0),row.names=NULL)  

# Roda a checagem
for (cod in COD_UC) {
    linha <- data.frame(cod,cod %in% tab_final$COD_UC)
    tab_final_faltantes <- rbind(tab_final_faltantes,linha)
    cat(cod,cod %in% tab_final$COD_UC, "\n")
  }
names(tab_final_faltantes) <- c("CNUC","PRESENTE")

# Codigos que estao faltantes  
uc_faltantes <- subset(tab_final_faltantes, PRESENTE == "FALSE") 

#### ####
write.table(tab_final_repetidos,file="tab_final_repetidos.txt",quote=FALSE,sep = "\t",row.names = FALSE)
````
===
