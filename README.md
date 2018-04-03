# msigdbr: MSigDB for Common Model Organisms in a Tidy Data Format

## Overview

Most people who work with genomic data are eventually tasked with performing pathway analysis.
The majority of pathways analysis software tools are R-based.
Depending on the tool, it may be necessary to import the pathways into R, translate genes to the appropriate species, convert between symbols and IDs, and format the object in the required way.

The goal of `msigdbr` is to provide MSigDB gene sets:

* for multiple frequently studied species
* in an R-friendly format (a data frame in a "long" format with one gene per row)
* as gene symbols and Entrez Gene IDs
* that can be used in a script without requiring additional files

Using MSigDB v6.1 (released October 2017).

## Installation

```r
devtools::install_github("igordot/msigdbr")
```

## Usage

Load package.

```r
library(msigdbr)
```

Check the species available in the database.

```r
msigdbr_show_species()
#>  [1] "Bos taurus"               "Caenorhabditis elegans"   "Canis lupus familiaris"  
#>  [4] "Danio rerio"              "Drosophila melanogaster"  "Gallus gallus"           
#>  [7] "Homo sapiens"             "Mus musculus"             "Rattus norvegicus"       
#> [10] "Saccharomyces cerevisiae" "Sus scrofa"
```

Retrieve all human gene sets.

```r
m_df = msigdbr(species = "Homo sapiens")
head(m_df)
#> # A tibble: 6 x 9
#>   gs_name        gs_id gs_cat gs_subcat human_gene_symb… species_name entrez_gene gene_symbol sources
#>   <chr>          <chr> <chr>  <chr>     <chr>            <chr>              <int> <chr>       <chr>  
#> 1 AAACCAC_MIR140 M126… C3     MIR       ABCC4            Homo sapiens       10257 ABCC4       NA     
#> 2 AAACCAC_MIR140 M126… C3     MIR       ACTN4            Homo sapiens          81 ACTN4       NA     
#> 3 AAACCAC_MIR140 M126… C3     MIR       ACVR1            Homo sapiens          90 ACVR1       NA     
#> 4 AAACCAC_MIR140 M126… C3     MIR       ADAM9            Homo sapiens        8754 ADAM9       NA     
#> 5 AAACCAC_MIR140 M126… C3     MIR       ADAMTS5          Homo sapiens       11096 ADAMTS5     NA     
#> 6 AAACCAC_MIR140 M126… C3     MIR       AGER             Homo sapiens         177 AGER        NA  
```

Retrieve mouse hallmark gene sets.

```r
m_df = msigdbr(species = "Mus musculus", category = "H")
head(m_df)
#> # A tibble: 6 x 9
#>   gs_name  gs_id gs_cat gs_subcat human_gene_symbol species_name entrez_gene gene_symbol sources     
#>   <chr>    <chr> <chr>  <chr>     <chr>             <chr>              <int> <chr>       <chr>       
#> 1 HALLMAR… M5905 H      ""        ABCA1             Mus musculus       11303 Abca1       Inparanoid,…
#> 2 HALLMAR… M5905 H      ""        ABCB8             Mus musculus       74610 Abcb8       Inparanoid,…
#> 3 HALLMAR… M5905 H      ""        ACAA2             Mus musculus       52538 Acaa2       Inparanoid,…
#> 4 HALLMAR… M5905 H      ""        ACADL             Mus musculus       11363 Acadl       Inparanoid,…
#> 5 HALLMAR… M5905 H      ""        ACADM             Mus musculus       11364 Acadm       Inparanoid,…
#> 6 HALLMAR… M5905 H      ""        ACADS             Mus musculus       11409 Acads       Inparanoid,…
```

The gene sets data frame can also be filtered manually.

```r
m_df = msigdbr(species = "Mus musculus") %>% filter(gs_cat == "H")
```

Use the gene sets data frame for `clusterProfiler` (for genes as Entrez Gene IDs).

```r
m_term2gene = m_df %>%
  dplyr::select(gs_name, entrez_gene) %>%
  as.data.frame()
enricher(gene = genes_entrez, TERM2GENE = m_term2gene, ...) 
```

Use the gene sets data frame for `clusterProfiler` (for genes as gene symbols).

```r
m_term2gene = m_df %>%
  dplyr::select(gs_name, gene_symbol) %>%
  as.data.frame()
enricher(gene = genes_symbols, TERM2GENE = m_term2gene, ...) 
```

Use the gene sets data frame for `fgsea` (convert to list).

```r
m_list = m_df %>% split(x = .$gene_symbol, f = .$gs_name)
fgsea(pathways = m_list, ...)
```

## Details

The Molecular Signatures Database (MSigDB) is a collection of gene sets originally created for use with the Gene Set Enrichment Analysis (GSEA) software.

Gene homologs are provided by HUGO Gene Nomenclature Committee at the European Bioinformatics Institute which integrates the orthology assertions predicted for human genes by eggNOG, Ensembl Compara, HGNC, HomoloGene, Inparanoid, NCBI Gene Orthology, OMA, OrthoDB, OrthoMCL, Panther, PhylomeDB, TreeFam and ZFIN.
For each human equivalent within each species, only the ortholog supported by the largest number of databases is used.

