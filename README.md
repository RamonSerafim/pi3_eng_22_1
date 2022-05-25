Curso Superior em Engenharia Eletrônica

Unidade Curricular: Projeto Integrador III

Professor: Daniel Lohmann e Robinson Pizzio

Florianópolis, Abril de 2022 

Aluno: Ramon Busiquia Serafim



# Análise de caixas de som




## Introdução

​		Esse relatório tem a finalidade de analisar, simular, montar em bancada e explicar um circuito que faz a análise de determinados espectros de frequência afim de determinar se a caixa de som está funcionando perfeitamente em todas as faixas de frequência.

​		Utilizando diversos componentes como um microfone ajustado para o propósito aplicado, construindo um ambiente acústico controlado sem reverberação e um microcontrolador para emitir padrões de sinais, fazendo a análise posteriormente comparando com o sinal recebido/enviado.

​		Possui o intuito de verificar alguns problemas encarados quando passamos da simulação para a execução em bancada, possíveis discrepâncias nos resultados entre ambas.





### Metodologia

​		Com o escopo do projeto definido, para fazer a análise semanalmente foi eleita algumas dúvidas para pesquisa, sendo apresentadas abaixo. Primeiro não devemos pensar na solução final, mas sim separar os problemas em partes e apresentar todas as possibilidades e posteriormente verificar qual será melhor parar o projeto.

- Como vai ser feita a captura do microfone?
- Onde vamos analisar a onda captada?
- Vai ser comparada com a onda emitida (original)?
- Como e qual microcontrolador será conectado na caixa de som?




### Microfone

​		Necessita de um bloco controlador para manter a tensão da saída fixa.



### Microcontrolador

​		Foi feito um primeiro questionamento sobre qual usar. Ao detalhar o projeto, seria necessário montar todos os blocos em algum microcontrolador, porém o Professor Lohmann discutiu a possibilidade de utilizar um DSP, microprocessadores direcionados para realizar processamento digital de sinais. 

​		Com isso, debatendo com o Professor Pacheco que leciona aula de Processamento Digitais de Sinais II a principal questão imposta foi qual seria o método de análise do projeto, pois algumas análises só poderiam ser feitas com um DSP, como uma transformada de Fourier. Outros microcontroladores como ARM possuem ponto fixo, não podendo realizar essas análises.

​		O intuito do projeto é visualizar e comparar intensidade de ondas RMS, e isso pode ser feito com um ARM, não sendo preciso utilizar um DSP. Portanto, foi definido que para este projeto será utilizado um STM32F103C8.

#### STM32F103C8 

​		Microcontrolador mais conhecido como Blue Pill atende todos os pré-requisitos de projeto, utilizando o software STM32CubeMX é possível configurar os pinos de forma mais simples, facilitando o desenvolvimento do projeto e disponibiliza a biblioteca HAL (Hardware Abstraction Layer).




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

​		

​		Após verificar todos os amplificadores, para o projeto foi escolhido o TDA7294, um aplificador de classe AB, possui uma boa potência e uma facilidade de implementação, pois como demonstra a figura abaixo, o próprio fabricante disponibiliza um circuito montado no datasheet.


## Referências

https://github.com/BroeringFelipe/automatic-equalizer/blob/master/Relat%C3%B3rio%20do%20Projeto/Relat%C3%B3rio%20Final.pdf

https://www.audioreputation.com/how-to-measure-sound-quality-of-speakers/

https://technologyreviewer.com/measure-speaker-sound-quality/

https://www.st.com/resource/en/datasheet/tda7294.pdf
