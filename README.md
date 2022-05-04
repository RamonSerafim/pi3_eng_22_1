# pi3_eng_22_1
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
- Como o microcontrolador será conectado na caixa de som?



### Microfone

​		Necessita de um bloco controlador para manter a tensão da saída fixa.



### Microcontrolador

​		Utilizar o AD9833 para gerar os tipos de ondas sonoras.



### Caixa de som

​	Precisará utilizar filtros?

​	Como vai se conectar com o microcontroador?

​	Tensão de saída vai ser do próprio AD?



## Referências

https://github.com/BroeringFelipe/automatic-equalizer/blob/master/Relat%C3%B3rio%20do%20Projeto/Relat%C3%B3rio%20Final.pdf

https://www.audioreputation.com/how-to-measure-sound-quality-of-speakers/
