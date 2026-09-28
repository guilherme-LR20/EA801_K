# EA801_K
# Simulador de Sensor de Temperatura para Câmara Fria

**Projeto desenvolvido para a disciplina EA801 - Laboratório de Projetos de Sistemas Embarcados**  
**Instituição:** Faculdade de Engenharia Elétrica e de Computação (FEEC) - UNICAMP    
**Autor:** Guilherme Lopes Ribeiro   
**Data:** Setembro/2026  

---

##  Introdução
A eficiência energética é um tema cada vez mais central na indústria frigorífica, principalmente no que diz respeito às câmaras frias, pois estas são responsáveis por, aproximadamente, 70% dos gastos com energia elétrica em uma planta industrial. 

Dessa forma, sabe-se que quando a porta de uma câmara fica aberta para além do tempo necessário para carga ou descarga, o ar frio escapa por baixo, e o ar quente e húmido entra por cima. Tal cenário obriga o compressor a trabalhar na capacidade máxima e gera gelo no evaporador, destruindo a eficiência do sistema.

##  Objetivo do Projeto
Portanto, torna-se interessante, do ponto de vista econômico, a implementação de um sistema embarcado capaz de reduzir o custo de funcionamento de câmaras frigoríficas ao fechar a malha do sistema de controlo de temperatura. O projeto visa a eficiência energética do sistema e, além disso, impedir a proliferação bacteriana e possíveis embargos de produção e multas por parte dos agentes sanitários.

---

##  Metodologia de Projeto (TpM)
Para a realização deste projeto, foi utilizada a **TpM (Three Phase Methodology)**, que propõe a divisão do projeto em três fases fundamentais:

### 📌 Fase 1: Considerando o Negócio (Considering the Business)
* **Negócio:** Indústria frigorífica, mais especificamente a eficiência energética da câmara fria.
* **Regras do Negócio:** A indústria da carne brasileira apresenta uma margem de lucro muito pequena (entre 1% e 5%). Fatores que contribuem para a oscilação desta margem incluem o preço da arroba do boi, custos dos grãos para nutrição animal e o preço do dólar (considerando que o mercado externo, nomeadamente o asiático, é o destino principal).
* **Visão do Especialista:** Após o abate e corte, a carne é armazenada na câmara fria, que precisa de operar entre **0 °C e 4 °C** para carne vermelha, e próximo de 0 °C para aves e miudezas. A quebra destes limites resulta em embargo de produção, proliferação bacteriana e multas.
* **Parâmetros:** Medição constante de temperatura e humidade. Como as temperaturas rondam os 0 °C, é comum haver condensação de água, o que dificulta a leitura. Nesta validação inicial, o foco é a simulação da malha de temperatura.

---

###  Fase 2: Levantamento de Requisitos
Nesta fase, traduziram-se os requisitos do negócio para requisitos técnicos, utilizando a abordagem *Top-Down* (das camadas superiores para as inferiores):

| Camada | Descrição do Requisito |
| :--- | :--- |
| **L6 - Interface** | Exibição em tempo real da temperatura atual em °C e do estado do sistema para o operador local ("Normal", "Atenção" e "Alarme!"). |
| **L5 - Algoritmo** | Regras de conversão do sinal analógico e definição de faixas: <br>• **Normal:** 0 °C a 4 °C <br>• **Atenção:** -2 °C a 0 °C ou 4 °C a 6 °C <br>• **Alarme!:** < -2 °C ou > 6 °C. |
| **L4 - Memória** | Necessidade de um *buffer* temporário na memória RAM (1024 bytes) para manipular a imagem antes do envio para o ecrã. |
| **L3 - Borda** | Processamento local através do microcontrolador RP2040 para garantir uma resposta em tempo real. |
| **L2 - Conectividade**| Protocolo de comunicação interna I2C com frequência de 100 kHz e porta USB serial para depuração. |
| **L1 - Sensores** | Uso de um Joystick ADC como simulador analógico de temperatura, LED RGB para sinalização luminosa de estado e Buzzer para aviso sonoro de emergência. |

---

###  Fase 3: A Implementação
A implementação da tecnologia necessária para atender ao cliente foi realizada com base no modelo *Bottom-Up* (da base física para a interface):

#### **L1 - Sensores e Atuadores**
* **Joystick ADC:** Pino `GP26 (ADC0)`, utilizado para simular a temperatura.
* **LED RGB:** Pinos `GP11` (Verde), `GP12` (Azul) e `GP13` (Vermelho).
* **Buzzer:** Pino `GP21`, utilizado para emitir som no estado de "Alarme!".

#### **L2 - Conectividade**
Utilização do protocolo **I2C**:
* **SDA (Dados):** Pino `GP2`.
* **SCL (Clock):** Pino `GP3`.
* O RP2040 atua como *Master* e o ecrã OLED como *Slave* (Endereço `0x3C`). Velocidade configurada a 100 kHz, suportada por resistências de *pull-up* internas.

#### **L3 - Processamento na Borda (Edge Computing)**
Desenvolvido em **Linguagem C** com o Pico SDK. O código está dividido da seguinte forma:

<details>
<summary><b>1. Configurações Iniciais e Bibliotecas</b></summary>

```c
#include <stdio.h>
#include <stdlib.h>
#include "pico/stdlib.h"
#include "hardware/adc.h"
#include "hardware/i2c.h"
#include "ssd1306.h"

#define I2C_PORT i2c1
#define I2C_SDA 2
#define I2C_SCL 3
#define LED_G 11
#define LED_B 12
#define LED_R 13
#define BUZZER 21
#define JOYSTICK_Y_PIN 26
#define TEMP_MIN -3.0f
#define TEMP_MAX 9.0f
