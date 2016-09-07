# Seleciona apenas arquivos que tenham unicamente a extensão “.tif”
# Se colocar apenas <pattern= ".tif"> selecionará também outros arquivos tipo “.tif.aux” ou “.tif.xml” etc.
files <- list.files(pattern = "\\.tif$")
