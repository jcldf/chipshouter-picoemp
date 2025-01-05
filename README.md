

## 1. PicoEMP – Descrição das Funcionalidades e Limites Operacionais

Nesta seção estão reunidas as principais características de operação do PicoEMP, incluindo frequência de pulsos, modos de alta tensão interna e externa, parâmetros de configuração e considerações de segurança.

### 1.1. Frequência de Pulsos por Segundo

A frequência máxima de geração de pulsos pelo PicoEMP depende de três fatores principais:  
1. A duração do pulso (`pulse_time`).  
2. O tempo de recarga do circuito de alta tensão.  
3. As configurações específicas do dispositivo.

Embora o código não imponha um limite exato, o circuito de carga de alta tensão necessita de um intervalo adequado para recarregar entre pulsos. Isso faz com que a taxa de repetição de pulsos geralmente se limite a alguns pulsos por segundo, de modo a garantir a operação segura e confiável do dispositivo.

---

### 1.2. Modos `hvp_internal` e `hvp_external`

Existem dois modos de operação referentes à fonte de alta tensão:

- **`hvp_internal`:**  
  Quando este modo está ativo, o PicoEMP utiliza sua fonte interna de alta tensão (aprox. 250V) para gerar pulsos eletromagnéticos. Para habilitá-lo, chama-se a função `picoemp_configure_pulse_output()` com a variável `hvp_internal = true`.

- **`hvp_external`:**  
  Neste modo, configura-se o PicoEMP para usar uma fonte externa de alta tensão, chamando-se `picoemp_configure_pulse_external()` e ajustando `hvp_internal = false`. É fundamental garantir que a tensão externa esteja dentro das especificações de segurança e compatibilidade do dispositivo para evitar danos ou riscos.

---

### 1.3. Função `fast_trigger` e Limites dos Parâmetros

#### 1.3.1. Função `fast_trigger`

A função `fast_trigger` utiliza a Máquina de Estados Programável (PIO) do Raspberry Pi Pico para gerar pulsos com alta precisão temporal. Ela configura o PIO para emitir um pulso baseado em dois parâmetros principais:

- `pulse_delay_cycles`: número de ciclos de clock de atraso antes do pulso.  
- `pulse_time_cycles`: duração do pulso em ciclos de clock.

#### 1.3.2. Limites dos Parâmetros

- **`pulse_delay_cycles`:**  
  Define o número de ciclos de clock a aguardar antes de gerar o pulso. O valor máximo depende da largura do registrador que armazena essa informação. Se for 32 bits, então o limite teórico é 2³² - 1 (4.294.967.295). Na prática, valores muito altos podem ser pouco úteis devido a limitações de tempo real e do hardware.

- **`pulse_time_cycles`:**  
  Determina a duração do pulso em ciclos de clock. Assim como em `pulse_delay_cycles`, o valor máximo depende da capacidade do registrador. Deve-se configurar este valor de acordo com a frequência de operação do clock para obter a duração de pulso desejada.

---

### 1.4. Tempo Máximo para `config_pulse_time` em Microssegundos

A função `config_pulse_time` define a duração do pulso em microssegundos. Embora o código não estabeleça um limite máximo explícito, a duração do pulso é restringida pela capacidade do hardware e pelas condições seguras de operação. Pulsos muito longos podem causar sobrecarga do circuito e possíveis danos. É recomendável consultar a documentação oficial do PicoEMP ou realizar testes controlados para identificar o tempo máximo seguro para cada aplicação específica.

---

### 1.5. Valores para `config_pulse_power`

O ajuste de potência do pulso é feito pela função `config_pulse_power`. No código fornecido, o valor padrão é 0,0122 (unidade não especificada, possivelmente watts ou alguma escala relativa). Os valores possíveis para `pulse_power` devem respeitar os limites de projeto do hardware, sob risco de sobrecarga ou danos ao dispositivo. Aconselha-se iniciar com valores baixos e aumentar gradativamente, monitorando o comportamento do sistema e consultando as especificações técnicas para evitar exceder o limite de potência suportado.

---

### 1.6. Função `toggle_gp1`

- **Objetivo:**  
  A função `toggle_gp1` alterna o estado do pino GPIO 1 do Raspberry Pi Pico, usando `gpio_xor_mask(1<<1)`. Assim, se o pino estiver em nível alto, ele passa a baixo, e vice-versa.

- **Aplicações:**  
  Essa funcionalidade pode ser útil para depuração (debug) ou controle de periféricos externos ligados ao pino GPIO 1, como LEDs, relés ou outros dispositivos de controle. Antes de conectar hardware externo ao pino, verifique a compatibilidade de tensão e corrente para evitar danos ao Pico ou ao dispositivo externo.

---

### 1.7. Documentação e Referências

Para mais detalhes e informações específicas, consulte:  
- [Documentação Oficial do PicoEMP](https://github.com/newaetech/chipshouter-picoemp)  
- [Documentação e repositórios do Raspberry Pi Pico](https://www.raspberrypi.com/documentation/microcontrollers/)

---

## 2. Comandos Disponíveis no Console Serial do PicoEMP

O PicoEMP possui diversos comandos que podem ser enviados via console serial, cada um com um código associado e uma funcionalidade específica. A seguir, listam-se os principais comandos:

### 2.1. `cmd_arm` (Código: 0)

- **Função:** Ativa o estado de prontidão do PicoEMP para gerar pulsos eletromagnéticos.  
- **Implementação:**  
  - Chama a função `arm()`.  
  - Acende o LED indicador de carregamento (`PIN_LED_CHARGE_ON`).  
  - Define a variável `armed = true`.

---

### 2.2. `cmd_disarm` (Código: 1)

- **Função:** Desativa o estado “armado” do PicoEMP, interrompendo o carregamento da alta tensão e colocando o dispositivo em um modo seguro.  
- **Implementação:**  
  - Chama a função `disarm()`.  
  - Apaga o LED de carregamento.  
  - Define `armed = false`.  
  - Desativa o PWM chamando `picoemp_disable_pwm()`.

---

### 2.3. `cmd_pulse` (Código: 2)

- **Função:** Dispara um pulso eletromagnético único, desde que o dispositivo já esteja armado.  
- **Implementação:**  
  - Invoca `picoemp_pulse(pulse_time)`, gerando um pulso com duração especificada pela variável `pulse_time`.

---

### 2.4. `cmd_status` (Código: 3)

- **Função:** Retorna o estado atual do PicoEMP, incluindo informação sobre se está armado, níveis de tensão e prontidão para disparo.  
- **Implementação:**  
  - Chama `get_status()`, que verifica as variáveis `armed`, `PIN_IN_CHARGED`, `timeout_active` e `hvp_internal`.  
  - Combina essas informações em um valor de retorno único.

---

### 2.5. `cmd_enable_timeout` (Código: 4)

- **Função:** Ativa um temporizador de segurança, que desarma o dispositivo após um intervalo de inatividade.  
- **Implementação:**  
  - Define `timeout_active = true`.  
  - Atualiza o tempo limite chamando `update_timeout()`.

---

### 2.6. `cmd_disable_timeout` (Código: 5)

- **Função:** Desabilita o temporizador de segurança, permitindo que o dispositivo permaneça armado indefinidamente até ser desarmado manualmente.  
- **Implementação:**  
  - Define `timeout_active = false`.

---

### 2.7. `cmd_fast_trigger` (Código: 6)

- **Função:** Inicia um disparo rápido utilizando a máquina de estados programável (PIO) para gerar pulsos com alta precisão temporal.  
- **Implementação:**  
  - Chama `fast_trigger()`, configurando o PIO para gerar o pulso a partir dos parâmetros `pulse_delay_cycles` e `pulse_time_cycles`.

---

### 2.8. `cmd_config_pulse_delay_cycles` (Código não especificado)

- **Função:** Configura o número de ciclos de atraso antes do disparo do pulso.  
- **Implementação:**  
  - Atualiza `pulse_delay_cycles` com o valor obtido via FIFO.

---

### 2.9. `cmd_config_pulse_time_cycles` (Código não especificado)

- **Função:** Define a duração do pulso em ciclos de clock.  
- **Implementação:**  
  - Atualiza `pulse_time_cycles` com o valor recebido.

---

### 2.10. `cmd_internal_hvp` (Código não especificado)

- **Função:** Configura o PicoEMP para utilizar a fonte interna de alta tensão.  
- **Implementação:**  
  - Chama `picoemp_configure_pulse_output()`.  
  - Define `hvp_internal = true`.

---

### 2.11. `cmd_external_hvp` (Código não especificado)

- **Função:** Configura o dispositivo para usar uma fonte de alta tensão externa.  
- **Implementação:**  
  - Chama `picoemp_configure_pulse_external()`.  
  - Define `hvp_internal = false`.

---

### 2.12. `cmd_config_pulse_time` (Código não especificado)

- **Função:** Define a duração do pulso em microssegundos.  
- **Implementação:**  
  - Atualiza a variável `pulse_time` com o valor recebido.

---

### 2.13. `cmd_config_pulse_power` (Código não especificado)

- **Função:** Configura a potência do pulso.  
- **Implementação:**  
  - Atualiza `pulse_power` com o valor recebido, usando uma união para converter entre `float` e `uint32_t`.

---

### 2.14. `cmd_toggle_gp1` (Código não especificado)

- **Função:** Alterna o estado do pino GPIO 1, podendo ser usado para debug ou funcionalidades adicionais.  
- **Implementação:**  
  - Utiliza `gpio_xor_mask(1<<1)` para inverter o estado atual do pino.

---

## 3. ChipSHOUTER-PicoEMP

A seguir, apresenta-se o texto original (em inglês) sobre o projeto PicoEMP (baseado no ChipSHOUTER), incluindo detalhes de hardware, histórico, instruções de montagem, exemplos de sondas de injeção e referências adicionais.

---

### 3.1. Overview *(Texto Original em Inglês)*

> [![CC BY-SA 3.0][cc-by-sa-shield]][cc-by-sa]  
> ![](hardware/picoemp-red.jpeg)  
> The PicoEMP is a low-cost Electromagnetic Fault Injection (EMFI) tool, designed *specifically* for self-study and hobbiest research. Under the safety shield it looks like this:
> 
> ![](hardware/picoemp.jpeg)
> 
> You can see some details of the design in the [Intro Video](https://www.youtube.com/watch?v=nB5arJi-tVE).

#### 3.1.1. Thanks / Contributors

> PicoEMP is a community-focused project, with major contributions from:  
> - Colin O'Flynn (original HW design, simple Python demo)  
> - [stacksmashing](https://twitter.com/ghidraninja) (C firmware for full PIO feature-set)  
> - [Lennert Wouters](https://twitter.com/LennertWo) (C improvements, first real demo)  
> - [@nilswiersma](https://github.com/nilswiersma) (Triggering/C improvements)

---

#### 3.1.2. Background

> The [ChipSHOUTER](http://www.chipshouter.com) is a high-end Electromagnetic Fault Injection (EMFI) tool designed by Colin at [NewAE Technology](http://www.newae.com). While not the first commercially available EMFI tool, ChipSHOUTER was the first "easily purchasable" (even if expensive) tool with extensive open documentation. The tool was *not* open-source, but it did contain a variety of detailed description of the design and architecture in the [User Manual](https://github.com/newaetech/ChipSHOUTER/tree/master/documentation). The ChipSHOUTER design optimization focused in rough order on (1) safe operation, (2) high performance, (3) usability, and finally (4) cost. This results in a tool that covers many use-cases, but may be overkill (and too costly) for many. In additional, acquiring the safety testing/certification is not cheap, and must be accounted for in the product sale price.
>
> The PicoEMP tries to fill in the gap that ChipSHOUTER leaves at the lower end of the spectrum. This PicoEMP project is *not* the ChipSHOUTER. Instead it's designed to present a "bare bones" tool that has a design optimization focused in rough order of (1) safe operation, (2) cost, (3) usability, (4) performance. Despite the focus on safety and low-cost, it works *suprisingly* well. It is also *not* sold as a complete product - you are responsible for building it, ensuring it meets any relevant safety requirements/certifications, and we completely disclaim all liability for what happens next. Please **only** use PicoEMP where you are building and controlling it yourself, with total understanding of the operation and risks. It is *not* designed to be used in professional or educational environments, where tools are expected to meet safety certifications (ChipSHOUTER was designed for these use-cases).
>
> As an open-source project it also collects inputs from various community members, and welcomes your contributions! It also has various remixes of it, including:
>
> * TODO link to people's remixes.

---

### 3.2. Building a PicoEMP *(Texto Original em Inglês)*

> The PicoEMP uses a Raspberry Pi Pico as the controller, inspired by @nezza using it for the debug-n-dump tool. You could alternatively use an Arduino or another microcontroller. You basically just need a few things:
>
> 1. PWM output to drive HV transformer.  
> 2. Pulse pin to generate a pulse.  
> 3. Status pin to monitor the HV status.

#### 3.2.1. Scratch Build

> The PCB is *mostly* one layer. Original versions of it were milled on a Bantam PCB mill, and the final 'production' version is designed to still allow this simple milling process. You can find details in the [gerbers](hardware/gerbers) folder, including Bantam-optimized files which remove some of the smaller vias (used for the mounting holes), and require you to surface-mount the Raspberry Pi Pico. Here was 'rev3' of the PCB with a few hacked up tests:
>
> ![](hardware/design_notes/img/proto_rev3_hackedup.jpeg)
>
> If you've got time you can order the "real" PCBs from the [gerbers](hardware/gerbers) as well.
>
> The BOM and build details are described in the [hardware](hardware) folder. If you cannot find the plastic shield (the upper half of Hammond 1551BTRD is used), you can find a simple 3D-printable shield as well. The official shield is low-cost and available from Digikey/Mouser/Newark so you can purchase alongside everything else you need.
>
> **IMPORTANT**: The plastic shield is critical for safe operation. While the output itself is isolated from the input connections, you will still **easily shock yourself** on the exposed high-voltage capacitor and circuitry. **NEVER** operate the device without the shield.

#### 3.2.2. Easy-Assemble Build

> The Easy-Assembly build uses a "mostly complete" SMD board, which you need to solder a Raspberry Pi Pico, switches, and through-hole headers. Currently it's available only on the [NewAE Store](https://store.newae.com/chipshouter-picoemp). We're working to get this listed on Mouser for much cheaper worldwide shipping (the NewAE store doesn't get great rates & due to issues with Canada's postal system for international shipments quotes mostly via DHL).

#### 3.2.3. Programming the PicoEMP

> You'll need to program the PicoEMP with the firmware in the [firmware](firmware) directory. You can run other tasks on the microcontroller as well.

---

### 3.3. Building the EM Injection Tip *(Texto Original em Inglês)*

> You will also need an "injection tip", typically made with a ferrite core and some wires wrapped around it. You can see examples of such cores in the ChipSHOUTER kit. The following shows a few homemade & commercial tips:
>
> ![](hardware/injection_tips/examples/tips-sma.jpg)
>
> You can make your own from suitable SMA connectors, magnet wire, and a ferrite core material. See the [injection_tips](hardware/injection_tips) folder for more examples and details on building the probes.
>
> *Reader Note: Please submit your own examples with a pull-request to this repo, it would be great to have more examples of probe geometries*
>
> You can find additional examples of homemade cores in research papers such as:
>
> - A. Cui, R. Housley, "BADFET: Defeating Modern Secure Boot Using Second-Order Pulsed Electromagnetic Fault Injection," USENIX Workshop on Offensive Technologies (WOOT 17), 2017.  [Paper Link.](https://www.usenix.org/conference/woot17/workshop-program/presentation/cui) [Slides Link.](https://github.com/RedBalloonShenanigans/BADFET)  
> - J. Balasch, D. Arumí and S. Manich, "Design and validation of a platform for electromagnetic fault injection," 2017 32nd Conference on Design of Circuits and Integrated Systems (DCIS), 2017, pp. 1-6. [Paper Link.](https://upcommons.upc.edu/bitstream/handle/2117/116688/bare_conf.pdf)  
> - J. Toulemont, G. Chancel, J. M. Galliere, F. Mailly, P. Nouet and P. Maurine, "On the scaling of EMFI probes," 2021 Workshop on Fault Detection and Tolerance in Cryptography (FDTC), 2021. [Paper Link.](https://ieeexplore.ieee.org/abstract/document/9565575) [Slides Link.](https://jaif.io/2021/media/JAIF2021%20-%20Toulemont.pdf)  
> - LimitedResults. "Enter the Gecko," 2021. [Blog Link](https://limitedresults.com/2021/06/enter-the-efm32-gecko/)  
> - C. Gaine, J-P. Nikolovski, D. Aboulkassimi, J-M. Dutertre. "New probe design for hardware characterization by ElectroMagnetic Fault Injection," EMC Europe 2022 [Paper Link.](https://hal-cea.archives-ouvertes.fr/cea-03657852/file/article_archivesouvertes.pdf)

---

### 3.4. Useful References *(Texto Original em Inglês)*

> If you don't know where to start with FI, you may find a couple chapters of the [Hardware Hacking Handbook](https://nostarch.com/hardwarehacking) useful.
>
> You can see a demo of PicoEMP being used on a real attack in this [TI CC SimpleLink attack demo](https://github.com/KULeuven-COSIC/SimpleLink-FI/blob/main/notebooks/5_ChipSHOUTER-PicoEMP.ipynb).

---

### 3.5. Using the PicoEMP *(Texto Original em Inglês)*

> The general usage of the PicoEMP is as follows:
>
> 1. Press the "ARM" button. The red "ARMING" led will come on instantly telling you it's trying to charge the high voltage.
> 2. The red "HV" led will come on after a few seconds saying it is charged to "some voltage".
> 3. Place the probe tip overtop of the target.
> 4. Press the "Pulse" button.
>
> You can see more examples of this in the [Intro Video](https://www.youtube.com/watch?v=nB5arJi-tVE).
>
> You can even use the Raspberry Pi Pico to attack a Raspberry Pi "regular"! Here's a demo hitting a RSA signature on a Raspberry Pi (the demo code taken from Colin's [Remoticon 2021 Talk](https://github.com/colinoflynn/remoticon-2021-levelup-hardware-hacking/tree/master/rpi-glitching)):
>
> ![](hardware/demo.jpg)
>
> **WARNING**: The high voltage will be applied across the SMA connector. If an injection tip (coil) is present, it will absorb most of the power. If you leave the SMA connector open, you will present a high voltage pulse across this SMA and could shock yourself. Do NOT touch the output SMA tip as a general "best practice", and treat the output as if it has a high voltage present.
>
> The full ChipSHOUTER detects the missing connector tip and refuses to power up the high voltage, the PicoEMP does not have this failsafe!

---

### 3.6. About the High Voltage Isolation *(Texto Original em Inglês)*

> Most EMFI tools generate high voltages (similar to a camera flash). Many previous designs of open-source EMFI tools would work well, but [exposed the user to high voltages](https://github.com/RedBalloonShenanigans/BADFET). This was fine provided you use the tool correctly, but of course there is always a risk of grabbing the electrically "hot" tool! This common design choice happens because the easiest way to design an EMFI tool is with "low-side switching" (there is a very short mention of these design choices as well in my [book](https://www.nostarch.com/hardwarehacking) if you are curious). With low-side switching the output connector is always "hot", which presents a serious shock hazard.
>
> PicoEMP gets around this problem by floating the high-voltage side, meaning there is no electrical path between the EMFI probe output and the input voltage ground. With the isolated high voltage output we can use the simple "low-side switching" in a safe manner. Some current will still flow due to the high-frequency spikes, so this isn't *perfect*, but it works well enough in practice (well enough you will shock yourself less often).
>
> The caveat here is for this to work you also need to isolate your gate drive. There are a variety of [solutions to this](https://www.analog.com/en/technical-articles/powering-the-isolated-side-of-your-half-bridge-configuration.html), with the simplist being a gate drive transformer (GDT). The PicoEMP uses the transformer architecture, with some simplifications to further reduce BOM count.
>
> More details of the design are available in the [hardware](hardware) folder.

#### 3.6.1. Hipot Testing for Validating Isolation

> Easy-assemble builds have been subject to a hipot test. This test validates the isolation exists, and has not been compromised by things like leftover flux on the PCB.
>
> This test applies a high voltage (1000V) from the SMA connector pads to the low-voltage signals shorted together. The test is done at 1000V DC, with test passing if LESS than 1 uA of current flows over the 60 seconds test duration. Note this limit is *far* lower than most industry standard limits.

---

### 3.7. Technical Differences Between ChipSHOUTER and PicoEMP *(Texto Original em Inglês)*

> The main differences from a technical standpoint:
>
> - ChipSHOUTER uses a much more powerful high voltage circuit and transformer (up to ~30W vs ~0.2W) that gives it almost unlimited glitch delivery, typically limited by your probe tip. The PicoEMP is slower to recover, typically ~1 to 4 seconds between glitches.
> - ChipSHOUTER has a larger internal energy storage & more powerful output drivers.
> - ChipSHOUTER has a controlled high-voltage setting from 150V to 500V. PicoEMP generates ~250V, there is some feedback but it's uncalibrated.  
>   **NOTE**: The PicoEMP allows some control of output pulse size by instead controlling the drive signal. This is less reliable (more variability na saída), mas atende ao objetivo de custo reduzido.

---

### 3.8. License *(Texto Original em Inglês)*

> This work is licensed under a [Creative Commons Attribution-ShareAlike 3.0 International License][cc-by-sa].
>
> [cc-by-sa]: http://creativecommons.org/licenses/by-sa/3.0/  
> [cc-by-sa-image]: https://licensebuttons.net/l/by-sa/3.0/88x31.png  
> [cc-by-sa-shield]: https://img.shields.io/badge/License-CC%20BY--SA%203.0-lightgrey.svg  
>
> ChipSHOUTER is a trademark of NewAE Technology Inc., registered in the US, European Union, and other jurisdictions.  
> PicoEMP is a trademark of NewAE Technology Inc.

---

### 3.9. Observações Finais

- A parte em inglês, referente ao projeto e à filosofia do PicoEMP, foi mantida integralmente para preservar as informações originais.  
- Recomenda-se a leitura cuidadosa de todos os avisos de segurança antes de montar ou operar o dispositivo.  
- O uso do PicoEMP demanda conhecimento de eletrônica de alta tensão e suas implicações. Qualquer modificação ou uso indevido é de inteira responsabilidade do usuário.

---

