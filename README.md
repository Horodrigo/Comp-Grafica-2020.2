# IFCE - ComputaÁ„o Gr·fica- RasterizaÁ„o de Linhas

Rodrigo Viana Castelo Branco - 20152017020218

# IntroduÁ„o

  Este trabalho tem como objetivo apresentar alguns algoritmos para rasterizaÁ„o de linhas em um monitor gr·fico diretamente e memÛria de video, como os sistemas operacionais modernos impedem acesso direto a memÛria de vÌdeo, foi utilizado um framework para realizar esta simulaÁ„o.

Mas antes, devemos conhecer alguns conceitos como: primitivas, rasterizaÁ„o, como rasterizar primitivas gr·ficas.
  
## RasterizaÁ„o

  Primitivas em computaÁ„o gr·fica s„o elementos b·sicos, como pontos e retas. A rasterizaÁ„o È respons·vel por capturar estas primitivas e transform·-las em novas imagens, ou seja, È justamente o que acontece em monitores, displays, e atÈ mesmo impressoras.

  
## Pixel

  Um Pixel È um ponto luminoso no monitor, uma imagem È formanda a partir de um conjunto de pixels.

… composto por trÍs canais de cores - Vermelho, Verde e Azul, e mais um canal para tratar a transparÍncia destas cores, chamamos este canal de alpha, compondo um sistema chamado RGBA.
  
  
  ![Disposi√ß√£o_Pixel](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/Disposi%C3%A7%C3%A3o_Pixel.png?raw=true)
  
  Para uma melhor clareza de cÛdigo, foram utilizadas 02 estruturas:

*  Define a intensidade das cores, onde 0 significa ausÍncia de cor e 255 a intensidade m·xima

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
  
  Estas cores s„o armazenadas em uma regi„o de memÛria chamada Color_Buffer , que È respons·vel por armazenar informaÁıes da imagem que ser· rasterizada na tela.
  
  
  ![ColorBuffer](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/ColorBuffer.png?raw=true)
    

  Tendo em vista que o Color_Buffer È uma regi„o de memÛria, e a ˙nica informaÁ„o que temos s„o pontos de coordenadas, utilizamos uma funÁ„o offset para encontrar a informaÁ„o para determinada coordenada do monitor.

````C
offset = 4 * (x + y * IMAGE_WIDTH);
````

# Rasteriza√ß√£o de linhas

ApÛs entender a interaÁ„o entre monitores e computadores, os algoritmos para traÁar retas ser„o entendidos sem maiores dificuldades.

Antes de iniciar, assumimos que:

* ?x ? ?y, ou seja, 0 <= angulo_reta <= 1
* Nosso grid tem apenas coordenadas positivas (maior ou igual a 0)
* P(x0,y0) = (0,0)
* A dist‚ncia entre cada pixel È 1

Note que retas que possuem ‚ngulos iguais a 0∞, 45∞ e 90∞ s„o triviais de serem rasterizados.

![angulos_triviais](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/Pixel%20angulos%200%2090%2045.jpg?raw=true)


Por√©m, retas com angula√ß√µes diferentes demandam um maior esfor√ßo para serem desenhadas. A seguir s√£o demonstrados alguns algoritmos para realizar esta tarefa.

### Equa√ß√£o geral da reta

Podemos representar uma reta _r_ do plano cartesiano por meio de uma equa√ß√£o. Essa equa√ß√£o pode ser obtida a partir de um ponto A(xA, yA) e do coeficiente angular _m_ dessa reta.

Considere uma reta r n√£o-vertical, de coeficiente angular m, que passa pelo ponto A(xA, yA). Vamos obter a equa√ß√£o dessa reta, tomando um ponto _P(x, y)_ tal que P ‚â† A.

Este √© o meio mais trivial de se obter uma reta atrav√©s de dois pontos. Vajamos a seguir seu desenvolvimento.

````
y - y0 = m * (x - x0)  
y - y0 = (m * x) + (-m * x0)
     y = (m * x) + (-m * x0)  + y0
     y = (m * x) + [ (-m * x0)  + y0 ]
     y = m * x + b
````


![Eq_Geral_Reta](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/Equa%C3%A7%C3%A3o%20Geral.png?raw=true)


Atrav√©s desta equa√ß√£o podemos encontrar qualquer reta.
OBS: _Devemos nos atentar nos casos em que:_

````
Œîx = 0 (pois inviabiliza a divis√£o Œîy/Œîx).
Œîx > Œîx, isso nos obriga a trocar o valor dos pontos.
````

A equa√ß√£o geral da reta √© ineficiente em meios computacionais pois, para execut√°-la √© necess√°rio realizar multiplica√ß√µes e arredondamentos de n√∫meros reais a cada ponto que percorremos e isso torna o processamento lento.


### Algoritmo de Bresenham

Tamb√©m conhecido como algoritmo do ponto m√©dio, √© utilizado para tra√ßar retas de forma incremental, trabalhando apenas com n√∫meros inteiros.

A ideia do algoritmo √© bastante simples, ele utiliza a equa√ß√£o impl√≠cita da reta como uma fun√ß√£o de decis√£o, para identificar qual o pr√≥ximo pixel a ser ativado. Esta fun√ß√£o √© utilizada de forma incremental em cada pixel. Veja abaixo o desenvolvimento da **Equa√ß√£o Impl√≠cita da Reta**.

````
y = mx + b                            //  Partimos da Equa√ß√£o Reduzida da Reta
y = (dy/dx)* x + b                    //  Destrinchando 'm' obtemos '(dy/dx)'
y * dx = dy * x + b * dx              //  Multiplicando a equa√ß√£o por 'dx'
dy * x + (-y * dx) + b * dx = 0       //  Igualando a equa√ß√£o a 0
````

Realizando as modifica√ß√µes abaixo para melhor entendimento:

````
Œ± = dy
√ü = -dx
c = b * dx
````

Obtemos:
````
decis√£o = f(x,y) = Œ±x + √üy + c = 0
````

Se aplicarmos um ponto na equa√ß√£o impl√≠cita e obtivemos 0 como resultado, significa que o ponto est√° sobre a reta.

![Efeito_Eq_Implicita](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/Efeito%20Equa%C3%A7%C3%A3o%20Impl%C3%ADcita.png?raw=true)

Seja m = (x0 + 1, y0 + 1/2) o ponto m√©dio entre os pixels  (x0 + 1, y0 + 1) e  (x0 + 1, y0), iremos utilizar a fun√ß√£o de decis√£o para avaliar qual pixel acender.


![decPontoMedio](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/PontoM%C3%A9dio.png?raw=true)


Note que a reta parte do ponto _(x0, y0)_ logo, n√£o existe decis√£o tomada anteriormente, podemos identificar o nosso valor de decis√£o aplicando f(x0 + 1,  y0  + 1/2) - f (x0,  y0).

``
 f (m) - f (inicio) =  ax0 + by0 + c + a + b/2 - (ax0 + by0 + c )
 f (m) - f (inicio) = a + b/2
``

Logo, nosso valor de decis√£o inicial √©:

````
d = a + b/2
````

![pontoMedio](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/Ponto_m%C3%A9dio.png?raw=true)

Ap√≥s identificar qual pixel ativar atrav√©s da fun√ß√£o de decis√£o, √© necess√°rio verificar qual ser√° o nosso pr√≥ximo ponto m√©dio, (x0 + 2, y0 + 1/2) ou (x0 + 2, y0 + 3/2)


![Escolha_e_ne](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/Escolha_e_ne.png?raw=true)


````C
if (decis√£o <= 0){
  d = m3
 	putPixel (xi + 1, yi) //Pixel E
}else{
  d = m2
	putPixel (xi + 1, yi + 1) //Pixel NE
}
````

_Importante:_ Esta vers√£o do algoritmo de Bresenham funciona apenas para _0¬∞ <= angulo <= 1¬∞_ por√©m podemos obter retas com outros coeficientes angulares por reflex√£o, como demonstrado na imagem abaixo:


![Bresenham_Octantes](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/Bresenham_Octantes.png?raw=true)


## Debugging - Algoritmo de Bresenham

Execu√ß√£o do algoritmo de Bresenham


![Bresenham](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/test%20retas.png?raw=true)


# Resultados

### Fun√ß√£o _DrawLine_

Tra√ßa retas a partir de 02 pontos utilizando o algoritmo de Bresenham


![Retas](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/Linhas_Octantes.png?raw=true)


### Fun√ß√£o _Interporla√ß√£oCor_

````
	( COR_pontoA - COR_pontoB ) / DISTANCIA ENTRE OS PONTOS
	
	Se o canal_de_cor >= 255 ou canal_de_cor <=0
		stop_interpola√ß√£o();
````


![interpolaCor](https://github.com/FelipeNasci/Line_Rasterization/blob/master/images/interpola%C3%A7%C3%A3oCor.png?raw=true)