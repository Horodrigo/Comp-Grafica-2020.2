# IFCE - Computa��o Gr�fica- Rasteriza��o de Linhas

Rodrigo Viana Castelo Branco - 20152017020218

# Introdu��o

  Este trabalho tem como objetivo apresentar alguns algoritmos para rasteriza��o de linhas em um monitor gr�fico diretamente e mem�ria de video, como os sistemas operacionais modernos impedem acesso direto a mem�ria de v�deo, foi utilizado um framework para realizar esta simula��o.

Mas antes, devemos conhecer alguns conceitos como: primitivas, rasteriza��o, como rasterizar primitivas gr�ficas.
  
## Rasteriza��o

  Primitivas em computa��o gr�fica s�o elementos b�sicos, como pontos e retas. A rasteriza��o � respons�vel por capturar estas primitivas e transform�-las em novas imagens, ou seja, � justamente o que acontece em monitores, displays, e at� mesmo impressoras.

  
## Pixel

  Um Pixel � um ponto luminoso no monitor, uma imagem � formanda a partir de um conjunto de pixels.

� composto por tr�s canais de cores - Vermelho, Verde e Azul, e mais um canal para tratar a transpar�ncia destas cores, chamamos este canal de alpha, compondo um sistema chamado RGBA.
  
  
  ![Disposição_Pixel](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/Disposi%C3%A7%C3%A3o_Pixel.png?raw=true)
  
  Para uma melhor clareza de c�digo, foram utilizadas 02 estruturas:

*  Define a intensidade das cores, onde 0 significa aus�ncia de cor e 255 a intensidade m�xima

```C
typedef struct Color
{   //INTENSIDADE DAS CORES
    int r = 255;
    int g = 255;
    int b = 255;
    int a = 0;
} Color;
```

* Define um pixel na tela, possui coordenadas _(x,y)_ e uma _cor_

```C
typedef struct Point
{
    int x;
    int y;

    struct Color color;

} Point;
  ```
  
  Estas cores s�o armazenadas em uma regi�o de mem�ria chamada Color_Buffer , que � respons�vel por armazenar informa��es da imagem que ser� rasterizada na tela.
  
  
  ![ColorBuffer](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/ColorBuffer.png?raw=true)
    

  Tendo em vista que o Color_Buffer � uma regi�o de mem�ria, e a �nica informa��o que temos s�o pontos de coordenadas, utilizamos uma fun��o offset para encontrar a informa��o para determinada coordenada do monitor.

````C
offset = 4 * (x + y * IMAGE_WIDTH);
````

# Rasterização de linhas

Ap�s entender a intera��o entre monitores e computadores, os algoritmos para tra�ar retas ser�o entendidos sem maiores dificuldades.

Antes de iniciar, assumimos que:

* ?x ? ?y, ou seja, 0 <= angulo_reta <= 1
* Nosso grid tem apenas coordenadas positivas (maior ou igual a 0)
* P(x0,y0) = (0,0)
* A dist�ncia entre cada pixel � 1

Note que retas que possuem �ngulos iguais a 0�, 45� e 90� s�o triviais de serem rasterizados.

![angulos_triviais](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/Pixel%20angulos%200%2090%2045.jpg?raw=true)


Porém, retas com angulações diferentes demandam um maior esforço para serem desenhadas. A seguir são demonstrados alguns algoritmos para realizar esta tarefa.

### Equação geral da reta

Podemos representar uma reta _r_ do plano cartesiano por meio de uma equação. Essa equação pode ser obtida a partir de um ponto A(xA, yA) e do coeficiente angular _m_ dessa reta.

Considere uma reta r não-vertical, de coeficiente angular m, que passa pelo ponto A(xA, yA). Vamos obter a equação dessa reta, tomando um ponto _P(x, y)_ tal que P ≠ A.

Este é o meio mais trivial de se obter uma reta através de dois pontos. Vajamos a seguir seu desenvolvimento.

````
y - y0 = m * (x - x0)  
y - y0 = (m * x) + (-m * x0)
     y = (m * x) + (-m * x0)  + y0
     y = (m * x) + [ (-m * x0)  + y0 ]
     y = m * x + b
````


![Eq_Geral_Reta](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/Equa%C3%A7%C3%A3o%20Geral.png?raw=true)


Através desta equação podemos encontrar qualquer reta.
OBS: _Devemos nos atentar nos casos em que:_

````
Δx = 0 (pois inviabiliza a divisão Δy/Δx).
Δx > Δx, isso nos obriga a trocar o valor dos pontos.
````

A equação geral da reta é ineficiente em meios computacionais pois, para executá-la é necessário realizar multiplicações e arredondamentos de números reais a cada ponto que percorremos e isso torna o processamento lento.


### Algoritmo de Bresenham

Também conhecido como algoritmo do ponto médio, é utilizado para traçar retas de forma incremental, trabalhando apenas com números inteiros.

A ideia do algoritmo é bastante simples, ele utiliza a equação implícita da reta como uma função de decisão, para identificar qual o próximo pixel a ser ativado. Esta função é utilizada de forma incremental em cada pixel. Veja abaixo o desenvolvimento da **Equação Implícita da Reta**.

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

Se aplicarmos um ponto na equação implícita e obtivemos 0 como resultado, significa que o ponto está sobre a reta.

![Efeito_Eq_Implicita](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/Efeito%20Equa%C3%A7%C3%A3o%20Impl%C3%ADcita.png?raw=true)

Seja m = (x0 + 1, y0 + 1/2) o ponto médio entre os pixels  (x0 + 1, y0 + 1) e  (x0 + 1, y0), iremos utilizar a função de decisão para avaliar qual pixel acender.


![decPontoMedio](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/PontoM%C3%A9dio.png?raw=true)


Note que a reta parte do ponto _(x0, y0)_ logo, não existe decisão tomada anteriormente, podemos identificar o nosso valor de decisão aplicando f(x0 + 1,  y0  + 1/2) - f (x0,  y0).

``
 f (m) - f (inicio) =  ax0 + by0 + c + a + b/2 - (ax0 + by0 + c )
 f (m) - f (inicio) = a + b/2
``

Logo, nosso valor de decisão inicial é:

````
d = a + b/2
````

![pontoMedio](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/Ponto_m%C3%A9dio.png?raw=true)

Após identificar qual pixel ativar através da função de decisão, é necessário verificar qual será o nosso próximo ponto médio, (x0 + 2, y0 + 1/2) ou (x0 + 2, y0 + 3/2)


![Escolha_e_ne](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/Escolha_e_ne.png?raw=true)


````C
if (decisão <= 0){
  d = m3
 	putPixel (xi + 1, yi) //Pixel E
}else{
  d = m2
	putPixel (xi + 1, yi + 1) //Pixel NE
}
````

_Importante:_ Esta versão do algoritmo de Bresenham funciona apenas para _0° <= angulo <= 1°_ porém podemos obter retas com outros coeficientes angulares por reflexão, como demonstrado na imagem abaixo:


![Bresenham_Octantes](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/Bresenham_Octantes.png?raw=true)


## Debugging - Algoritmo de Bresenham

Execução do algoritmo de Bresenham


![Bresenham](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/test%20retas.png?raw=true)


# Resultados

### Função _DrawLine_

Traça retas a partir de 02 pontos utilizando o algoritmo de Bresenham


![Retas](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/Linhas_Octantes.png?raw=true)


### Função _InterporlaçãoCor_

````
	( COR_pontoA - COR_pontoB ) / DISTANCIA ENTRE OS PONTOS
	
	Se o canal_de_cor >= 255 ou canal_de_cor <=0
		stop_interpolação();
````


![interpolaCor](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/interpola%C3%A7%C3%A3oCor.png?raw=true)