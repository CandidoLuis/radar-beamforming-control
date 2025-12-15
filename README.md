# radar-beamforming-control
Control module for radar beamforming and calibration modes
Segue um **README.md** académico, pronto para colocar no repositório.

---

# Radar Beamforming Control Module

## Descrição

Este repositório contém o **módulo de controlo de um sistema de radar com beamforming**, desenvolvido no contexto de um trabalho académico de modernização de um radar multicanal. O módulo é responsável pela gestão dos modos de operação, controlo dos phase shifters, calibração local e suporte a técnicas de beamsteering e Direction of Arrival (DoA).

## Objetivos

* Implementar controlo centralizado do sistema de radar
* Suportar modos de **calibração** e **operação automática**
* Garantir coerência de fase entre canais
* Permitir experimentação em beamsteering (Tx) e DoA (Rx)

## Funcionalidades

* Controlo dos phase shifters por canal
* Gestão dos modos de operação:

  * Modo de calibração
  * Modo automático
* Interface com os módulos de transmissão (Tx) e receção (Rx)
* Suporte à configuração dinâmica de fases
* Estrutura preparada para expansão multicanal

## Estrutura do repositório

```
radar-beamforming-control/
├── src/          # Código-fonte do módulo de controlo
├── include/      # Headers e definições
├── docs/         # Documentação técnica
└── README.md
```

## Integração no sistema

O módulo de controlo comunica com:

* **Módulo Tx:** configuração de fases para beamsteering
* **Módulo Rx:** gestão de calibração e aquisição para DoA
* **Sintetizador PLL e restantes periféricos RF**

## Requisitos

* Plataforma de controlo embebida (microcontrolador ou SBC)
* Interfaces SPI/GPIO
* Ambiente de compilação compatível com C/C++

## Estado do desenvolvimento

* ✔ Estrutura base definida
* ✔ Controlo de fase implementado
* ⚠ Módulo Rx em fase de otimização (phase shifter CH2, LNA CH3)

## Trabalho futuro

* Finalizar a reparação do módulo Rx
* Refinar o algoritmo de calibração automática
* Integrar algoritmos de DoA
* Validar o sistema completo em ambiente experimental

## Contexto académico

Este projeto foi desenvolvido no âmbito de uma dissertação de mestrado em Engenharia Eletrónica e Telecomunicações, com foco na modernização de sistemas de radar e técnicas de beamforming.

## Licença

Uso académico e de investigação.
