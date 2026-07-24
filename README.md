# norm.R — Normalização Radiométrica de Imagens Landsat via PIFs

Script em R para normalização radiométrica de imagens Landsat-5, usando Pontos Pseudoinvariantes (PIF) e regressão linear entre bandas. Implementa o método descrito em:

> **Modelo automático de normalização radiométrica de série multitemporal Landsat-5 usando pontos pseudoinvariantes (PIF)**
> Revista Brasileira de Cartografia — [acesse o artigo completo](https://seer.ufu.br/index.php/revistabrasileiracartografia/article/view/44014/23265)

## O que o script faz

Dada uma imagem a ser corrigida e uma imagem de referência, o script:

1. Extrai (`untar`) a cena Landsat bruta de dentro do diretório de trabalho para a pasta `IMAGENS_BRUTAS/`.
2. Carrega as bandas B1, B2, B3, B4, B5 e B7 da imagem a ser normalizada.
3. Carrega as bandas correspondentes da imagem de referência (já corrigida), identificadas pelo padrão de nome `B<n>...C...tif`.
4. Lê um shapefile de pontos (os PIFs) presente no diretório.
5. Para cada banda, ajusta uma regressão linear simples entre os valores extraídos na imagem de referência e na imagem a normalizar, e aplica a equação da reta (`y = a·x + b`) sobre toda a banda.
6. Empilha as 6 bandas normalizadas e salva o resultado como `imagem_normalizada.tif` (GeoTIFF).
7. Gera, na pasta `RELATORIOS/`, uma planilha `.xls` por banda com os valores extraídos, o R² antes e depois da normalização e o erro (RMSE) da correção.
8. Gera `graficos.bmp`, com os gráficos de dispersão (antes/depois) das 6 bandas, reta ajustada e equação/R² em cada painel.

## O que são os PIFs

Pontos Pseudoinvariantes (*Pseudo-Invariant Features*) são alvos na cena cuja reflectância se mantém estável ao longo do tempo (ex.: afloramentos rochosos, áreas urbanas consolidadas, corpos d'água profundos). Eles servem de âncora para calcular a relação linear entre a imagem a ser corrigida e a imagem de referência, permitindo compensar diferenças atmosféricas e de iluminação entre datas de aquisição.

## Requisitos

- R (testado com o pacote `raster`)
- Pacote `raster` (traz as funções `raster`, `extract`, `stack`, `writeRaster`, `shapefile`)

```r
install.packages("raster")
```

## Estrutura de diretório esperada

Antes de rodar a função, o diretório de trabalho deve conter:

```
diretorio_de_trabalho/
├── *.tar                  # cena Landsat bruta (compactada, será extraída)
├── B1...C...tif           # bandas 1, 2, 3, 4, 5, 7 da imagem de referência (já normalizada)
├── B2...C...tif
├── ...
└── *.shp (+ .shx, .dbf...) # shapefile com os pontos PIF
```

- O arquivo `.tar` é a cena bruta a ser normalizada (formato de distribuição do USGS).
- As imagens de referência devem estar soltas no diretório, com nomes contendo `B<n>` e `C` (ex.: `LC08_..._B1_C.tif`), pois é esse padrão que o script usa para localizá-las.
- O shapefile de pontos deve ser o único `.shp` no diretório (o script pega o primeiro encontrado).
- **Atenção:** use `/` (barra normal) nos caminhos, não `\`.

## Como usar

```r
source("norm.R")

# Se o diretório atual já é o diretório de trabalho:
norm()

# Ou especificando o caminho:
norm(directory = "C:/dados/cena_landsat")
```

## Saídas geradas

| Saída | Local | Conteúdo |
|---|---|---|
| `imagem_normalizada.tif` | diretório de trabalho | GeoTIFF com as 6 bandas normalizadas |
| `RELATORIO_B1.xls` … `RELATORIO_B7.xls` | `RELATORIOS/` | Valores extraídos nos PIFs, R² antes/depois e RMSE, por banda |
| `graficos.bmp` | diretório de trabalho | Gráficos de dispersão antes/depois, por banda, com reta ajustada |

## Observações e limitações

- O script foi escrito para bandas do Landsat-5 (B1, B2, B3, B4, B5, B7); para outros sensores (ex. Landsat-8, com bandas espectrais diferentes) é necessário adaptar os nomes de banda.
- A busca dos arquivos depende de padrões de nome (`pattern=`) via `dir()`; nomes de arquivo fora do padrão esperado não serão localizados.
- A função assume exatamente um shapefile de pontos no diretório — se houver mais de um `.shp`, apenas o primeiro (ordem alfabética) é usado.
- O pacote `raster` está em modo de manutenção; para uso continuado em projetos novos, considere migrar para `terra`.

## Referência

Para a fundamentação teórica completa do método (seleção de PIFs, formulação da regressão, validação estatística), consulte o artigo publicado na Revista Brasileira de Cartografia:
https://seer.ufu.br/index.php/revistabrasileiracartografia/article/view/44014/23265
