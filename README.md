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

## Código Principal

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// LCD I2C 
LiquidCrystal_I2C lcd(0x27, 16, 2);

// Pinos 
const int PINO_TEMP   = A0;
const int PINO_UMID   = A1;
const int PINO_LDR    = A2;
const int LED_VERDE   = 8;
const int LED_AMARELO = 9;
const int LED_VERM    = 10;
const int BUZZER      = 7;

// Faixas ideais 
const float TEMP_MIN = 10.0;
const float TEMP_MAX = 15.0;
const float UMID_MIN = 50.0;
const float UMID_MAX = 70.0;
const int LUZ_ESCURO = 300;
const int LUZ_MEIA   = 500;

// Temporizador 
const unsigned long INTERVALO_LEITURA = 1000;
const unsigned long INTERVALO_DISPLAY = 5000;

unsigned long ultimaLeitura = 0;
unsigned long ultimoDisplay = 0;

float somaTemp = 0;
float somaUmid = 0;
long  somaLuz  = 0;
int   contagem = 0;

float tempMedia = 0;
float umidMedia = 0;
int   luzMedia  = 0;

int telaAtual = 0;

// Animacao 
bool animacaoAtiva         = true;
int  frameAnimacao         = 0;
unsigned long tempoAnteriorFrame = 0;

const unsigned long DURACAO_FRAMES[] = {
  600,   // frame 0: tacas bem separadas
  500,   // frame 1: tacas se aproximando
  400,   // frame 2: tacas bem proximas
  1200,  // frame 3: BRINDE com faiscas
  200,   // frame 4: faiscas apagam (pisca 1)
  200,   // frame 5: faiscas voltam (pisca 1)
  200,   // frame 6: faiscas apagam (pisca 2)
  600    // frame 7: faiscas voltam (final)
};
const int TOTAL_FRAMES = sizeof(DURACAO_FRAMES) / sizeof(DURACAO_FRAMES[0]);

// Chars customizados 
byte tacaEsq[8] = {
  B00011, B00111, B00111, B00011,
  B00001, B00001, B00111, B00000
};
byte tacaDir[8] = {
  B11000, B11100, B11100, B11000,
  B10000, B10000, B11100, B00000
};
byte tacaEsqBrinde[8] = {
  B00001, B00011, B00111, B00011,
  B00010, B00100, B01000, B00000
};
byte tacaDirBrinde[8] = {
  B10000, B11000, B11100, B11000,
  B01000, B00100, B00010, B00000
};
byte faiscas[8] = {
  B00100, B10001, B01010, B00000,
  B01010, B10001, B00100, B00000
};

void desenharFrameAnimacao(int frame) {
  switch (frame) {
    case 0:
      lcd.clear();
      lcd.setCursor(3, 0);  lcd.write(byte(0));
      lcd.setCursor(12, 0); lcd.write(byte(1));
      lcd.setCursor(4, 1);  lcd.print("CP02");
      break;
    case 1:
      lcd.clear();
      lcd.setCursor(5, 0);  lcd.write(byte(0));
      lcd.setCursor(10, 0); lcd.write(byte(1));
      lcd.setCursor(4, 1);  lcd.print("CP02");
      break;
    case 2:
      lcd.clear();
      lcd.setCursor(6, 0); lcd.write(byte(0));
      lcd.setCursor(9, 0); lcd.write(byte(1));
      lcd.setCursor(4, 1); lcd.print("CP02");
      break;
    case 3:
      lcd.clear();
      lcd.setCursor(5, 0);  lcd.write(byte(2));
      lcd.setCursor(7, 0);  lcd.write(byte(4));
      lcd.setCursor(8, 0);  lcd.write(byte(4));
      lcd.setCursor(10, 0); lcd.write(byte(3));
      lcd.setCursor(1, 1);  lcd.print("Bem-vindo(a)!");
      break;
    case 4:
    case 6:
      lcd.setCursor(7, 0); lcd.print(" ");
      lcd.setCursor(8, 0); lcd.print(" ");
      break;
    case 5:
    case 7:
      lcd.setCursor(7, 0); lcd.write(byte(4));
      lcd.setCursor(8, 0); lcd.write(byte(4));
      break;
  }
}

void atualizarAnimacao() {
  if (!animacaoAtiva) return;

  unsigned long agora = millis();

  if (tempoAnteriorFrame == 0) {
    desenharFrameAnimacao(frameAnimacao);
    tempoAnteriorFrame = agora;
    return;
  }

  if (agora - tempoAnteriorFrame >= DURACAO_FRAMES[frameAnimacao]) {
    frameAnimacao++;
    tempoAnteriorFrame = agora;

    if (frameAnimacao >= TOTAL_FRAMES) {
      animacaoAtiva = false;
      lcd.clear();
      ultimoDisplay  = millis();
      ultimaLeitura  = millis();
    } else {
      desenharFrameAnimacao(frameAnimacao);
    }
  }
}

float lerTemperatura() {
  int valor = analogRead(PINO_TEMP);
  float tensao = valor * (5.0 / 1023.0);
  return (tensao - 0.5) * 100.0;
}

float lerUmidade() {
  int valor = analogRead(PINO_UMID);
  return map(valor, 0, 1023, 0, 100);
}

int lerLuz() {
  return analogRead(PINO_LDR);
}

void atualizarAtuadores(float temp, float umid, int luz) {
  bool tempCritica = (temp < TEMP_MIN || temp > TEMP_MAX);
  bool umidCritica = (umid < UMID_MIN || umid > UMID_MAX);
  bool muitoClaro  = (luz >= LUZ_MEIA);
  bool meiaLuz     = (luz >= LUZ_ESCURO && luz < LUZ_MEIA);

  digitalWrite(LED_VERDE,   LOW);
  digitalWrite(LED_AMARELO, LOW);
  digitalWrite(LED_VERM,    LOW);

  if (umidCritica || muitoClaro) {
    digitalWrite(LED_VERM, HIGH);
  } else if (tempCritica || meiaLuz) {
    digitalWrite(LED_AMARELO, HIGH);
  } else {
    digitalWrite(LED_VERDE, HIGH);
  }

  if (tempCritica || umidCritica || muitoClaro) {
    digitalWrite(BUZZER, HIGH);
  } else {
    digitalWrite(BUZZER, LOW);
  }
}

void atualizarDisplay(float temp, float umid, int luz) {
  lcd.clear();

  if (telaAtual == 0) {
    if (luz >= LUZ_MEIA) {
      lcd.setCursor(0, 0); lcd.print("Ambiente muito");
      lcd.setCursor(0, 1); lcd.print("CLARO");
    } else if (luz >= LUZ_ESCURO) {
      lcd.setCursor(0, 0); lcd.print("Ambiente a meia");
      lcd.setCursor(0, 1); lcd.print("luz");
    } else {
      lcd.setCursor(0, 0); lcd.print("Luminosidade OK");
      lcd.setCursor(0, 1); lcd.print("Ambiente escuro");
    }
  }
  else if (telaAtual == 1) {
    lcd.setCursor(0, 0);
    if (temp > TEMP_MAX)      lcd.print("Temp. ALTA");
    else if (temp < TEMP_MIN) lcd.print("Temp. BAIXA");
    else                      lcd.print("Temperatura OK");
    lcd.setCursor(0, 1);
    lcd.print("Temp. = ");
    lcd.print(temp, 1);
    lcd.print((char)223);
    lcd.print("C");
  }
  else {
    lcd.setCursor(0, 0);
    if (umid > UMID_MAX)      lcd.print("Umidade ALTA");
    else if (umid < UMID_MIN) lcd.print("Umidade BAIXA");
    else                      lcd.print("Umidade OK");
    lcd.setCursor(0, 1);
    lcd.print("Umidade = ");
    lcd.print(umid, 0);
    lcd.print("%");
  }

  telaAtual = (telaAtual + 1) % 3;
}

void setup() {
  Serial.begin(9600);

  pinMode(LED_VERDE,   OUTPUT);
  pinMode(LED_AMARELO, OUTPUT);
  pinMode(LED_VERM,    OUTPUT);
  pinMode(BUZZER,      OUTPUT);

  digitalWrite(LED_VERDE,   LOW);
  digitalWrite(LED_AMARELO, LOW);
  digitalWrite(LED_VERM,    LOW);
  digitalWrite(BUZZER,      LOW);

  lcd.init();
  lcd.backlight();

  lcd.createChar(0, tacaEsq);
  lcd.createChar(1, tacaDir);
  lcd.createChar(2, tacaEsqBrinde);
  lcd.createChar(3, tacaDirBrinde);
  lcd.createChar(4, faiscas);

  animacaoAtiva      = true;
  frameAnimacao      = 0;
  tempoAnteriorFrame = 0;
}

void loop() {
  if (animacaoAtiva) {
    atualizarAnimacao();
    return;
  }

  unsigned long agora = millis();

  if (agora - ultimaLeitura >= INTERVALO_LEITURA) {
    ultimaLeitura = agora;

    somaTemp += lerTemperatura();
    somaUmid += lerUmidade();
    somaLuz  += lerLuz();
    contagem++;

    atualizarAtuadores(
      somaTemp / contagem,
      somaUmid / contagem,
      somaLuz  / contagem
    );

    Serial.print("Temp: "); Serial.print(lerTemperatura());
    Serial.print(" C | Umid: "); Serial.print(lerUmidade());
    Serial.print(" % | Luz: "); Serial.println(lerLuz());
  }

  if (agora - ultimoDisplay >= INTERVALO_DISPLAY) {
    ultimoDisplay = agora;

    if (contagem > 0) {
      tempMedia = somaTemp / contagem;
      umidMedia = somaUmid / contagem;
      luzMedia  = somaLuz  / contagem;
    }

    atualizarDisplay(tempMedia, umidMedia, luzMedia);

    somaTemp = 0;
    somaUmid = 0;
    somaLuz  = 0;
    contagem = 0;
  }
}
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

[Assista aqui](https://youtu.be/NXYR0pghTmU?si=jRxmmFLynwz6QA-7)

---

## Equipe do Projeto

| Nome | RM | Responsabilidade |
| :--- | :--- | :--- |
| Diego Candido Stoianof | 570748 | Desenvolvimento |
| Felipe Moreira Mendes | 570807 | Desenvolvimento |
| Lucas Zezzi Custódio | 571161 | Desenvolvimento |
| Romulo Mendes Sousa | 570620 | Desenvolvimento |
