# Sistema de Monitoramento de Luminosidade com Arduino

## 📌 Descrição

Este projeto utiliza um fotoresistor (LDR) para medir a luminosidade do
ambiente e classificar em três estados:

-   🟢 **OK (escuro)** → LED verde
-   🟡 **Alerta (luz média)** → LED amarelo
-   🔴 **Perigo (muita luz)** → LED vermelho + buzzer

O sistema também utiliza controle de tempo com `millis()` para evitar
travamentos.

------------------------------------------------------------------------

## ⚙️ Componentes Utilizados

-   Arduino (Uno)
-   1x LDR (fotoresistor)
-   4x resistor (1kΩ)
-   3x LEDs (verde, amarelo, vermelho)
-   1x buzzer
-   Protoboard e jumpers

------------------------------------------------------------------------

## 🔌 Funcionamento

O LDR mede a luz ambiente e envia um valor analógico (0--1023) para o
Arduino.

Com base nesse valor:

  Valor   Estado   Ação
  ------- -------- -----------------------
  Baixo   OK       LED verde
  Médio   Alerta   LED amarelo
  Alto    Perigo   LED vermelho + buzzer

------------------------------------------------------------------------

## 🧠 Lógica do Sistema

-   Leitura contínua do sensor com `analogRead()`
-   Comparação com limites (`min` e `max`)
-   Controle do buzzer com `millis()` (sem `delay()`)

------------------------------------------------------------------------

## ⏱ Controle de Tempo

O buzzer utiliza `millis()` para alternar entre ligado e desligado sem
travar o sistema:

-   3 segundos ligado
-   1 segundo desligado

------------------------------------------------------------------------

## 💻 Código Principal

``` cpp
const int ledGreen = 12;
const int ledYellow = 11;
const int ledRed = 10;
const int fotoResistor = A0;
const int buzzer = 8;

int sensor;

const int max = 500;  
const int min = 200; 

bool buzzerEstado = false;

unsigned long tempoAnterior = 0;

const unsigned long tempoLigado = 3000;  
const unsigned long tempoDesligado = 1000;

void setup()
{
  pinMode(ledGreen, OUTPUT);
  pinMode(ledYellow, OUTPUT);
  pinMode(ledRed, OUTPUT);
  pinMode(buzzer, OUTPUT);
  pinMode(fotoResistor, INPUT);

  Serial.begin(9600);
}

void loop()
{
  sensor = analogRead(fotoResistor);
  Serial.println(sensor);

  //PERIGO
  if(sensor > max)
  {
    digitalWrite(ledGreen, LOW);
    digitalWrite(ledYellow, LOW);
    digitalWrite(ledRed, HIGH);
	
    if(buzzerEstado == false && (millis() - tempoAnterior >= tempoDesligado))
    {
      digitalWrite(buzzer, HIGH);
      buzzerEstado = !buzzerEstado;
      tempoAnterior = millis();
    }

    if(buzzerEstado == true && (millis() - tempoAnterior >= tempoLigado))
    {
      digitalWrite(buzzer, LOW);
      buzzerEstado = !buzzerEstado;
      tempoAnterior = millis();
    }
  }
  
  //ALERTA
  else if(sensor > min)
  {
    digitalWrite(ledGreen, LOW);
    digitalWrite(ledYellow, HIGH);
    digitalWrite(ledRed, LOW);

    digitalWrite(buzzer, LOW);
  }

  //OK
  else
  {
    digitalWrite(ledGreen, HIGH);
    digitalWrite(ledYellow, LOW);
    digitalWrite(ledRed, LOW);

    digitalWrite(buzzer, LOW);
  }
}
```

------------------------------------------------------------------------

## ⚠️ Observação

-   Os valores de `min` e `max` podem variar conforme o ambiente

------------------------------------------------------------------------

## 🎥 Vídeo explicativo
[Assista aqui](https://youtu.be/W0Ew7Kdyyu4)

## 🔧 Simulação no Tinkercad
[Acessar projeto](https://www.tinkercad.com/things/eTcBVCVjSaL-sensor-de-luz-vinho?sharecode=P5YtqkrpzU0nAVl10NznkygJ9-fpc6pXPqkxsycQ5mE)


------------------------------------------------------------------------

## 👨‍💻 Autor

Diego Candido Stoianof - RM: 570748
Felipe Moreira Mendes - RM 570807
Lucas Zezzi Custódio - RM: 571161
Romulo Mendes Sousa -  RM: 570620
