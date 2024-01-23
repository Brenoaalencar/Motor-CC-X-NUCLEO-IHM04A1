# Motor-CC-X-NUCLEO-IHM04A1
Acionamento de motor de corrente contínua utilizando a topologia de Ponte H de um hardware de potência dedicado. A montagem ainda contará com um encoder que realizará a contagem de pulsos 
a cada inversão de nível lógico provocada por um disco dentado fixado ao eixo.

## Diagrama de blocos 
A seguir, diagrama de blocos do projeto.
<div align="center">
<img src ="https://github.com/Brenoaalencar/DAC_R2R_8bits/assets/72100554/70d4ef90-de0c-45c8-bf53-71c23e915de4" width="500px"/>
</div>

## Diagrama de ligações
A seguir, podemos visualizar o driver usado no projeto (X-NUCLEO-IHM04A1), bem como a descrição 
de seus respectivos pinos. Assinalados em vermelho estão os pinos usados para o 
acionamento do motor.
Os pinos IN1A e IN2A foram usados para entrada dos sinais de controle no 
Shield, PA_10 foi usado para a saída do PWM da placa NÚCLEO, chaveando a ponte H, PC_0 para a leitura do 
sinal analógico fornecido pelo potenciômetro e PB_8 para a leitura do sinal digital 
fornecido pelo encoder.

<div align="center">
<img src ="https://github.com/Brenoaalencar/DAC_R2R_8bits/assets/72100554/db2ad284-a984-4cd5-b0d8-0109aca1bd2c" width="500px"/>
</div>

## Esquemático
No desenvolvimento do circuito eletrônico, foi usado um módulo de potência ST 
X-NUCLEO-IHM04A1 como ponte H a ser chaveada pelo PWM proveniente da 
NUCLEO F303RE. O sinal do encoder chega ao microcontrolador para contagem de 
pulsos assim como o valor de tensão do potenciômetro para alterar o PWM.

<div align="center">
<img src ="https://github.com/Brenoaalencar/DAC_R2R_8bits/assets/72100554/d150c15c-2c5c-4271-9808-478de421dfe2" width="500px"/>
</div>

## Fluxograma
A etapa anterior a elaboração do código de acionamento do motor consiste na 
realização de um fluxograma que represente e facilite a organização da estratégia de 
programação.

<div align="center">
<img src ="https://github.com/Brenoaalencar/DAC_R2R_8bits/assets/72100554/d2eeeb55-19fa-4638-97d3-5013ee4d583d" width="400px"/>
</div>

No fluxograma acima, observa-se que o cálculo da velocidade no código será feito a partir de 
um timer e da contagem de pulsos, caso o timer não esteja esgotado, serão contados mais 
pulsos, caso contrário, a velocidade será calculada e o timer resetado.

## Software
A seguir, código da aplicação.

```C++
// Inclusão das bibliotecas usadas
#include "mbed.h"
// Declaração dos pinos e variáveis
InterruptIn D0_encoder(PA_8);          // Entrada do sinal digital do encoder
PwmOut PWM(PA_10);                     // Saída do PWM
DigitalOut IN1A(PB_4);                 // Saída do sinal de controle
DigitalOut IN2A(PB_5);                 // Saída do sinal de controle
AnalogIn Pot_10k(A5);                  // Entrada do sinal analógico do potenciômetro 
Serial pc(USBTX, USBRX);               // Saída serial de dados
// Variável de tempo
Timer timer;
// Variável de velocidade
int RPM = 0;
// Variável de PWM
float pwm = 0;
// Contador de pulsos
int pulsos = 0;

void conta_pulsos() { pulsos++; }
int main() {
  // Sinais de controle
  IN1A = 0;
  IN2A = 1;
  // Leitura do sinal digital do enconder
  D0_encoder.fall(&conta_pulsos);
  // Inicia o timer
  timer.start();
  while (1) {
    // Leitura do poteciômetro e geração do sinal PWM
    pwm = Pot_10k.read();
    PWM = pwm;
    if (timer.read() >= 0.5) {
      float RPS = pulsos / timer.read();
      RPM = RPS * 60 / 30;
      printf("Velocidade em RPM: %i\n\r", RPM);
      printf("Modulacao PWM: %.4f\n\r", pwm);
      pulsos = 0;
      timer.reset();
    }
  }
}
```
Observa-se que se fez necessário o uso de uma interrupção para contar as bordas de 
descida, de forma a identificar os pulsos do encoder. A partir desses pulsos e de um 
timer foi possível calcular a rotação do motor em RPS e posteriormente, o valor 
calculado foi convertido para RPM ao ser multiplicado por 60. 
Do ponto de vista do controle de velocidade por meio do potenciômetro, uma vez que 
tanto a saída do PWM como a entrada analógica fornecem valores que variam de 0 a 1, 
não foram necessários cálculos intermediários e somente a leitura do valor fornecido 
pelo potenciômetro bastou para efetuar o controle. 
Os sinais de controle IN1A e IN12 foram definidos conforme a tabela verdade do 
acionamento por meio da ponte H, caso seus valores fossem invertidos, o sentido de 
rotação do motor também alternaria.

## Montagem física
Após esquemático concluído, a montagem física foi realizada com o auxílio de 
uma protoboard para fixação do potenciômetro.

<div align="center">
<img src ="https://github.com/Brenoaalencar/DAC_R2R_8bits/assets/72100554/dcb5d7ad-ad05-48d4-9464-8c353991d3e2" width="500px"/>
</div>

## Captura e resultados
Tendo em vista que o controle da velocidade do motor CC pode ser efetuado 
tanto pela alteração direta dos valores de tensão quanto pelo PWM, uma forma de 
observar a proporcionalidade das duas grandezas bem como de verificar a veracidade 
dos dados calculados com auxílio do microcontrolador é plotar os gráficos de tensão 
pela velocidade. No caso do controle realizado diretamente pela variação da tensão na 
fonte de bancada, o gráfico está representado na imagem abaixo.

<div align="center">
<img src ="https://github.com/Brenoaalencar/DAC_R2R_8bits/assets/72100554/85c37f78-b2ac-4b1c-8902-2beb46270bab" width="500px"/>
</div>

Por fim, para os dados de velocidade calculados pelo microcontrolador, dada a 
rotação do motor, são enviadas as informações via serial para o computador e a tensão
verificada diretamente por um voltímetro. Os dados foram inseridos em uma planilha 
para a criação de um gráfico. A velocidade também foi aferida com o motor sendo 
energizado diretamente pela fonte, sem PWM. Como pode ser visto, o comportamento
do motor responde linearmente ao aumento de tensão em sua armadura, validando os
cálculos. 

<div align="center">
<img src ="https://github.com/Brenoaalencar/DAC_R2R_8bits/assets/72100554/4cce0f4d-8739-4140-a2a2-72c2f8fb63bb" width="500px"/>
</div>

## Apêndice
Uma alternativa para a aquisição de dados vem da leitura de tensão pelo 
microcontrolador. Para isso, será utilizado um divisor de tensão que limitará a tensão de 
entrada para até 3 volts. Dessa forma, a velocidade é monitorada juntamente com a 
tensão de armadura e pode ser envida via serial para um software, como Tera Term ou 
uma planilha do Excel. 

<div align="center">
<img src ="https://github.com/Brenoaalencar/DAC_R2R_8bits/assets/72100554/24872182-7a30-4461-9831-ee0d672d334c" width="500px"/>
</div>

Os autores, no entanto, verificaram divergências entre leituras no decorrer da 
variação de tensão, de forma que alguns valores flutuam. Abaixo evidência dessa 
flutuação. A dinâmica do próprio motor pode gerar tal ruído. Um capacitor de 100 nF 
foi adicionado em paralelo com a saída do motor para atenuar as flutuações.

<div align="center">
<img src ="https://github.com/Brenoaalencar/DAC_R2R_8bits/assets/72100554/472724b3-cb2e-4126-903c-01938c2f9554" width="500px"/>
</div>

Os dados foram coletados com o Excel e a resposta da velocidade, salvo os 
pontos que saltaram para próximo de 12 volts, se mostrou linearmente dependente da 
tensão de armadura.

<div align="center">
<img src ="https://github.com/Brenoaalencar/DAC_R2R_8bits/assets/72100554/fc71d79f-8e1f-49a0-bef9-4805f3a87024" width="500px"/>
</div>

## Próximos passos

Para melhores leituras, outras implementações de hardware e software foram 
pensadas pelos autores, como desconsiderar valores de tensão muito diferentes dos 
imediatamente anteriores, ou um cálculo da tensão média para determinado intervalo, o 
que faria que o teste se tornasse um pouco mais demorado, porém mais preciso, e um 
hardware que auxiliasse na filtragem do ruído.



