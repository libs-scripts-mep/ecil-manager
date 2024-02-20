# Ecil Menager

## Documentação 

- Manual Cappo 10 Plus 
  - documentation\Manual-Cappo-10-Plus.pdf
  
- Protocolo de Comunicação Cappo 10 Plus
  - documentation\Protocolo-Comunicação-Cappo-10-Plus.pdf


## Protocolo

### Template Requisição 

    [IDNAME] [Instruction] [DATA1] [DATA2] [DATA3] [DATA4] [CHKSUM]


#### IDNAME 

    O IDNAME é o valor que foi configurado no processo '10.1 - Digital interface data program mode' 
    descrita no documento 'Protocolo-Comunicação-Cappo-10-Plus.pdf'

#### Instruction 

    Existem varias funções que podem ser utilizadas no cappo ecil, elas podem ser observadas no 
    documento 'Protocolo-Comunicação-Cappo-10-Plus.pdf', mas as mais utilizadas são:

    | Instrução                        | Valor | Tipo  |
    | -------------------------------- | ----- | ----- |
    | Leitura dos valor                | 0x18  | READ  |
    | Seta o tipo de sensor            | 0x19  | WRITE |
    | Seta configurações de calibração | 0x1A  | WRITE |
    | Seta valores de temperatura      | 0x1B  | WRITE |

#### DATAS

    Os datas são configurados dependendo da função escolhida, o que deve ser escrito nos datas pode ser 
    encontrada no documento 'Protocolo-Comunicação-Cappo-10-Plus.pdf', segue exemplo das intruções mais 
    utilizadas:


    0x18:

        Descrição: Na leitura dos valores de temperatura os 'datas' devem ser preenchidos com '0x00'.  

        Exemplo: 0x01 0x18 0x00 0x00 0x00 0x00 0x00 // le valores da entrada do cappo.

        Resposta: 
    

    0x19:

        Descrição: Na escrita do tipo de sensor o valor manipulado deve ser o [DATA1], o restante deve ser 
        preenchido com 0x00, a baixo temos a tabala dos principais sensores com seus valores. 

        OBS: O chksum também deve ser manipulado.

            | Sensor | Valor de escrita |
            | ------ | ---------------- |
            | J      | 0x00             |
            | K      | 0x01             |
            | PT100  | 0x0E             |

        Exemplo: 0x01 0x19 0x01 0x00 0x00 0x00 0x01 // seta cappo em tipo K

        Resposta: 


    0x1A:

        Descrição: Na escrita das configurações do cappo o valor a ser manipulado é o [DATA1], para 
        manipulação desse byte é utilizado um sistema de soma de bits, ou seja caso o objetivo seja 
        adicionar duas configurações, será a soma dos valores dessas configurações a ser mandado pelo 
        [DATA1], segue os pricipais valores na tabela a baixo.

        OBS: O chksum também deve ser manipulado.

            | Configuração | Valor Habilitado    | Valor Desabilitado      |
            | ------------ | ------------------- | ----------------------- |
            | Compensação  | 0x10 (Rj ext)       | 0x00 (Rj int)           |
            | ITS          | 0x08 (ITS90)        | 0x00 (ITS68)            |
            | Escala       | 0x40 (°F)           | 0x00 (°C)               |
            | Out/In       | 0x20 (OUT)          | 0x00 (IN)               |
            | Casa Decimal | 0x00 0x01 0x02 0x03 | 0x04 (Sem casa decimal) |
                                    

        Exemplo: 0x01 0x1A 0x38 0x00 0x00 0x00 0x38 
        // Desabilia compensação, seta cappo em saída e habilita ITS90

        Resposta: 


    0x1B:

        Descrição: Na escrita do valor de temperatura os valores a serem manipulados são os [DATA1]
        e o [DATA2], caso o valor de chksum passe de 0x7F será manipulado o valor [DATA3] e refeito o calculo do chksum com o novo valor do [DATA3], o passo
        a passo dessa manipulação pode ser encontrada no documento 
        'Protocolo-Comunicação-Cappo-10-Plus.pdf', e podemos observar abaixo um exemplo de 
        manipulação:

        let DATA3 = 256 - parseInt(checkSum, 16)
        DATA3 = dataAux.toString(16)
        checkSum = [DATA1] + [DATA2] + [DATA3] + [DATA4]

        OBS: O chksum também deve ser manipulado.
                                    
        Exemplo 1: 0x01 0x1B 0x01 0x2C 0x00 0x00 0x2D  
        // Simulando 300°

        Exemplo 2: 0x01 0x1B 0x02 0xEE 0x10 0x00 0x00  
        // Simulando 750° (sendo necessário manipular o [DATA3])

        Outra questão é a manipulação de valores negativos, que deve seguir os seguintes passos:

        1. Transformar o numero decimal em binario e retirar o sinal negativo
        2. Inverter o número binario, trocando 0 por 1 e vice-versa
        3. Realiza a soma com +1 no número binario    
        4. Converter em Hexadecimal
        5. Enviar para o cappo 
        6. Recalcular o checksum
   
        Exemplo 3: 0x01 0x1B 0xFF 0x42 0x00 0x00 0x41 // Simulando -190°

        Resposta: 

#### CHKSUM

