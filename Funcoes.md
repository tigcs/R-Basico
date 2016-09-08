### >>> Função cat <<<
#### Similar à função `print`, escreve no console o que desejar. Para que escreva em linhas separadas (tipo enter) usar ao final do comando o separador **`\n`**. Exemplo:

````{r}
    cat(paste("Modelo de ",sp,"salvo.,"\n")) )

    Modelo de Panthera onca salvo.
    Modelo de Puma concolor salvo.
````

====

### >>> Função invisible <<<
#### Ao usar a função `lapply`, ao final do processamento, pode retornar, no console, valores `NULL` para cada elemento da lista usada no lapply. Exemplo:
````{r}
    lapply(lista_sp,final.model,algoritmo = "maxent", cod_algoritmo = "mx",performance = "TSS", 
    performance.value= 0.2, output.folder = "final_model",pondera.future = T)
 
    Modelo presente da he_pe ponderado pelo TSS salvo como: ./final_model/pre_he_pe_mx_wm.tif
    Modelo futuro da he_pe ponderado pelo TSS salvo como: ./final_model/fut_he_pe_mx_wm.tif
    Modelo presente da pe_ja ponderado pelo TSS salvo como: ./final_model/pre_pe_ja_mx_wm.tif
    Modelo futuro da pe_ja ponderado pelo TSS salvo como: ./final_model/fut_pe_ja_mx_wm.tif

[[1]]
NULL

[[2]]
NULL

[[3]]
NULL

[[4]]
NULL
````

#### Para evitar isso, pode-se usar a função `invisible` da seguinte forma:
````{r}
invisible(lapply(lista_sp,final.model,algoritmo = "maxent", cod_algoritmo = "mx",
performance = "TSS", performance.value= 0.2, output.folder = "final_model",pondera.future = T))
````

====

