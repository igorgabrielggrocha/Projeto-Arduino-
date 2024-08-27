# Despertador com Display de 7 Segmentos e Buzzer

## Descrição

Este projeto é um despertador básico usando um microcontrolador Arduino. O sistema conta com um display de 7 segmentos que mostra o tempo decorrido desde que o temporizador foi iniciado. O tempo pode ser definido por meio de três botões, que configuram o temporizador para 1 minuto, 2 minutos ou 3 minutos. Quando o tempo se esgota, um buzzer é ativado para alertar o usuário.

## Componentes Necessários

- **Arduino** (Uno, Nano, etc.)
- **Display de 7 segmentos** (4 dígitos)
- **Resistores** (para LEDs do display)
- **Buzzer**
- **3 Botões**
- **Protoboard e fios de conexão**

## Esquema de Ligação

### Display de 7 Segmentos
- **Pinos Digitais do Arduino**: Conecte os pinos do display aos pinos digitais do Arduino através de resistores.
- **Comum**: Se for um display de cátodo comum, conecte o pino comum ao GND do Arduino. Se for um display de ânodo comum, conecte ao VCC.

### Botões
- **Botão 1**: Conecte a um pino digital (pino 2).
- **Botão 2**: Conecte a um pino digital (pino 3).
- **Botão 3**: Conecte a um pino digital (pino 4).
- **GND**: Conecte o outro terminal de cada botão ao GND do Arduino.

### Buzzer
- **Pino Digital do Arduino**: Conecte um pino do buzzer ao pino digital 9.
- **GND**: Conecte o outro pino do buzzer ao GND do Arduino.

## Código Arduino

O código Arduino para este projeto configura o display de 7 segmentos para mostrar minutos e segundos passados desde o início do temporizador e aciona um buzzer quando o tempo se esgota.

```cpp
#include <SevSeg.h>

SevSeg sevseg; // Cria um objeto SevSeg

const int buttonPin1 = 2; // Pino do botão para 1 minuto
const int buttonPin2 = 3; // Pino do botão para 2 minutos
const int buttonPin3 = 4; // Pino do botão para 3 minutos
const int buzzerPin = 9; // Pino do buzzer

int buttonState1 = 0;
int buttonState2 = 0;
int buttonState3 = 0;

unsigned long startTime;
unsigned long timerDuration; // Duração do timer em milissegundos
bool timerRunning = false;

void setup() {
  // Configura o display de 7 segmentos
  byte numDigits = 4; // 4 dígitos para mostrar minutos e segundos
  byte digitPins[] = {5, 6, 7, 8}; // Pinos dos dígitos
  byte segmentPins[] = {9, 10, 11, 12, 13, A0, A1}; // Pinos dos segmentos
  bool resistorsOnSegments = false; // Se os resistores estão nos pinos dos segmentos
  bool updateWithDelays = false; // Atualizar o display com delays
  bool leadingZeros = false; // Mostrar zeros à esquerda
  bool disableDecPoint = true; // Desativar o ponto decimal
  
  sevseg.begin(COMMON_CATHODE, numDigits, digitPins, segmentPins, resistorsOnSegments);
  sevseg.setBrightness(90); // Configura o brilho do display
  
  pinMode(buttonPin1, INPUT_PULLUP); // Configura o botão 1 como entrada com pull-up
  pinMode(buttonPin2, INPUT_PULLUP); // Configura o botão 2 como entrada com pull-up
  pinMode(buttonPin3, INPUT_PULLUP); // Configura o botão 3 como entrada com pull-up
  pinMode(buzzerPin, OUTPUT); // Configura o buzzer como saída
}

void loop() {
  buttonState1 = digitalRead(buttonPin1);
  buttonState2 = digitalRead(buttonPin2);
  buttonState3 = digitalRead(buttonPin3);

  if (buttonState1 == LOW) { // Se o botão 1 é pressionado
    timerDuration = 1 * 60 * 1000; // 1 minuto em milissegundos
    startTimer();
  } else if (buttonState2 == LOW) { // Se o botão 2 é pressionado
    timerDuration = 2 * 60 * 1000; // 2 minutos em milissegundos
    startTimer();
  } else if (buttonState3 == LOW) { // Se o botão 3 é pressionado
    timerDuration = 3 * 60 * 1000; // 3 minutos em milissegundos
    startTimer();
  }

  if (timerRunning) {
    unsigned long currentTime = millis();
    unsigned long elapsedTime = currentTime - startTime;
    unsigned long remainingTime = timerDuration - elapsedTime;

    if (remainingTime <= 0) {
      remainingTime = 0;
      timerRunning = false;
      digitalWrite(buzzerPin, HIGH); // Ativa o buzzer
    }

    // Calcula minutos e segundos passados
    unsigned long minutesPassed = (elapsedTime / 1000) / 60;
    unsigned long secondsPassed = (elapsedTime / 1000) % 60;

    // Configura o display para mostrar minutos e segundos passados
    sevseg.setNumber(minutesPassed * 100 + secondsPassed); // Formato MMSS
    sevseg.refreshDisplay(); // Atualiza o display

    if (remainingTime <= 0 && millis() - startTime >= 10000) { // Buzzer ligado por 10 segundos
      digitalWrite(buzzerPin, LOW);
    }
  } else {
    sevseg.setNumber(0); // Mostra 0 quando o temporizador não está em execução
    sevseg.refreshDisplay();
  }
}

void startTimer() {
  if (!timerRunning) {
    startTime = millis();
    timerRunning = true;
    digitalWrite(buzzerPin, LOW); // Desativa o buzzer ao iniciar o timer
  }
}
