
# PicoEMP - Descrição das Funcionalidades e Limites Operacionais

## Frequência de Pulsos por Segundo

A frequência máxima de geração de pulsos pelo PicoEMP é influenciada por fatores como a duração do pulso (`pulse_time`), o tempo de recarga do circuito de alta tensão e as configurações específicas do dispositivo. Embora o código não especifique um limite exato, é importante considerar que o circuito de carga de alta tensão pode necessitar de tempo para recarregar entre os pulsos. Portanto, a taxa de repetição dos pulsos pode ser limitada a alguns pulsos por segundo para garantir a operação segura e eficaz do dispositivo.

---

## Modos `hvp_internal` e `hvp_external`

- **`hvp_internal`:** Quando ativado, o PicoEMP utiliza sua fonte interna de alta tensão para gerar os pulsos eletromagnéticos. Isso é configurado chamando a função `picoemp_configure_pulse_output()` e definindo a variável `hvp_internal` como `true`. A tensão interna gerada é de aproximadamente 250V.

- **`hvp_external`:** Neste modo, o dispositivo é configurado para utilizar uma fonte externa de alta tensão. Isso é feito chamando a função `picoemp_configure_pulse_external()` e definindo `hvp_internal` como `false`. Ao utilizar uma fonte externa, é crucial garantir que a tensão fornecida esteja dentro das especificações de segurança e compatibilidade do dispositivo para evitar danos ou riscos.

---

## Função `fast_trigger` e Limites dos Parâmetros

- **Função `fast_trigger`:** Esta função utiliza a Máquina de Estados Programável (PIO) do Raspberry Pi Pico para gerar pulsos com alta precisão temporal. Ela configura o PIO para emitir um pulso com base nos parâmetros `pulse_delay_cycles` (número de ciclos de atraso antes do pulso) e `pulse_time_cycles` (duração do pulso em ciclos).

- **Limites dos Parâmetros:**
  - **`pulse_delay_cycles`:** Este parâmetro define o número de ciclos de clock a serem aguardados antes de gerar o pulso. O valor máximo depende da largura do registrador utilizado para armazenar esse valor. Se for um registrador de 32 bits, o valor máximo seria 2³² - 1 (4.294.967.295). Valores muito altos podem não ser práticos devido às limitações de tempo real e ao comportamento do hardware.
  
  - **`pulse_time_cycles`:** Define a duração do pulso em ciclos de clock. Semelhante ao `pulse_delay_cycles`, o limite máximo depende da capacidade do registrador (por exemplo, 2³² - 1 para um registrador de 32 bits). É importante configurar esse valor de acordo com a frequência de operação do clock do sistema para obter a duração de pulso desejada.

---

## Tempo Máximo para `config_pulse_time` em Microssegundos

A função `config_pulse_time` define a duração do pulso em microssegundos. Embora o código fornecido não especifique um limite máximo explícito, a duração do pulso é limitada pela capacidade do hardware e pela segurança operacional. Pulsos muito longos podem sobrecarregar o circuito e potencialmente causar danos. Recomenda-se consultar a documentação oficial do PicoEMP ou realizar testes controlados para determinar o tempo máximo seguro para a sua aplicação específica.

---

## Valores para `config_pulse_power`

A função `config_pulse_power` configura a potência do pulso. No código fornecido, a potência padrão é definida como 0,0122 (presumivelmente em watts ou uma unidade relativa). Os valores que podem ser atribuídos a `pulse_power` devem estar dentro das capacidades do hardware para evitar sobrecarga ou danos. É aconselhável começar com valores baixos e aumentar gradualmente, monitorando o comportamento do dispositivo e consultando a documentação técnica para garantir que os limites de potência não sejam excedidos.

---

## Função `toggle_gp1`

- **Funcionamento:** A função `toggle_gp1` alterna o estado do pino GPIO 1 do Raspberry Pi Pico. Isso é realizado utilizando a função `gpio_xor_mask(1<<1)`, que inverte o estado atual do pino (se estiver em nível alto, muda para baixo, e vice-versa).

- **Utilização:** Esta função pode ser utilizada para depuração ou para controlar periféricos externos conectados ao pino GPIO 1. Por exemplo, pode ser usada para ligar ou desligar um LED, acionar um relé ou enviar sinais de controle para outros dispositivos. É importante garantir que o hardware conectado ao pino GPIO 1 seja compatível com os níveis de tensão e corrente fornecidos pelo Raspberry Pi Pico para evitar danos ao microcontrolador ou ao dispositivo externo.

---

Para informações mais detalhadas e específicas, recomenda-se consultar a documentação oficial do PicoEMP e do Raspberry Pi Pico, bem como os repositórios e manuais fornecidos pelos desenvolvedores. [Documentação Oficial](https://github.com/newaetech/chipshouter-picoemp)




# Comandos Disponíveis no Console Serial do PicoEMP

## `cmd_arm` (Código: 0)

**Função:** Ativa o estado de prontidão do PicoEMP, preparando-o para gerar pulsos eletromagnéticos.

**Implementação:** Chama a função `arm()`, que acende o LED indicador de carregamento (`PIN_LED_CHARGE_ON`) e define a variável `armed` como `true`.

---

## `cmd_disarm` (Código: 1)

**Função:** Desativa o estado armado do PicoEMP, interrompendo o carregamento de alta tensão e colocando o dispositivo em modo seguro.

**Implementação:** Chama a função `disarm()`, que apaga o LED de carregamento e define `armed` como `false`. Além disso, desativa o PWM chamando `picoemp_disable_pwm()`.

---

## `cmd_pulse` (Código: 2)

**Função:** Dispara um pulso eletromagnético único, desde que o dispositivo esteja previamente armado.

**Implementação:** Invoca `picoemp_pulse(pulse_time)`, que gera um pulso com a duração especificada pela variável `pulse_time`.

---

## `cmd_status` (Código: 3)

**Função:** Retorna o estado atual do PicoEMP, incluindo informações sobre se está armado, níveis de tensão e prontidão para disparo.

**Implementação:** Chama `get_status()`, que verifica o estado das variáveis `armed`, `PIN_IN_CHARGED`, `timeout_active` e `hvp_internal`, combinando essas informações em um único valor retornado.

---

## `cmd_enable_timeout` (Código: 4)

**Função:** Ativa o temporizador de segurança que desarma automaticamente o dispositivo após um período de inatividade.

**Implementação:** Define `timeout_active` como `true` e atualiza o tempo limite chamando `update_timeout()`.

---

## `cmd_disable_timeout` (Código: 5)

**Função:** Desativa o temporizador de segurança, permitindo que o dispositivo permaneça armado indefinidamente até ser desarmado manualmente.

**Implementação:** Define `timeout_active` como `false`.

---

## `cmd_fast_trigger` (Código: 6)

**Função:** Inicia um disparo rápido utilizando a máquina de estados programável (PIO) para gerar pulsos com alta precisão temporal.

**Implementação:** Chama a função `fast_trigger()`, que configura o PIO para gerar um pulso com os parâmetros `pulse_delay_cycles` e `pulse_time_cycles`.

---

## `cmd_config_pulse_delay_cycles` (Código não especificado)

**Função:** Configura o número de ciclos de atraso antes do disparo do pulso.

**Implementação:** Atualiza a variável `pulse_delay_cycles` com o valor recebido via FIFO.

---

## `cmd_config_pulse_time_cycles` (Código não especificado)

**Função:** Define a duração do pulso em ciclos de clock.

**Implementação:** Atualiza `pulse_time_cycles` com o valor recebido.

---

## `cmd_internal_hvp` (Código não especificado)

**Função:** Configura o PicoEMP para utilizar a fonte de alta tensão interna.

**Implementação:** Chama `picoemp_configure_pulse_output()` e define `hvp_internal` como `true`.

---

## `cmd_external_hvp` (Código não especificado)

**Função:** Configura o dispositivo para utilizar uma fonte de alta tensão externa.

**Implementação:** Chama `picoemp_configure_pulse_external()` e define `hvp_internal` como `false`.

---

## `cmd_config_pulse_time` (Código não especificado)

**Função:** Define a duração do pulso em microssegundos.

**Implementação:** Atualiza a variável `pulse_time` com o valor recebido.

---

## `cmd_config_pulse_power` (Código não especificado)

**Função:** Configura a potência do pulso.

**Implementação:** Atualiza `pulse_power` com o valor recebido, utilizando uma união para conversão entre `float` e `uint32_t`.

---

## `cmd_toggle_gp1` (Código não especificado)

**Função:** Alterna o estado do pino GPIO 1, podendo ser utilizado para funcionalidades adicionais ou depuração.

**Implementação:** Utiliza `gpio_xor_mask(1<<1)` para inverter o estado atual do pino.






# ChipSHOUTER-PicoEMP

[![CC BY-SA 3.0][cc-by-sa-shield]][cc-by-sa]

![](hardware/picoemp-red.jpeg)

The PicoEMP is a low-cost Electromagnetic Fault Injection (EMFI) tool, designed *specifically* for self-study and hobbiest research. Under the safety shield it looks like this:

![](hardware/picoemp.jpeg)

You can see some details of the design in the [Intro Video](https://www.youtube.com/watch?v=nB5arJi-tVE).

## Thanks / Contributors

PicoEMP is a community-focused project, with major contributions from:
* Colin O'Flynn (original HW design, simple Python demo)
* [stacksmashing](https://twitter.com/ghidraninja) (C firmware for full PIO feature-set)
* [Lennert Wouters](https://twitter.com/LennertWo) (C improvements, first real demo)
* [@nilswiersma](https://github.com/nilswiersma) (Triggering/C improvements)

## Background

The [ChipSHOUTER](http://www.chipshouter.com) is a high-end Electromagnetic Fault Injection (EMFI) tool designed by Colin
at [NewAE Technology](http://www.newae.com). While not the first commercially available EMFI tool, ChipSHOUTER was the first
"easily purchasable" (even if expensive) tool with extensive open documentation. The tool was *not* open-source, but it
did contain a variety of detailed description of the design and architecture in the
[User Manual](https://github.com/newaetech/ChipSHOUTER/tree/master/documentation). The ChipSHOUTER design optimization focused in rough order on (1) safe operation, (2) high performance, (3) usability, and finally (4) cost. This results in a tool that covers many use-cases, but may be overkill (and too costly) for many. In additional, acquiring the safety testing/certification is not cheap, and must be accounted for in the product sale price.

The PicoEMP tries to fill in the gap that ChipSHOUTER leaves at the lower end of the spectrum. This PicoEMP project is *not* the
ChipSHOUTER. Instead it's designed to present a "bare bones" tool that has a design optimization focused in rough order of (1) safe
operation, (2) cost, (3) usability, (4) performance. Despite the focus on safety and low-cost, it works *suprisingly* well. It is also
*not* sold as a complete product - you are responsible for building it, ensuring it meets any relevant safety requirements/certifications,
and we completely disclaim all liability for what happens next. Please **only** use PicoEMP where you are building and controlling it
yourself, with total understanding of the operation and risks. It is *not* designed to be used in professional or educational environments,
where tools are expected to meet safety certifications (ChipSHOUTER was designed for these use-cases).

As an open-source project it also collects inputs from various community members, and welcomes your contributions! It also has various remixes of it, including:

* TODO link to people's remixes.

## Building a PicoEMP

The PicoEMP uses a Raspberry Pi Pico as the controller, inspired by @nezza using it for the debug-n-dump tool. You could alternatively use an Arduino or another microcontroller. You basically just need a few things:

1. PWM output to drive HV transformer.
2. Pulse pin to generate a pulse.
3. Status pin to monitor the HV status.

You have two options for building the PicoEMP: (1) total scratch build, or (2) easy-assemble build.

### Scratch Build

The PCB is *mostly* one layer. Original versions of it were milled on a Bantam PCB mill, and the final 'production' version is designed
to still allow this simple milling process. You can find details in the [gerbers](hardware/gerbers) folder, including Bantam-optimized files
which remove some of the smaller vias (used for the mounting holes), and require you to surface-mount the Raspberry Pi Pico. Here was
'rev3' of the PCB with a few hacked up tests:

![](hardware/design_notes/img/proto_rev3_hackedup.jpeg)

If you've got time you can order the "real" PCBs from the [gerbers](hardware/gerbers) as well.

The BOM and build details are described in the [hardware](hardware) folder. If you cannot find the plastic shield (the upper half of Hammond
1551BTRD is used), you can find a simple 3D-printable shield as well. The official shield is low-cost and available from Digikey/Mouser/
Newark so you can purchase alongside everything else you need.

**IMPORTANT**: The plastic shield is critical for safe operation. While the output itself is isolated from the input connections, you will still **easily shock yourself** on the exposed high-voltage capacitor and circuitry. **NEVER** operate the device without the shield.

### Easy-Assemble Build

The Easy-Assembly build uses a "mostly complete" SMD board, which you need to solder a Raspberry Pi Pico, switches, and through-hole headers. Currently it's available only on the [NewAE Store](https://store.newae.com/chipshouter-picoemp). We're working to
get this listed on Mouser for much cheaper worldwide shipping (the NewAE store doesn't get great rates & due to issues with Canada's postal system for international shipments quotes mostly via DHL).

### Programming the PicoEMP

You'll need to program the PicoEMP with the firmware in the [firmware](firmware) directory. You can run other tasks on the microcontroller
as well.

### Building the EM Injection Tip (Probe / Coil)

You will also need an "injection tip", typically made with a ferrite core and some wires wrapped around it. You can see examples of such cores in the ChipSHOUTER kit. The following shows a few homemade & commercial tips:

![](hardware/injection_tips/examples/tips-sma.jpg)

You can make your own from suitable SMA connectors, magnet wire, and a ferrite core material. See the [injection_tips](hardware/injection_tips)
folder for more examples and details on building the probes.

*Reader Note: Please submit your own examples with a pull-request to this repo, it would be great to have more examples of probe geometries*

You can find additional examples of homemade cores in research papers such as:

* A. Cui, R. Housley, "BADFET: Defeating Modern Secure Boot Using Second-Order Pulsed Electromagnetic Fault Injection," USENIX Workshop on Offensive Technologies (WOOT 17), 2017.  [Paper Link.](https://www.usenix.org/conference/woot17/workshop-program/presentation/cui) [Slides Link.](https://github.com/RedBalloonShenanigans/BADFET)
* J. Balasch, D. Arumí and S. Manich, "Design and validation of a platform for electromagnetic fault injection," 2017 32nd Conference on Design of Circuits and Integrated Systems (DCIS), 2017, pp. 1-6. [Paper Link.](https://upcommons.upc.edu/bitstream/handle/2117/116688/bare_conf.pdf)
* J. Toulemont, G. Chancel, J. M. Galliere, F. Mailly, P. Nouet and P. Maurine, "On the scaling of EMFI probes," 2021 Workshop on Fault Detection and Tolerance in Cryptography (FDTC), 2021. [Paper Link.](https://ieeexplore.ieee.org/abstract/document/9565575) [Slides Link.](https://jaif.io/2021/media/JAIF2021%20-%20Toulemont.pdf)
* LimitedResults. "Enter the Gecko," 2021. [Blog Link](https://limitedresults.com/2021/06/enter-the-efm32-gecko/)
* C. Gaine, J-P. Nikolovski, D. Aboulkassimi, J-M. Dutertre. "New probe design for hardware characterization by ElectroMagnetic Fault Injection," EMC Europe 2022 [Paper Link.] (https://hal-cea.archives-ouvertes.fr/cea-03657852/file/article_archivesouvertes.pdf)
* 
### Useful References

If you don't know where to start with FI, you may find a couple chapters of the [Hardware Hacking Handbook](https://nostarch.com/hardwarehacking) useful.

You can see a demo of PicoEMP being used on a real attack in this [TI CC SimpleLink attack demo](https://github.com/KULeuven-COSIC/SimpleLink-FI/blob/main/notebooks/5_ChipSHOUTER-PicoEMP.ipynb).

## Using the PicoEMP

The general usage of the PicoEMP is as follows:

1. Press the "ARM" button. The red "ARMING" led will come on instantly telling you it's trying to charge the high voltage.
2. The red "HV" led will come on after a few seconds saying it is charged to "some voltage".
3. Place the probe tip overtop of the target.
4. Press the "Pulse" button.

You can see more examples of this in the [Intro Video](https://www.youtube.com/watch?v=nB5arJi-tVE).

You can even use the Raspberry Pi Pico to attack a Raspberry Pi "regular"! Here's a demo hitting a RSA signature on a Raspberry Pi (the demo code taken from Colin's [Remoticon 2021 Talk](https://github.com/colinoflynn/remoticon-2021-levelup-hardware-hacking/tree/master/rpi-glitching)):

![](hardware/demo.jpg)

**WARNING**: The high voltage will be applied across the SMA connector. If an injection tip (coil) is present, it will absorb most of the power. If you leave the SMA connector open, you will present a high voltage pulse across this SMA and could shock yourself. Do NOT touch the output SMA tip as a general "best practice", and treat the output as if it has a high voltage present.

The full ChipSHOUTER detects the missing connector tip and refuses to power up the high voltage, the PicoEMP does not have this failsafe!

## About the High Voltage Isolation

Most EMFI tools generate high voltages (similar to a camera flash). Many previous designs of open-source EMFI tools would work well, but [exposed the user to high voltages](https://github.com/RedBalloonShenanigans/BADFET). This was fine provided you use the tool correctly, but of course there is always a risk of grabbing the electrically "hot" tool! This common design choice happens because the easiest way to design an EMFI tool is with "low-side switching" (there is a very short mention of these design choices as well in my [book](https://www.nostarch.com/hardwarehacking) if you are curious). With low-side switching the output connector is always "hot", which presents a serious shock hazard.

PicoEMP gets around this problem by floating the high-voltage side, meaning there is no electrical path between the EMFI probe output and the input voltage ground. With the isolated high voltage output we can use the simple "low-side switching" in a safe manner. Some current will still flow due to the high-frequency spikes, so this isn't *perfect*, but it works well enough in practice (well enough you will shock yourself less often).

The caveat here is for this to work you also need to isolate your gate drive. There are a variety of [solutions to this](https://www.analog.com/en/technical-articles/powering-the-isolated-side-of-your-half-bridge-configuration.html), with the simplist being a gate drive transformer (GDT). The PicoEMP uses the transformer architecture, with some simplifications to further reduce BOM count.

More details of the design are available in the [hardware](hardware) folder.

### Hipot Testing for Validating Isolation

Easy-assemble builds have been subject to a hipot test. This test validates the isolation exists, and has not been compromised by things like leftover flux on the PCB.

This test applies a high voltage (1000V) from the SMA connector pads to the low-voltage signals shorted together. The test is done at 1000V DC, with test passing if LESS than 1 uA of current flows over the 60 seconds test duration. Note this limits is *far* lower than most industry standard limits.

### Technical Differences between ChipSHOUTER and PicoEMP

The main differences from a technical standpoint:

* ChipSHOUTER uses a much more powerful high voltage circuit and transformer (up to ~30W vs ~0.2W) that gives it
  almost unlimited glitch delivery, typically limited by your probe tip. The PicoEMP is slower to recover, typically ~1 to 4 seconds between
  glitches.

* ChipSHOUTER has a larger internal energy storage & more powerful output drivers.

* ChipSHOUTER has a controlled high-voltage setting from 150V to 500V. PicoEMP generates ~250V, there is some feedback but it's uncalibrated.
  **NOTE**: The PicoEMP allows some control of output pulse size by instead controlling the drive signal. This is less reliable (more variability
  in the output), but meets the goal of using the lowest-cost control method.

## License

This work is licensed under a [Creative Commons Attribution-ShareAlike 3.0 International License][cc-by-sa].

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/3.0/
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/3.0/88x31.png
[cc-by-sa-shield]: https://img.shields.io/badge/License-CC%20BY--SA%203.0-lightgrey.svg

ChipSHOUTER is a trademark of NewAE Technology Inc., registered in the US, European Union, and other jurisdictions.
PicoEMP is a trademark of NewAE Technology Inc.
