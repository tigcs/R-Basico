### >>> Função `par` configura o plot <<<

     mar : determina o tamanho das margens
     oma : determina o tamanho do espaço além das margens
     mfrow : divide a área do plot em linhas e colunas

````{r} 
 par(mar=c(0,0,0,0),oma=c(0,0,2,0),mfrow = c(1, 2))
````
===

### >>> Função `plot` <<<

#### `col` : determina a cor do item de acordo com o Colorchart_R
#### `alpha` : determina a transparência
#### `border` : determina a cor da borda
#### `lwd` : determina a espessura da borda
#### `add` : plota o item no mesmo plot, sobrepõe
````{r}
plot(tx_shp,col=rgb(205/255,0/255,0/255,alpha=0.5), border="red4",lwd=3,add=T)
````
#### `outer` : determina que o título fique fora da área de plot
#### Para colocar em itálico o título `substitute(expr=italic (nome_sp),env = list(nome_sp=nome_sp))`
````{r}
title(main= substitute(expr=italic (nome_sp),env = list(nome_sp=nome_sp)), cex.main = 2, outer=TRUE)
````
