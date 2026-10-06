# STM32F401 - Interrupt, Timer and State Machine

Projeto desenvolvido durante meus estudos de sistemas embarcados utilizando o microcontrolador **STM32F401CEU6**, com foco em interrupções, timers e máquinas de estados.

##  Objetivo

Desenvolver um sistema com três modos de operação controlados através de um botão:

- **Modo 0 - Normal:** LED1 aceso
- **Modo 1 - Emergência:** LED2 aceso
- **Modo 2 - Recuperação:** LED3 aceso

Ao entrar no modo de recuperação, o sistema permanece nesse estado por aproximadamente **5 segundos** e retorna automaticamente para o modo normal.

Durante o modo de recuperação, novos acionamentos do SW1 são ignorados.

##  Hardware

- STM32F401CEU6
- Protoboard
- 1 botão
- 3 LEDs
- Resistores
- Jumpers

##  Software

- C
- STM32CubeIDE
- STM32CubeMX
- STM32 HAL

##  Conceitos praticados

- GPIO
- GPIO Input/Output
- Pull-Up
- EXTI (External Interrupt)
- NVIC
- Callbacks da HAL
- Máquina de estados
- Timers
- Interrupção de Timer
- Ponteiros e estruturas em C
- Flags
- Temporização sem `HAL_Delay()`

##  Configuração do TIM3

O TIM3 foi configurado para gerar uma interrupção a cada **100 ms**.

```c
Prescaler = 24999;
Period = 99;
## Configuração do TIM3

O TIM3 foi configurado para gerar uma interrupção a cada **100 ms**.

```c
Prescaler = 24999;
Period = 99;
```

Considerando um clock do timer de **25 MHz**:

```text
25.000.000 / (24999 + 1) = 1.000 Hz

1.000 Hz → 1 ms

(99 + 1) × 1 ms = 100 ms

50 × 100 ms = 5 segundos
```

## Interrupção do botão

A interrupção externa é tratada através da callback:

```c
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
```

O pino responsável pela interrupção é identificado através do parâmetro `GPIO_Pin`.

Exemplo:

```c
if (GPIO_Pin == SW1_Pin)
{
    if (modo == 0)
    {
        modo = 1;
    }
    else if (modo == 1)
    {
        modo = 2;
    }
}
```

## Interrupção do Timer

A interrupção do TIM3 é tratada através de:

```c
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
```

O timer responsável pela interrupção é identificado através de:

```c
if (htim->Instance == TIM3)
```

Durante o modo de recuperação, o contador é incrementado a cada interrupção:

```c
if (modo == 2)
{
    contador_tempo++;

    if (contador_tempo >= 50)
    {
        contador_tempo = 0;
        flag = 1;
    }
}
```

Após 50 interrupções de 100 ms, aproximadamente **5 segundos** são alcançados.

A `flag` informa ao programa principal que o tempo terminou:

```c
if (flag == 1)
{
    flag = 0;
    modo = 0;
}
```

## Estados e LEDs

| Estado | LED  | Função      |
| ------ | ---- | ----------- |
| 0      | LED1 | Normal      |
| 1      | LED2 | Emergência  |
| 2      | LED3 | Recuperação |

## Aprendizado

Este projeto faz parte dos meus estudos de **sistemas embarcados e desenvolvimento com STM32**.

O principal objetivo foi compreender na prática como utilizar **interrupções externas, interrupções de timer e máquinas de estados** para desenvolver um sistema orientado a eventos.

Também foi praticada a utilização de callbacks da HAL e a comunicação entre interrupções e o programa principal através de flags.

## Próximos passos

* Implementar debounce de botões
* Trabalhar com múltiplos timers
* Explorar prioridades de interrupção
* Aprofundar o uso de ponteiros e estruturas em C
* Iniciar estudos com FreeRTOS
* Desenvolver aplicações de IoT utilizando STM32

---

> **Nota:** Este repositório contém o arquivo `main.c` desenvolvido durante o estudo. A configuração de GPIO, EXTI e TIM3 foi realizada através do STM32CubeMX/STM32CubeIDE.
