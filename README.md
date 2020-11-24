# IFCE - Computação Gráfica- Rasterização de Linhas

Rodrigo Viana Castelo Branco - 20152017020218

# Introdução

  Este trabalho tem como objetivo apresentar alguns algoritmos para rasterização de linhas em um monitor gráfico diretamente e memória de video, como os sistemas operacionais modernos impedem acesso direto a memória de vídeo, foi utilizado um framework para realizar esta simulação.

Mas antes, devemos conhecer alguns conceitos como: primitivas, rasterização, como rasterizar primitivas gráficas.
  
## Rasterização

  Primitivas em computação gráfica são elementos básicos, como pontos e retas. A rasterização é responsável por capturar estas primitivas e transformá-las em novas imagens, ou seja, é justamente o que acontece em monitores, displays, e até mesmo impressoras.

  
## Pixel

  Um Pixel é um ponto luminoso no monitor, uma imagem é formanda a partir de um conjunto de pixels.

É composto por três canais de cores - Vermelho, Verde e Azul, e mais um canal para tratar a transparência destas cores, chamamos este canal de alpha, compondo um sistema chamado RGBA.
  
  
  ![DisposiÃ§Ã£o_Pixel](https://github.com/Horodrigo/Comp-Grafica-2020.2/tree/main/images/Disposi%C3%A7%C3%A3o_Pixel.png?raw=true)
  
  Para uma melhor clareza de código, foram utilizadas 02 estruturas:

*  Define a intensidade das cores, onde 0 significa ausência de cor e 255 a intensidade máxima

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
  
  Estas cores são armazenadas em uma região de memória chamada Color_Buffer , que é responsável por armazenar informações da imagem que será rasterizada na tela.
  
  
  ![ColorBuffer](https://github.com/Horodrigo/Comp-Grafica-2020.2/tree/main/images/ColorBuffer.png?raw=true)
    

  Tendo em vista que o Color_Buffer é uma região de memória, e a única informação que temos são pontos de coordenadas, utilizamos uma função offset para encontrar a informação para determinada coordenada do monitor.

````C
offset = 4 * (x + y * IMAGE_WIDTH);
````

# RasterizaÃ§Ã£o de linhas

Após entender a interação entre monitores e computadores, os algoritmos para traçar retas serão entendidos sem maiores dificuldades.

Antes de iniciar, assumimos que:

* ?x ? ?y, ou seja, 0 <= angulo_reta <= 1
* Nosso grid tem apenas coordenadas positivas (maior ou igual a 0)
* P(x0,y0) = (0,0)
* A distância entre cada pixel é 1

Note que retas que possuem ângulos iguais a 0°, 45° e 90° são triviais de serem rasterizados.

![angulos_triviais](https://github.com/Horodrigo/Comp-Grafica-2020.2/tree/main/images/Pixel%20angulos%200%2090%2045.jpg?raw=true)


PorÃ©m, retas com angulaÃ§Ãµes diferentes demandam um maior esforÃ§o para serem desenhadas. A seguir sÃ£o demonstrados alguns algoritmos para realizar esta tarefa.

### EquaÃ§Ã£o geral da reta

Podemos representar uma reta _r_ do plano cartesiano por meio de uma equaÃ§Ã£o. Essa equaÃ§Ã£o pode ser obtida a partir de um ponto A(xA, yA) e do coeficiente angular _m_ dessa reta.

Considere uma reta r nÃ£o-vertical, de coeficiente angular m, que passa pelo ponto A(xA, yA). Vamos obter a equaÃ§Ã£o dessa reta, tomando um ponto _P(x, y)_ tal que P â‰  A.

Este Ã© o meio mais trivial de se obter uma reta atravÃ©s de dois pontos. Vajamos a seguir seu desenvolvimento.

````
y - y0 = m * (x - x0)  
y - y0 = (m * x) + (-m * x0)
     y = (m * x) + (-m * x0)  + y0
     y = (m * x) + [ (-m * x0)  + y0 ]
     y = m * x + b
````


![Eq_Geral_Reta](https://github.com/Horodrigo/Comp-Grafica-2020.2/tree/main/images/Equa%C3%A7%C3%A3o%20Geral.png?raw=true)


AtravÃ©s desta equaÃ§Ã£o podemos encontrar qualquer reta.
OBS: _Devemos nos atentar nos casos em que:_

````
Î”x = 0 (pois inviabiliza a divisÃ£o Î”y/Î”x).
Î”x > Î”x, isso nos obriga a trocar o valor dos pontos.
````

A equaÃ§Ã£o geral da reta Ã© ineficiente em meios computacionais pois, para executÃ¡-la Ã© necessÃ¡rio realizar multiplicaÃ§Ãµes e arredondamentos de nÃºmeros reais a cada ponto que percorremos e isso torna o processamento lento.


### Algoritmo de Bresenham

TambÃ©m conhecido como algoritmo do ponto mÃ©dio, Ã© utilizado para traÃ§ar retas de forma incremental, trabalhando apenas com nÃºmeros inteiros.

A ideia do algoritmo Ã© bastante simples, ele utiliza a equaÃ§Ã£o implÃ­cita da reta como uma funÃ§Ã£o de decisÃ£o, para identificar qual o prÃ³ximo pixel a ser ativado. Esta funÃ§Ã£o Ã© utilizada de forma incremental em cada pixel. Veja abaixo o desenvolvimento da **EquaÃ§Ã£o ImplÃ­cita da Reta**.

````
y = mx + b                            //  Partimos da EquaÃ§Ã£o Reduzida da Reta
y = (dy/dx)* x + b                    //  Destrinchando 'm' obtemos '(dy/dx)'
y * dx = dy * x + b * dx              //  Multiplicando a equaÃ§Ã£o por 'dx'
dy * x + (-y * dx) + b * dx = 0       //  Igualando a equaÃ§Ã£o a 0
````

Realizando as modificaÃ§Ãµes abaixo para melhor entendimento:

````
Î± = dy
ÃŸ = -dx
c = b * dx
````

Obtemos:
````
decisÃ£o = f(x,y) = Î±x + ÃŸy + c = 0
````

Se aplicarmos um ponto na equaÃ§Ã£o implÃ­cita e obtivemos 0 como resultado, significa que o ponto estÃ¡ sobre a reta.

![Efeito_Eq_Implicita](https://github.com/Horodrigo/Comp-Grafica-2020.2/tree/main/images/Efeito%20Equa%C3%A7%C3%A3o%20Impl%C3%ADcita.png?raw=true)

Seja m = (x0 + 1, y0 + 1/2) o ponto mÃ©dio entre os pixels  (x0 + 1, y0 + 1) e  (x0 + 1, y0), iremos utilizar a funÃ§Ã£o de decisÃ£o para avaliar qual pixel acender.


![decPontoMedio](https://github.com/Horodrigo/Comp-Grafica-2020.2/tree/main/images/PontoM%C3%A9dio.png?raw=true)


Note que a reta parte do ponto _(x0, y0)_ logo, nÃ£o existe decisÃ£o tomada anteriormente, podemos identificar o nosso valor de decisÃ£o aplicando f(x0 + 1,  y0  + 1/2) - f (x0,  y0).

``
 f (m) - f (inicio) =  ax0 + by0 + c + a + b/2 - (ax0 + by0 + c )
 f (m) - f (inicio) = a + b/2
``

Logo, nosso valor de decisÃ£o inicial Ã©:

````
d = a + b/2
````

![pontoMedio](https://github.com/Horodrigo/Comp-Grafica-2020.2/tree/main/images/Ponto_m%C3%A9dio.png?raw=true)

ApÃ³s identificar qual pixel ativar atravÃ©s da funÃ§Ã£o de decisÃ£o, Ã© necessÃ¡rio verificar qual serÃ¡ o nosso prÃ³ximo ponto mÃ©dio, (x0 + 2, y0 + 1/2) ou (x0 + 2, y0 + 3/2)


![Escolha_e_ne](https://github.com/Horodrigo/Comp-Grafica-2020.2/tree/main/images/Escolha_e_ne.png?raw=true)


````C
if (decisÃ£o <= 0){
  d = m3
 	putPixel (xi + 1, yi) //Pixel E
}else{
  d = m2
	putPixel (xi + 1, yi + 1) //Pixel NE
}
````

_Importante:_ Esta versÃ£o do algoritmo de Bresenham funciona apenas para _0Â° <= angulo <= 1Â°_ porÃ©m podemos obter retas com outros coeficientes angulares por reflexÃ£o, como demonstrado na imagem abaixo:


![Bresenham_Octantes](https://github.com/Horodrigo/Comp-Grafica-2020.2/tree/main/images/Bresenham_Octantes.png?raw=true)


## Debugging - Algoritmo de Bresenham

ExecuÃ§Ã£o do algoritmo de Bresenham


![Bresenham](https://github.com/Horodrigo/Comp-Grafica-2020.2/tree/main/images/test%20retas.png?raw=true)


# Resultados

### FunÃ§Ã£o _DrawLine_

TraÃ§a retas a partir de 02 pontos utilizando o algoritmo de Bresenham


![Retas](https://github.com/Horodrigo/Comp-Grafica-2020.2/tree/main/images/Linhas_Octantes.png?raw=true)


### FunÃ§Ã£o _InterporlaÃ§Ã£oCor_

````
	( COR_pontoA - COR_pontoB ) / DISTANCIA ENTRE OS PONTOS
	
	Se o canal_de_cor >= 255 ou canal_de_cor <=0
		stop_interpolaÃ§Ã£o();
````


![interpolaCor](https://github.com/Horodrigo/Comp-Grafica-2020.2/tree/main/images/interpola%C3%A7%C3%A3oCor.png?raw=true)
