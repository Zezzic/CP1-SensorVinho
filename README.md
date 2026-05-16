# Vinheria Agnello - Sistema de Monitoramento Ambiental com Arduino

[![Typing SVG](https://readme-typing-svg.demolab.com?font=Fjalla+One&weight=600&size=45&pause=1000&color=9A00F7&background=C5C5C5&center=true&vCenter=true&width=1100&height=100&lines=Sistema+de+Monitoramento;Ambiental+com+Arduino;C%2B%2B)](https://git.io/typing-svg)

![Status](https://img.shields.io/badge/status-Finalizado-green)
![Tipo](https://img.shields.io/badge/tipo-Projeto%20Acad%C3%AAmico-blue)
![Código](https://img.shields.io/badge/código-C%2B%2B-blue)
![Plataforma](https://img.shields.io/badge/plataforma-Arduino-blueviolet)

Projeto acadêmico desenvolvido na FIAP com foco em monitoramento ambiental aplicado ao armazenamento de vinhos. O sistema realiza leitura contínua de temperatura, umidade e luminosidade, exibe os valores em um display LCD 16x2 via I2C e aciona alertas visuais (LEDs) e sonoros (buzzer) quando os parâmetros saem das faixas ideais de conservação.

---

## Visão Geral do Circuito

> **Simulação disponível no Tinkercad:**
> [Acessar projeto](https://www.tinkercad.com/things/2WXkDtJZLoM-tinkercad-vinheria-agnello?sharecode=CgAH5GMNjIC6G_dgxCdvZmwMGxarI3vzUWCnvYRKrn4)

<img width="1469" height="491" alt="image" src="https://github.com/user-attachments/assets/5c5d3a9e-63b3-456a-a5ec-f491c143aeb9" />


---

## Funcionalidades Técnicas

- **Monitoramento contínuo:** Leitura analógica de luminosidade (LDR), temperatura (sensor TMP/NTC) e umidade (potenciômetro simulando DHT11).
- **Média de leituras:** Os valores exibidos no display são calculados como média de leituras acumuladas a cada 1 segundo, atualizados a cada 5 segundos.
- **Feedback visual:** Escalonamento de alertas por cores — Verde (OK), Amarelo (alerta), Vermelho (perigo).
- **Alerta sonoro:** Buzzer acionado continuamente em situações críticas de temperatura, umidade ou luminosidade.
- **Display LCD I2C:** Alternância automática entre três telas — luminosidade, temperatura e umidade.
- **Animação de abertura:** Sequência animada com caracteres customizados exibida no LCD ao inicializar o sistema.
- **Controle de tempo:** Uso de `millis()` para todas as temporizações, sem bloqueios com `delay()`.
- **Comunicação serial:** Monitoramento de dados em tempo real via Serial Monitor (9600 baud).

---

## Componentes Utilizados

| Componente | Descrição |
| :--- | :--- |
| Arduino Uno | Microcontrolador principal |
| LDR | Sensor de luminosidade analógico |
| Sensor TMP/NTC | Leitura analógica de temperatura |
| Potenciômetro | Simulação de umidade (substitui o DHT11 no Tinkercad) |
| LCD 16x2 com módulo I2C | Exibição de status e valores medidos |
| LED Verde | Sinalização de ambiente dentro dos parâmetros ideais |
| LED Amarelo | Sinalização de alerta |
| LED Vermelho | Sinalização de perigo |
| Buzzer | Emissão sonora de alerta |
| Resistores 220Ω | Proteção dos LEDs |
| Resistor 10kΩ | Divisor de tensão para o LDR |
| Protoboard | Estrutura para montagem do protótipo |

---

## Lógica do Sistema

O sistema monitora três parâmetros ambientais com as seguintes faixas de referência:

| Parâmetro | Faixa Ideal | Fora da Faixa |
| :--- | :--- | :--- |
| Temperatura | 10°C a 15°C | LED Amarelo + Buzzer |
| Umidade | 50% a 70% | LED Vermelho + Buzzer |
| Luminosidade (LDR) | < 300 (escuro) | 300–499: LED Amarelo / >= 500: LED Vermelho + Buzzer |

O display alterna entre três telas a cada ciclo de 5 segundos:

```
Tela 1 — Luminosidade       Tela 2 — Temperatura       Tela 3 — Umidade
──────────────────────       ──────────────────────       ──────────────────
Luminosidade OK              Temperatura OK               Umidade OK
Ambiente escuro              Temp. = 12.5°C               Umidade = 65%
```

---

## Como Simular no Tinkercad

1. Acesse o link da simulação: [tinkercad.com — Vinheria Agnello](https://www.tinkercad.com/things/2WXkDtJZLoM-tinkercad-vinheria-agnello?sharecode=CgAH5GMNjIC6G_dgxCdvZmwMGxarI3vzUWCnvYRKrn4)
2. Clique em **"Copiar e Editar"** para criar sua própria cópia editável (requer conta gratuita no Tinkercad).
3. Clique em **"Iniciar Simulação"** (botão verde no canto superior direito).
4. Interaja com os componentes para testar os diferentes estados do sistema:
   - Clique no **LDR** e arraste o controle deslizante para variar a luminosidade.
   - Clique no **potenciômetro** e gire o knob para simular variações de umidade.
   - Clique no **sensor de temperatura** e altere o valor para simular diferentes temperaturas.
5. Observe o **display LCD** alternando entre as três telas a cada 5 segundos e os **LEDs** e **buzzer** respondendo conforme os valores simulados.

---

## Execução Local (Arduino IDE)

1. Instale o [Arduino IDE](https://www.arduino.cc/en/software).
2. Instale a biblioteca **LiquidCrystal_I2C** via Gerenciador de Bibliotecas (`Ferramentas > Gerenciar Bibliotecas > buscar "LiquidCrystal I2C"` — autor: Frank de Brabander).
3. Clone ou baixe este repositório e abra o arquivo `vinheria_agnello.ino` no Arduino IDE.
4. Conecte o Arduino Uno via USB, selecione a placa e a porta em `Ferramentas` e faça o upload (`Ctrl+U`).
5. Abra o Serial Monitor para acompanhar as leituras em tempo real.

---

## Vídeo Explicativo

[Assista aqui](https://youtu.be/W0Ew7Kdyyu4)

---

## Equipe do Projeto

| Nome | RM | Responsabilidade |
| :--- | :--- | :--- |
| Diego Candido Stoianof | 570748 | Desenvolvimento |
| Felipe Moreira Mendes | 570807 | Desenvolvimento |
| Lucas Zezzi Custódio | 571161 | Desenvolvimento |
| Romulo Mendes Sousa | 570620 | Desenvolvimento |
