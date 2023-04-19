# Netlfix-Challenge - APS5_ALGLIN

Esse é um projeto da matéria de Algebra Linear e Teoria da Informação do Insper para o curso de Ciência da Computação.

# Projeto: Desafio Netflix

Neste projeto, trabalharemos com o [The Movies Dataset](https://www.kaggle.com/datasets/rounakbanik/the-movies-dataset), que possui a avaliação de usuários em relação a filmes. Utilizando o csv `ratings_small.csv`, presente neste dataset.

## Descrição do Projeto

Enunciado: Um problema que existe hoje em dia com as grandes empresas de streaming (Netflix, Spotify, etc.) é que elas têm um acervo de conteúdo muito grande, e os usuários tendem a gostar, cada um, de uma pequena parte desse acervo. Então, como poderíamos escolher quais filmes vão aparecer tela inicial do seu streaming?

O projeto consiste em fazer a implementação de um sistema preditor de nota de filmes por usuário, o sistema projetado deve ser avaliado usando um histograma dos erros ao longo de várias estimativas. O número de estimações deve ser, no mínimo, mil.

## Descrição Matemática 

O modelo matemático utilizado para o desenvolvimento do projeto foi baseado na ideia de que o sistema de recomendação utiliza "perfis" típicos de usuários. Os perfis, são vetores que mostram que notas são tipicamente atribuídas para cada filme por usuários daquele perfil. Por exemplo, talvez tenhamos dois perfis e três filmes, e nesse caso poderíamos ter os perfis:

* $p_0 = [2, 5, 2]$, isto é, o perfil $0$ é de uma pessoa que gosta muito do filme $f_1$, e
* $p_1 = [5, 0, 4]$, isto é, o perfil $1$ é de uma pessoa que gosta dos filmes $f_0$ e $f_2$. 

Porém, sabemos que usuários reais raramente se comportam estritamente como um perfil. As notas realmente atribuídas por um usuário aos filmes, então, são modeladas como combinações lineares dos perfis. Por exemplo, podemos ter usuários:

* $u_0 = 0.1 p_0 + 0.9 p_1$, para um usuário muito próximo de $p_1$ mas distante de $p_0$,
* $u_1 = 0.1 p_0 + 0.1 p_1$, para um usuário distante tanto de $p_0$ quanto de $p_1$,

Então, o que precisamos é de uma maneira de mapear usuários para perfis, e então perfis para filmes. Precisamos então *decompor* nossa matriz $A$ de usuários $\times$ filmes em componentes:

$A = X Y Z$,

onde:
* $A$ tem uma linha por usuário e uma coluna por filme,
* $X$ tem uma linha por usuário e uma coluna por perfil,
* $Y$ é quadrada e mapeia perfis para perfis,
* $Z$ tem uma linha por perfil e uma coluna por filme.


No notebook da aula 5, vimos que os autovetores e autovalores são conceitos importantes para a visualização de dados em uma matriz. Nesse projeto, usamos esses conceitos para fazer recomendações de filmes.


Primeiramente, como precisamos, através de um arquivo csv contendo os dados: usuário, filme e nota, criar uma matriz de usuários $\times$ filmes, usamos a função `pivot_table` do pacote `pandas`. Essa função recebe como parâmetro o arquivo csv, e retorna uma matriz com os dados do arquivo. 

Os autovetores são vetores especiais de uma matriz quadrada, e quando multiplicados por essa matriz eles resultam em um vetor que é uma multiplicação escalar do autovetor original. Por exemplo, se temos uma matriz $A$ e um autovetor $v$, então $Av$ é um vetor que é uma multiplicação escalar de $v$ por um número real $x$. Esse número real $x$ é chamado de autovalor.

$Av = v x$

Com o entendimento desses conceitos, vimos tambem que podemos reduzir a dimensionalidade de uma matriz, mantendo a maior parte da informação, ao manter apenas os autovetores com autovalores maiores. Isso é feito multiplicando a matriz pela matriz de autovetores, e depois multiplicando o resultado pela transposta da matriz de autovetores, como no exemplo abaixo:

$A = X Y Z$

$A \approx X Y Z = X Y Z V V^T$

$A \approx X Y Z V V^T = X Y Z V \cdot V^T$

$A \approx X Y Z V \cdot V^T = X Y Z V \cdot V^T Z^T$

$A \approx X Y Z V \cdot V^T Z^T = X Y V \cdot V^T Z^T$

$A \approx X Y V \cdot V^T Z^T = X Y V \cdot V^T$

$A \approx X Y V \cdot V^T = X Y V V^T$

$A \approx X Y V V^T = X Y$

Para o projeto, usamos a decomposição SVD para reduzir a dimensionalidade da matriz de usuários $\times$ filmes. A decomposição SVD é uma decomposição de valores singulares, e é uma generalização da decomposição em valores próprios. A decomposição SVD é feita da seguinte forma:

$A = U \Sigma V^T$

onde:

* $A$ é uma matriz $m \times n$,

* $U$ é uma matriz $m \times m$,

* $\Sigma$ é uma matriz diagonal $m \times n$,

* $V$ é uma matriz $n \times n$.

A matriz $\Sigma$ é uma matriz diagonal, e seus elementos são os autovalores da matriz $A$. A matriz $U$ é uma matriz de autovetores da matriz $A A^T$, e a matriz $V$ é uma matriz de autovetores da matriz $A^T A$.

Nossa resolução para o desafio consiste em fazer uma "redução de ruído" em uma matriz com dados "estragados", com intuito de voltar a matriz para um estado mais próximo de sua forma original. Para isso, nós escolhemos uma posição aleatória da matriz que contenha uma nota válida, e atribuímos a ela um valor aleatório.

Para fazer o cálculo da decomposição SVD, usamos a função `svd` do pacote `scipy.linalg`. Essa função recebe como parâmetro a matriz $A$, e retorna as matrizes $U$, $\Sigma$ e $V$. Através da matriz $\Sigma$, que contem os autovalores, podemos escolher quantos autovetores queremos manter. Para isso, através de um parametro `k`, escolhemos quantos autovalores queremos manter, e então, através do método `diagsvd`, da biblioteca `scipy.linalg`, criamos uma matriz diagonal com os autovalores escolhidos.

Para fazer a recomendação de filmes, usamos a função `dot` do pacote `numpy`, que faz a multiplicação de matrizes. A função `dot` recebe como parâmetro as matrizes $U$, $\Sigma$ e $V$, e retorna a matriz $A$ reduzida.

Tendo a matriz $A$ reduzida, podemos calcular a diferença entre a matriz original e a matriz reduzida. Essa diferença é o erro cometido na redução de dimensionalidade. Para fazer uma análise profunda, nós fizemos 2 iterações de 670 iterações, e para cada iteração, o parametro `k` é incrementado em 1 dentro de um intervalor de 1 a 670. Para cada iteração, nós calculamos o erro, e salvamos em um arquivo csv. No final, nós plotamos um gráfico com o erro em função do parametro `k`. 



## Como rodar o projeto e Funcionalidades

### Clone o repositório do nosso projeto:

```py
https://github.com/WeeeverAlex/Netlfix-Challenge
```

### Depois instale os requirements:

```py
pip install -r requirements.txt
```

## Por fim, basta executar o arquivo demo.py: 

```py
python demo.py
```
O projeto possui as seguintes funcionalidades:

## Resultado final

## Autores

- [@st4pzz](https://github.com/st4pzz)
- [@WeeeverAlex](https://github.com/WeeeverAlex)

