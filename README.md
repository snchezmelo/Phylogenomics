# Phylogenomics
En este repositorio encontrará el archivo del taller de filogenómica realizado en la clase de bioinformática.

<div class=text-justify>

# **Taller introductorio a Filogenómica** :raised_hands: :page_facing_up:
##### Presentado por: Catalina Sánchez y David Cardona :sunglasses:

##### *Métodos*

Durante esta práctica se accedió al servidor _bola_ para construir la filogenia de 23 especies  de vertebrados usando enfoques **coalescentes** y **concatenados** a partir de un subset de los datos.

1. Usamos **Orthofinder** y los transcriptomas de las especies para hallar regiones ortólogas entre todas las proteínas.Así:

``orthofinder -os -M msa -S blast -f vertebrate_proteomes``

2. Usamos **Requal** para identificar fragmentos que no comparten homología y removerlos en cada output.Así:

``for f in *fa; do prequal $f ; done``

3. Para el alinemiento múltiple, utilizamos **MAFFT** para alinear cada ortogrupo. Así:

``for f in *filtered; do mafft $f > $f.mafft; done``

3. Con el objetivo de quedarnos solo con las regiones bien alineadas, usamos **BMGE** para remover las posiciones que tuvieran gaps superiores al 80%. Así:

``for f in *mafft; do java -jar /home/bioinformatics/softwares/BMGE-1.0/BMGE.jar -i $f -t AA -g 0.8 -h 1 -w 1 -of $f.g08.fas; done``

4. Usamos **FASconCAT** para unir todas las secuencias que teniamos en archivos separados.Así:

``perl /software/FASconCAT-G_v1.04.pl -l -s``

Con esto, tenemos un archivo final que debe contener 23 taxa con 21 genes. La matriz de concatenación está en ``FcC_supermatrix.fas`` y el archivo ``FcC_supermatrix_partition.txt`` contiene las particiones.

5. Construcción de árboles

* **Máxima verosimilitud**:Se realizaron dos análisis con **IQTREE**, uno que considera el alineamiento concatenado cómo un solo locus que evoluciona de una misma forma (_unpartitioned_) y otro que precisa donde inicia y termina cada gen para que calcule el mejor modelo para cada una de estas particiones (_partitioned_).Así:


``iqtree -s FcC_supermatrix.fas -m TEST -msub nuclear -bb 1000 -alrt 1000 -nt 1 -bnni -pre unpartitioned``

``iqtree -s FcC_supermatrix.fas -spp FcC_supermatrix_partition.txt -m TEST -msub nuclear -merit AICc -bb 1000 -alrt 1000 -nt 1 -bnni -pre partitioned``

* **Coalescencia**: En este caso, usamos **ASTRAL** para tomar el input de los árboles de genes para generar un árbol de especies. Primero se calculan los árboles de genes, así:

``for f in *filtered.mafft.g08.fas; do iqtree -s $f -m TEST -msub nuclear -merit AICc -nt 1; done``

Y luego el de especies, así:

``java -jar /home/bioinformatics/softwares/ASTRAL/astral.5.7.3.jar -i my_gene_trees.tre -o species_tree_ASTRAL.tre 2> out.log``

6. Para la visualización del árbol, copiamos los archivos para poderlos visualizar con **FigTree**; todoslos árboles se enraizaron por punto medio. Así:

``scp bioinformatics@bola-bioevolutiva.urosario.edu.co:~/SnchezMelo/filogenomica/vertebrate_proteomes/OrthoFinder/Results_May01/Orthogroup_Sequences/filtered/alinfinal ~/home/Documents``

##### *Resultados*:

* **Máxima verosimilitud**:

En general, se encontró que ambos métodos generan árboles muy similares para el conjunto de datos dado. Estas similitudes son especialmente evidentes en la topología y en la longitud de las ramas los cuales no difieren entre los dos métodos. Ahora bien, las diferencias encontradas radican principalmente en algunos de los valores de soporte de los nodos. Para el árbol _partitioned_ los valores de bootstrap tienden a ser mayores que los del árbol _unpartitioned_. Sin embargo, a pesar de esta diferencia, los valores de soporte de los nodos más basales son bastante similares entre los dos métodos y por ende la robustez que proporciona la estabilidad de los clados derivados no difiere de manera importante entre los dos modelos.

> _**Partitioned**_

![Arbol correspondiente a máxima verosimilitud (partitioned)](/Users/snchezmelo/Documents/partitioned.jpg)


> _**Unpartitioned**_

![Arbol correspondiente a máxima verosimilitud (unpartitioned)](/Users/snchezmelo/Documents/unpartitioned.jpg)

* **Coalescencia**:

En lo que respecta al árbol generado por coalescencia, este difiere de manera importantes de los árboles obtenidos por métodos de máxima verosimilitud. En lo que respecta a los valores de soporte, estos son significativamente más bajos y  tienden a afectar predominantemente los nodos de arranque, los cuales muestran los menores valores (Bootstrap < 0.6). A pesar de que topología varía poco con respecto a los otros arboles, la mayoría de cambios se reflejan en los clados terminales –donde además se reportan valores más bajos de Bootstrap–. En general, la topología también es muy diferenciada ya que a pesar de tener dos clados hermanos principales, las ramificaciones de estos no son similares a los de máxima verosimilitud.

![Arbol correspondiente a coalescencia]( /Users/snchezmelo/Documents/Coalescencia.jpg)


</div>
