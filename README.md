# Introdução

  Este trabalho tem como objetivo apresentar um algoritmo para rasterização de pelo menos três linhas em um monitor.

Mas antes, apresentarei alguns conceitos aprendidos em aula como: primitivas, rasterização, como rasterizar primitivas gráficas.
  
## Rasterização

  Primitivas em computação gráfica são elementos básicos, como pontos e retas. A rasterização é responsável por capturar estas primitivas e transformá-las em novas imagens, ou seja, é justamente o que acontece em monitores, displays, e até mesmo impressoras.

# Rasterização de linhas

Antes de iniciar, assumimos que:

* x ≥ Δy, ou seja, 0 <= angulo_reta <= 1
* Nosso grid tem apenas coordenadas positivas (maior ou igual a 0)
* P(x0,y0) = (0,0)
* A distância entre cada pixel é 1

Retas que possuem ângulos iguais a 0°, 45° e 90° são triviais de serem rasterizados.
Porém, retas com angulações diferentes demandam um maior esforço para serem desenhadas. A seguir são demonstrados alguns algoritmos para realizar esta tarefa.

### Equação geral da reta

Podemos representar uma reta r do plano cartesiano por meio de uma equação. Essa equação pode ser obtida a partir de um ponto A(xA, yA) e do coeficiente angular m dessa reta.

Considere uma reta r não-vertical, de coeficiente angular m, que passa pelo ponto A(xA, yA). Vamos obter a equação dessa reta, tomando um ponto P(x, y) tal que P ≠ A.

Este é o meio mais trivial de se obter uma reta através de dois pontos. Vajamos a seguir seu desenvolvimento.
````
y - y0 = m * (x - x0)  
y - y0 = (m * x) + (-m * x0)
     y = (m * x) + (-m * x0)  + y0
     y = (m * x) + [ (-m * x0)  + y0 ]
     y = m * x + b
````
Através desta equação podemos encontrar qualquer reta.

### Algoritmo de Bresenham

Também conhecido como algoritmo do ponto médio, é utilizado para traçar retas de forma incremental, trabalhando apenas com números inteiros.

A ideia do algoritmo é bastante simples, ele utiliza a equação implícita da reta como uma função de decisão, para identificar qual o próximo pixel a ser ativado. Esta função é utilizada de forma incremental em cada pixel. Veja abaixo o desenvolvimento da Equação Implícita da Reta.

````
y = mx + b                            //  Partimos da Equação Reduzida da Reta
y = (dy/dx)* x + b                    //  Destrinchando 'm' obtemos '(dy/dx)'
y * dx = dy * x + b * dx              //  Multiplicando a equação por 'dx'
dy * x + (-y * dx) + b * dx = 0       //  Igualando a equação a 0
````

Realizando as modificações abaixo para melhor entendimento:

````
α = dy
ß = -dx
c = b * dx
````

Obtemos:
````
decisão = f(x,y) = αx + ßy + c = 0
````

## Debugging - Algoritmo de Bresenham

Execução do algoritmo de Bresenham

![Bresenham](https://github.com/Horodrigo/Comp-Grafica-2020.2/tree/main/images/test%20retas.png?raw=true)


# Resultados

### Função _DrawLine_

Traça retas a partir de 02 pontos utilizando o algoritmo de Bresenham

Não apenas três, mas várias retas foram traçadas para testar o algoritmo e tentar gerar retas horizontais e verticais.

![Retas](https://github.com/Horodrigo/Comp-Grafica-2020.2/tree/main/images/Linhas_Octantes.png?raw=true)

A primeira imagem foi usada para gerar pontos cardinais e subcardinais, de modo a se verificar com precisão as retas verticais e horizontais. Na segunda, várias retas foram geradas em dimensões diferentes para se verificar o funcionamento do algoritmo.

