# README #

This README would normally document whatever steps are necessary to get your application up and running.

### What is this repository for? ###

* Quick summary
* Version
* [Learn Markdown](https://bitbucket.org/tutorials/markdowndemo)

### How do I get set up? ###

* Summary of set up
* Configuration
* Dependencies
* Database configuration
* How to run tests
* Deployment instructions


1) Antes de começar, removi a mesa da Rostock, porque fica mais fácil trabalhar no plano da própria base dela. No caso da Kossell é indicado colocar um vidro já que ela não tem base, ou manter a mesa desde que corretamente paralela aos perfis da base. Para desabilitar a mesa é necessário alterar o seguinte

#define TEMP_SENSOR_BED 0

2) Para testar se os motores estavam conectados direito e testar os endstops, instalei o firmware como cartesiana e pelo Pronterface movi cada motor separadamente. Os movimentos X- Y- e Z- movimentaram cada carriage da respectiva coluna para baixo e X+ Y+ Z+, para cima. Se algum motor seu fez movimento diferente, corrija. Enquanto fazia essa operação percebi que não dá para confiar nos dados "padrão" do cálculo dos steps das 3 colunas. No meu caso (polia de 20 dentes, correia GT2, 1/16 de micropasso nos drivers X, Y e Z) o valor calculado devia ser 80, mas ao "mandar andar" o Z, por ex. 50mm, ele andava 55

Como testar uma impressora delta como cartesiana? Comente essas duas linhas e suba novamente o firmware.
//#define DELTA
//#define ENABLE_AUTO_BED_LEVELING

Como descobrir se o "step" das colunas está correto? Peguei o paquímetro e medi a distância do carriage até o topo. Mandei mover o carriage da mesma coluna 50mm e comparei a nova medição com a antiga. Colocando na calculadora do Prusa[4] os dados medidos cheguei ao novo valor, 71.65, que coloquei em

#define DEFAULT_AXIS_STEPS_PER_UNIT   {71.65, 71.65, 71.65, 458}

Com essa mudança os passos ficaram corretos.

ATENÇÃO: O número 458 se refere a relação engrenagem/polia/micropasso do extrusor, no meu caso referente a deste thingiverse[5] e 16 micropassos. Se você usa outra engrenagem, o valor vai ser diferente. Foge ao objetivo deste artigo explicar isso, mas as instruções do cálculo podem ser achadas aqui[6].

OBS: Se a sua plataforma parecer andar "torta" (não ficar paralela a base quando estiver fora do centro), COM CERTEZA os seus "steps" estão errados. Perdi bastante tempo com esse detalhe!!!

3) É bom frisar que os endstops são a parte mais importante de uma impressora delta (bem, junto com os braços serem do exato mesmo tamanho). Os endstops devem estar funcionando perfeitamente, sendo instalados em cima e devidamente conectados na placa na posição de "MAX" dos três eixos. Eles devem permanecer abertos na operação normal e fechados quando o carriage estiver na posição "HOME". Teste com o multímetro se estão assim antes de continuar.

Para uso de endstops mecânicos, é necessário ligar ENDSTOPPULLUPS:

#define ENDSTOPPULLUPS

Deltas só usam endstops de máximo:

#define DISABLE_MIN_ENDSTOPS
//#define DISABLE_MAX_ENDSTOPS

O homing move na direção dos endstops de máximo:

#define X_HOME_DIR 1
#define Y_HOME_DIR 1
#define Z_HOME_DIR 1

Através do comando gcode M119 voce pode observar se os endstops foram reconhecidos e se aparecem "OPEN" na operação normal. Segure cada um deles e repita o M119 para verificar se apareceu "TRIGGERED". É um bom momento para conferir se eles estão ligados nos locais corretos (o endstop do X fica na coluna da esquerda, o endstop do Y na coluna da direita e o do Z na coluna traseira).

4) Agora vamos começar com a configuração de delta, propriamente dita. Para mudar de cartesiana para delta precisamos informar algumas coisas no firmware. Parti dos valores padrão (você deve colocar o ponto decimal nos itens abaixo para que o número seja considerado tipo float pelo compilador do arduino):

#define DELTA
#define ENABLE_AUTO_BED_LEVELING

Valores padrão da Rostock Mini:

#define DELTA_DIAGONAL_ROD 186.0
#define DELTA_SMOOTH_ROD_OFFSET 139.5
#define DELTA_EFFECTOR_OFFSET 33.0
#define DELTA_CARRIAGE_OFFSET 22.0

Valores padrão da Mini Kossel:

#define DELTA_DIAGONAL_ROD 186.0 
#define DELTA_SMOOTH_ROD_OFFSET 128.0 
#define DELTA_EFFECTOR_OFFSET 19.9 
#define DELTA_CARRIAGE_OFFSET 19.5 

5) Sobre o tamanho e posicionamento da mesa, a Rostock Mini tem espaço para uma mesa com cerca de 180mm mas o normal é 160mm. Usei 150mm na configuração para não imprimir colado na borda da minha manta aquecedora. 150mm / 2 são os 75 informados abaixo (X varia de -75 a 75, total 150, Y varia de -75 a 75, total 150mm, lembrando que a área de impressão da delta é REDONDA)

#define DELTA_RADIUS 75

#define X_MAX_POS DELTA_RADIUS
#define X_MIN_POS -DELTA_RADIUS
#define Y_MAX_POS DELTA_RADIUS
#define Y_MIN_POS -DELTA_RADIUS
#define Z_MAX_POS MANUAL_Z_HOME_POS
#define Z_MIN_POS 0

#define MANUAL_HOME_POSITIONS
//#define BED_CENTER_AT_0_0

Não deixaremos o usuário tentar sair do diâmetro da área de impressão para evitar danos aos braços (prevenção nunca é demais):

#define min_software_endstops true
#define max_software_endstops true

O valor do Z_HOME_POS (abaixo) em 196 foi calculada com base na altura do meu bico. O ideal é usar a própria impressora para achar o valor correto, fazendo homing (atraves do gcode G28) e baixando o Z manualmente de 10 em 10mm e depois de 1 em 1mm e 0.1 em 0.1mm até ele tocar na base. Veja a posição final com o gcode M114 e copie o valor correto para o campo. Quando estiver tudo pronto, vai precisar refazer esse passo levando em conta a instalação da mesa aquecida!

#define MANUAL_Z_HOME_POS 196

ATENÇÃO: Por "tocar" me refiro a colocar um cartão na base ou na mesa, o bico encostar nele e você não conseguir puxá-lo sem fazer força. 

6) As acelerações padrões são um pouco altas. Sugiro começar com valores menores e alterar quando a impressora estiver com os movimentos calibrados

#define HOMING_FEEDRATE {900, 900, 900, 0}
#define DEFAULT_MAX_FEEDRATE          {1200, 1200, 1200, 200}
#define DEFAULT_MAX_ACCELERATION      {1500,1500,1500,1500}
#define DEFAULT_ACCELERATION          1200
#define DEFAULT_ZJERK                 DEFAULT_XYJERK

7) Se o seu Marlin tiver o código para M665, M666 e M251 (ou seja, for razoavelmente recente), pode ligar a EEPROM agora retirando o comentário da linha correspondente. Caso contrario deixe para fazer quando terminar. Habilitar o "chitchat" inclui o comando M503, que é bem útil, então procure deixa-lo ligado sempre, tanto em deltas quanto em cartesianas.

//#define EEPROM_SETTINGS
#define EEPROM_CHITCHAT

8) Agora que terminamos as configurações iniciais, compile/suba o firmware e conecte na impressora pelo Pronterface para continuarmos com a calibração

Pelo Pronterface, execute o HOME (G28) e em seguida desça o bico até o Z tocar na base. Anote a posição com o gcode M114 (se você ajustou corretamente o MANUAL_Z_HOME_POS antes, vai ser zero)

Agora execute outro HOME (G28) e mova o bico usando o Pronterface até a plataforma ficar com os braços paralelos as barras verticais da coluna do X (a esquerda). Anote a posição com o GCODE M114. Na minha ficou com os valores abaixo (na sua vai ser com certeza diferente!)

G28
G0 X-71 Y-38 Z-1.1

Agora execute outro HOME (G28) e mova o bico da mesma forma até os braços ficarem paralelos as barras da coluna Y (a da direita). Anote com o M114. Na minha ficou

G28
G0 X69 Y-38 Z-3.7

Finalmente faça o mesmo (G28, mover) até a coluna Z (atrás). Anote com M114. Na minha ficou

G28
G0 X-4 Y83 Z0.4

Se o seu Marlin possui o comando M666, você pode gravar os valores que achou (no meu caso, -1.1, -3.7 e 0.4, no seu caso com certeza serão outros!) ao invés de ajustar manualmente os parafusos dos endstops nos carriages. O comando para isso é (se não tiver habilitado a EPROM precisa executar o comando toda vez que ligar a impressora)

M666 X-1.1 Y-3.7 Z0.4

Se tiver habilitado a EPROM, grave com o gcode M500.

Para saber os valores atuais use o gcode M503

Finalmente, faça um novo home geral (G28) e visite novamente as colunas X, Y e Z usando o Pronterface. O bico deve ficar na mesma distância dessa vez. Se você não tem o comando M666, ajuste o parafuso no carriage de contato com cada endstop e repita o passo 8 novamente até que o movimento das 3 colunas encontrar a mesma altura do Z

9) Agora vem a parte complicada. Faça um novo home (G28) e desça o bico devagar até tocar na base (o lance do cartão novamente!). Se ele chegou lá com Z=0, parabéns!!! Compre um bilhete de loteria agora mesmo!

Caso negativo, vai ser necessário alterar o DELTA_RADIUS. Se o bico tocou na mesa abaixo de 0, o seu DELTA_RADIUS está muito alto, diminua a linha abaixo em alguns milímetros, recompile e repita todo o processo de regulagem das colunas do passo 8.

#define DELTA_SMOOTH_ROD_OFFSET 139.5

Se o bico ficou distante da mesa com Z=0, o seu DELTA_RADIUS está baixo, é necessario AUMENTAR a linha do DELTA_SMOOTH_ROD_OFFSET, recompilar e repetir todo o processo de regulagem das colunas do passo 8.

10) Agora é hora de por a mesa e mover o bico novamente até as 3 colunas, de forma a que a mesa esteja na mesma altura nos três pontos. A minha mesa ficou com 18,8mm. Diminuido do MANUAL_Z_HOME_POS (196) o novo valor ficou 177.2. Agora, ao digitar o gcode G0 Z0 o bico se posiciona no centro da mesa na altura correta

// esse valor vai variar COM A SUA MESA E BICO
#define MANUAL_Z_HOME_POS 177.2

11) Falta agora imprimir a peça de calibração. A sugestão do Minow é o seguinte openscad (stl em anexo):

cube([100,2,2],center=true);

Após imprimir, meça o comprimento do objeto. Se ele estiver errado, será necessário alterar o DELTA_DIAGONAL_ROD usando a seguinte fórmula:

novo_valor = 100 / valor medido em mm * DELTA_DIAGONAL_ROD anterior

Coloque o novo valor (lembrando que precisa do ponto decimal mesmo que seja .0)

#define DELTA_DIAGONAL_ROD novo_valor

recompile, suba novamente o firmware e repita o passo 11.

ANEXO: Como usar o Repetier-Host com delta.

1. Entre no Repetier-Host e clique no botão das engrenagens "Config. da Impressora"
2. Digite "Delta" no campo "Impressora".
3. Clique na aba "Forma da Impressora".
4. Em Printer Type escolha "Rostock Printer"
5. Informe 0 em Home X e Home Y, e escolha MAX em Home Z.
6. Nos campos Printable RADIUS e Printable Height, coloque os mesmos valores de DELTA_RADIUS e MANUAL_Z_HOME_POS que colocou no firmware.
7. Clique em Aplique/OK e está pronto!