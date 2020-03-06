De Qiime2 a Phyloseq
================

### Exportando datos de Qiime2

Para poder trabajar con la libreria Phyloseq, es necesario construir un
**phyloseq object**, sin embargo no es posible usar los artefactos .qza
de Qiime2 para este fin; por lo que es necesario exportarlos y tratarlos
para poder construir el objeto.

Para la construccion del objeto phyloseq, necesitaremos:

  - OTU table - (filtered-table.qza)  
  - Taxonomy Table - (taxonomy.qza)  
  - Tree - (unrooted-tree.qza)  
  - metadata - (Tabla de metadatos)

Antes de empezar, crearemos una carpeta exclusiva para exportar los
archivos.

**Paso 1: Exportando la OTU table**

    qiime tools export \
    --input-path filtered-table.qza \
    --output-path phyloseq

Esto exportara nuestro artefacto .qza a un archivo .biom, por lo cual
hay que convertirlo al formato .txt.

    biom convert -i feature-table.biom -o otu_table.txt --to-tsv

Ya con nuestra otu\_table.txt, la abriremos en excel y cambiaremos el
header \#OTU ID a OTUID

**Paso 2: Exportar la tabla de taxonomia**

    qiime tools export \
    --input-path taxonomy.qza \
    --output-path phyloseq

ahora hay que abrir la tabla de taxonomia importada y cambiar el header
*Feature ID* por *OTUID*

**Paso 3: Exportar el arbol filogenetico**

    qiime tools export \
    --input-path unrooted-tree.qza \
    --output-path phyloseq

### Preparando los archivos en R

Ahora prepararemos nuestros archivos exportados para poder crear el
objeto phyloseq, debido a que trabajamos con una tabla de otus filtrada
y nuestro archivo de taxonomia no se encuentra filtrado, deberemos
realizar un par de pasos extra, sin embargo no es nada del otro mundo.

Una vez en R (de preferencia R Studio), marcaremos la carpeta phyloseq
como nuestro directorio de trabajo (si estas en un cluster, puedes
extraer los documentos y pasarlos a tu computadora personal)

    setwd("direccion_de_la_carpeta/phyloseq")

una vez marcado nuestro directorio de trabajo, procedemos a que R lea
nuestros archivos otu\_table.txt y taxonomy.tsv y los carge al ambiente,
para poder realizar la correccion mencionada con anterioridad

    # Para la tabla de otus / asv
    otu <-  read.table(file =   "otu_table.txt",    header  =   TRUE)
    head(otu)
    
    # Para la tabla de taxonomia
    tax <-  read.table(file =   "taxonomy.tsv", sep =   '\t',   header  =   TRUE)
    head(tax)

Ahora uniremos ambas tablas y las exportaremos en formato .txt

    # Uniendo las tablas
    merged_file <-  merge(otu,  tax,    by.x    =   c("OTUID"), by.y=c("OTUID"))
    head(merged_file)
    
    # Exportando en formato .tsv
    write.table(merged_file,    file    =   "combined_otu_tax", sep =   '\t',   col.names   =   
    TRUE,   row.names   =   FALSE)

Procederemos abriendo el archivo “combined\_otu\_tax” que contiene
nuestras tablas unidas en excel o un equivalente, ahora procederemos a
crear dos archivos a partir de este.

El primer archivo debera contener las columnas OTUID y las
correspondientes a las muestras con sus abundacias, este sera nombrado
como *OTU MAtrix* y lo guardaremos en formato .CSV

El segundo archivo sustituira a nuestro archivo de taxonomia anterios y
debera contener las columnas OTUID y Taxa, ya que tenemos ambas columnas
en el archivo, seleccionaremos la columna correspondiente a Taxa y nos
iremos a Datos -\> Herramientas de Datos -\> Texto en columnas; alli
seleccionaremos la opcion de datos delimitados, los separaremos por
tabulacion y por punto y coma (semicolon) y por ultimo, elegiremos el
formato general, ahora podremos asignar a mano los diferentes niveles
taxonomicos asociados\! Guardaremos el archivo como taxonomy en formato
.CSV.

### Creando nuestro archivo Phyloseq (Al fin\!)

Para iniciar, cargaremos las siguientes librerias

  - library(“ggplot2”)
  - library(“phyloseq”)
  - library(“ape”)

cargaremos nuestros archivos finales al ambiente de trabajo

    # La tabla de otus / asv
    otu_table   =   read.csv("otu_matrix.csv",  sep=",",    row.names=1)
    otu_table   =   as.matrix(otu_table)
    
    # La tabla de taxonomia
    taxonomy    =   read.csv("taxonomy.csv",    sep=",",    row.names=1)
    taxonomy    =   as.matrix(taxonomy)
    
    # La tabla de metadatos
    metadata    =   read.table("Sea_Cucumber_metadata.txt", row.names=1)
    
    # El arbol filogenetico
    phy_tree    =   read_tree("tree.nwk")

Una vez cargados en nuestro ambiente de trabajo, procedermos a darles
formato de objeto phyloseq:

    OTU =   otu_table(otu_table,    taxa_are_rows   =   TRUE)
    TAX =   tax_table(taxonomy)
    META    =   sample_data(metadata)

Ahora verificaremos que los OTU names sean consistentes entre los
archivos TAX, OTU y phy\_tree, ademas verificaremos que nuestros
archivos OTU y META tengan los mismos nombres de muestra.

    # OTU names
    taxa_names(TAX)
    taxa_names(OTU)
    taxa_names(phy_tree)
    
    # Sample names
    sample_names(OTU)
    sample_names(META)

**Ahora si**, ya que verificamos que todo esta en orden, crearemos el
objeto phyloseq definitivo.

    physeq  =   phyloseq(OTU,   TAX,    META,   phy_tree)
    physeq

  - **Nogard**
