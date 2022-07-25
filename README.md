Curso Superior em Engenharia Eletrônica 
Unidade Curricular: Projeto Integrador III
Professor: Daniel Lohmann e Robinson Pizzio
Florianópolis, Abril de 2022 
Aluno: Ramon Busiquia Serafim



# Análise de caixas de som



## Introdução

​		Esse relatório tem a finalidade de analisar, simular, montar em bancada e explicar um circuito que faz a análise de determinados espectros de frequência afim de determinar se a caixa de som está funcionando perfeitamente em todas as faixas de frequência.

Utilizando diversos componentes como um microfone ajustado para o propósito aplicado, construindo um ambiente acústico controlado sem reverberação e um microcontrolador para emitir padrões de sinais, fazendo a análise posteriormente comparando com o sinal recebido/enviado.

Possui o intuito de verificar alguns problemas encarados quando passamos da simulação para a execução em bancada, possíveis discrepâncias nos resultados entre ambas.



### Metodologia

​		Com o escopo do projeto definido, para fazer a análise semanalmente foi eleita algumas dúvidas para pesquisa, sendo apresentadas abaixo. Primeiro não devemos pensar na solução final, mas sim separar os problemas em partes e apresentar todas as possibilidades e posteriormente verificar qual será melhor parar o projeto.



### Microcontrolador

​		Foi feito um primeiro questionamento sobre qual usar. Ao detalhar o projeto, seria necessário montar todos os blocos em algum microcontrolador, porém o Professor Lohmann discutiu a possibilidade de utilizar um DSP, microprocessadores direcionados para realizar processamento digital de sinais. 

​		Com isso, debatendo com o Professor Pacheco que leciona aula de Processamento Digitais de Sinais II a principal questão imposta foi qual seria o método de análise do projeto, pois algumas análises só poderiam ser feitas com um DSP, como uma transformada de Fourier. Outros microcontroladores como ARM possuem ponto fixo, não podendo realizar essas análises.

​		O intuito do projeto é visualizar e comparar intensidade de ondas RMS, e isso pode ser feito com um ARM, não sendo preciso utilizar um DSP. Portanto, foi definido que para este projeto será utilizado um STM32F103C8.



### STM32F103C8 

​		Microcontrolador mais conhecido como Blue Pill atende todos os pré-requisitos de projeto, utilizando o software STM32CubeMX é possível configurar os pinos de forma mais simples, facilitando o desenvolvimento do projeto e disponibiliza a biblioteca HAL (Hardware Abstraction Layer). O código será apresentado no fim do documento.

#### Circuito para geração de sinais

​		Para realizar as diferentes frequências de áudio que serão estimuladas na caixa de som, é necessário um circuito que gerencie os sinais de interesse, para isso, será utilizado um Circuito Integrado AD9833. Possuindo um resolução de 28 bits,  comunicação SPI, capaz de gerar ondas senoidais, triangulares e quadradas em um intervalo de operação entre 0.1Hz a 25Mhz.

​		Ele gera um sinal de 12.65mW, considerado baixa potência para este projeto. Portanto, será necessário um bloco amplificador para conectá-lo a caixa de som.

#### Amplificador de áudio

​		Para reproduzir o sinal produzido pelo AD9833, precisamos utilizar um amplificador. Com isso, foi analisado alguns amplificadores afim de verificar qual a melhor escolha para o projeto.

|   Nome    | Classe |                  Descrição                   | Potência(W) |
| :-------: | :----: | :------------------------------------------: | :---------: |
|  TA7274P  |   D    | Amplificador de Potência para radio de carro |     12      |
|  TBA820   |   B    |            Amplificador de audio             |     1.2     |
|  TCA760   |   -    |            Amplificador de audio             |     2.1     |
| TDA1516BQ |   B    | Amplificador de Potência para radio de carro | 24 ou 2x12  |
|  TDA2030  |   AB   |            hi-fi audio amplifier             |     14      |
|  TDA2040  |   AB   |         hi-fi audio power amplifier          |     25      |
|  TDA7294  |   AB   |     DMOS audio amplifier with mute/st-by     |     100     |
| TDA7370B  |   AB   | Amplificador de Potência para radio de carro |     10      |
|  TDA7374  |   AB   | Amplificador de Potência para radio de carro |     21      |
|  TDA8920  |   D    |    high efficiency audio power amplifier     |    2x100    |
|  TDA8922  |   D    |    high efficiency audio power amplifier     |    2x75     |

​		Após verificar todos os amplificadores, para o projeto foi escolhido o TDA7294, um aplificador de classe AB, possui uma boa potência e uma facilidade de implementação, pois como demonstra a figura abaixo, o próprio fabricante disponibiliza um circuito montado no datasheet.

<img src="C:\Users\ramon\Desktop\PI3\circuito amplificador.JPG" alt="circuito amplificador" style="zoom: 67%;" />

​		Com o circuito em mãos, foi utilizado o Software KiCad para montar o layout para impressão da placa.

<img src="C:\Users\ramon\Desktop\PI3\layout amplificador.JPG" alt="layout amplificador" style="zoom:80%;" />



### Microfone

​		Para o projeto, foi utilizado Microfone Stereo TM-ST1 em conjunto com o circuito amplificador para medir a resposta da caixa de som ao gerador de sinais.

​		De acordo com o Datasheet do fabricante, o microfone garante uma resposta linear entre as faixas de frequência entre 100Hz a 15kHz, sendo aceitável para o projeto pois a máxima frequência audível para os humanos é de 20kHz. 

<img src="C:\Users\ramon\Desktop\PI3\layout mic.JPG" alt="layout mic" style="zoom:80%;" />

​		O circuito pode ser dividido em 3 blocos, o primeiro sendo utilizado como buffer para o microfone, com o objetivo de aumentar a impedância de entrada. Seguindo por um amplificador inversor com ganho de 200 unidades (pensando em colocar um trimpot). E finalizando por um limitador de tensão para a entrada do ADC no microcontrolador, que possui limites de entrada de 2,4V a 3,6V.



### Código

```
/*
 * AD9833.cpp
 */

#include <AD9833.h>
#include <math.h>
#include <stdio.h>
#include "stm32f1xx_hal.h"

/* Inicia totalmente uma onda, frequencia e sua fase */
void AD9833 :: Init (int wave, double frequency, double phase) {
	SetWaveform (wave);
	SetFrequency (frequency);
	SetPhase (phase);
}

/* Determina uma frequencia especifica em Hz */
void AD9833 :: SetFrequency (double frequency) {

	if (frequency > 25e6)
		frequency = 25e6;
	if (frequency < 0) frequency = 0;

	freqWord = (((frequency*pow(2,28))/FMCLK)+1);
	FRQHW = ((freqWord & 0xFFFC000) >> 14);
	FRQLW = (freqWord & 0x3FFF);
	FRQLW |= 0x4000;
	FRQHW |= 0x4000;

	HAL_GPIO_WritePin(AD9833PORT, AD9833DATA, GPIO_PIN_SET);
	HAL_GPIO_WritePin(AD9833PORT, AD9833SCK, GPIO_PIN_SET);
	HAL_GPIO_WritePin(AD9833PORT, AD9833SS, GPIO_PIN_SET);
	HAL_GPIO_WritePin(AD9833PORT, AD9833SS, GPIO_PIN_RESET); //low = selected
	ASM_NOP();
	writeSPI(0x2100); // enable 16bit words and set reset bit
	writeSPI(FRQLW);
	writeSPI(FRQHW);
	writeSPI(0x2000); // clear reset bit
	ASM_NOP();
	HAL_GPIO_WritePin(AD9833PORT,AD9833SS,GPIO_PIN_SET); //high = deselected
	ASM_NOP();
	return;
}

/* Determina uma fase para o sinal, determinado como (4096/2π).Phase */
void AD9833 :: SetPhase (float Phase) {
	if(Phase<0)Phase=0;
	if(Phase>360)Phase=360;
	phaseVal  = ((int)(Phase*(4096/360)))|0XC000;  // 4096/360 = 11.37 change per Degree for Register And using 0xC000 which is Phase 0 Register Address

	 writeSPI(phaseVal);
}

/* Seleciona o tipo de onda, sendo 0 Senoidal, 1 Quadrada, 2 Triangular, 3 Meia onda quadrada*/
void AD9833 :: SetWaveform (int Wave) {
	  switch(Wave){
	  case 0:
	    HAL_GPIO_WritePin(AD9833PORT,AD9833SS,GPIO_PIN_RESET);
	    writeSPI(0x2000); // Valor para onda Senoidal
	    HAL_GPIO_WritePin(AD9833PORT,AD9833SS,GPIO_PIN_SET);
	    WKNOWN=0;
	    break;
	  case 1:
	    HAL_GPIO_WritePin(AD9833PORT,AD9833SS,GPIO_PIN_RESET);
	    writeSPI(0x2028); // Valor para onda Quadrada
	    HAL_GPIO_WritePin(AD9833PORT,AD9833SS,GPIO_PIN_SET);
	    WKNOWN=1;
	    break;
	  case 2:
	    HAL_GPIO_WritePin(AD9833PORT,AD9833SS,GPIO_PIN_RESET);
	    writeSPI(0x2002); // Valor para onda Triangular
	    HAL_GPIO_WritePin(AD9833PORT,AD9833SS,GPIO_PIN_SET);
	    WKNOWN=2;
	  case 3:
		HAL_GPIO_WritePin(AD9833PORT,AD9833SS,GPIO_PIN_RESET);
		writeSPI(0x2020); // Valor para meia onda quadrada
		HAL_GPIO_WritePin(AD9833PORT,AD9833SS,GPIO_PIN_SET);
		WKNOWN=2;
	    break;
	  default:
	    break;
	  }
}

/* Retorna a atual frequencia */
float AD9833 :: GetActualFrequency () {
	return ((freqWord-1)*FMCLK/pow(2,28));
}

/* Retorna a fase atual */
float AD9833 :: GetActualPhase () {
	return (phaseVal/(4096/360));
}

void AD9833 :: writeSPI(uint16_t word) {
	for (uint8_t i = 0; i < 16 ; i++) {
        if(word & 0x8000) HAL_GPIO_WritePin(AD9833PORT,AD9833DATA,GPIO_PIN_SET);   //bit=1, Set High
		else HAL_GPIO_WritePin(AD9833PORT,AD9833DATA,GPIO_PIN_RESET);        //bit=0, Set Low
		ASM_NOP();
		HAL_GPIO_WritePin(AD9833PORT,AD9833SCK,GPIO_PIN_RESET);             //Data is valid on falling edge
		ASM_NOP();
		HAL_GPIO_WritePin(AD9833PORT,AD9833SCK,GPIO_PIN_SET);
		word = word<<1; //Shift left by 1 bit
        }
	HAL_GPIO_WritePin(AD9833PORT,AD9833DATA,GPIO_PIN_RESET);                    //Idle low
	ASM_NOP();
}
```

