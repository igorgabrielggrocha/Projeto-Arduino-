# Termômetro com Termistor, Buzzer e Controle por Botões

Este projeto é um termômetro digital simples usando um **termistor NTC**, controlado por **botões** e com feedback sonoro via **buzzer**. O termômetro mede a temperatura e emite diferentes padrões sonoros com base na leitura da temperatura, simulando um termômetro convencional.

## Objetivo do Projeto
Criar um termômetro digital acionado manualmente que fornece feedback sonoro de diferentes frequências para indicar se a temperatura está em um nível normal ou febril.

## Componentes Utilizados

- **Arduino Uno (ou similar)**: Microcontrolador para processar a lógica do sistema.
- **Termistor NTC**: Sensor de temperatura que varia sua resistência conforme a temperatura.
- **Resistor de 10kΩ**: Usado como divisor de tensão no circuito com o termistor.
- **Buzzer piezoelétrico**: Para emitir sons indicando as leituras de temperatura.
- **2 Botões (push-buttons)**: Um para iniciar a medição e outro para desligar o sistema.
- **Protoboard e Jumpers**: Para montagem do circuito.
- **Fonte de Alimentação ou Cabo USB**: Para alimentar o Arduino.

## Funcionamento

1. **Botão de Início**: Quando pressionado, ativa o sistema de medição de temperatura e o aviso sonoro (buzzer).
2. **Botão de Parar**: Desativa o sistema e o aviso sonoro, desligando o buzzer.
3. **Leitura do Termistor**: O termistor mede a temperatura, e o valor é calculado utilizando a equação de Steinhart-Hart para conversão de resistência em temperatura.
4. **Aviso Sonoro**:
    - **Temperatura abaixo de 37°C**: O buzzer toca lentamente (intervalo de 500 ms).
    - **Temperatura entre 37°C e 38°C**: O buzzer também toca lentamente (intervalo de 500 ms).
    - **Temperatura acima de 38°C**: O buzzer toca rapidamente (intervalo de 100 ms), indicando febre.

## Esquema de Conexão

- O **termistor** está conectado ao pino **A0** do Arduino, com um resistor de **10kΩ** formando um divisor de tensão.
- O **buzzer** está conectado ao pino **9** do Arduino.
- O **Botão de Início** está conectado ao pino **2**, e o **Botão de Parar** está conectado ao pino **3**.
- Ambos os botões usam resistores pull-up internos (`INPUT_PULLUP`).

## Código do Projeto

Aqui está o código completo em Arduino:

```cpp
// Pinos
const int termistorPin = A0;   // Pino de entrada analógica para o termistor
const int buzzerPin = 9;       // Pino de saída para o buzzer
const int botaoStartPin = 2;   // Pino de entrada digital para o botão de iniciar
const int botaoStopPin = 3;    // Pino de entrada digital para o botão de desligar

// Variáveis de controle
bool sistemaAtivo = false;     // Estado do sistema
bool buzzerAtivo = false;      // Controle do buzzer

// Limites de temperatura
float temperaturaNormal = 37.0;  // Limite de temperatura normal
float temperaturaFebre = 38.0;   // Limite de febre

void setup() {
  pinMode(termistorPin, INPUT);
  pinMode(buzzerPin, OUTPUT);
  pinMode(botaoStartPin, INPUT_PULLUP);
  pinMode(botaoStopPin, INPUT_PULLUP);

  Serial.begin(9600); // Para monitorar a temperatura
}

void loop() {
  // Verificar se o botão de início foi pressionado
  if (digitalRead(botaoStartPin) == LOW) {
    sistemaAtivo = true;  // Ativar o sistema
    buzzerAtivo = true;   // Ativar o buzzer
    delay(200); // Debouncing simples para o botão
  }

  // Verificar se o botão de desligar foi pressionado
  if (digitalRead(botaoStopPin) == LOW) {
    sistemaAtivo = false; // Desativar o sistema
    buzzerAtivo = false;  // Desativar o buzzer
    noTone(buzzerPin);    // Parar o buzzer
    delay(200); // Debouncing simples para o botão
  }

  // Se o sistema estiver ativo, medir a temperatura
  if (sistemaAtivo) {
    float temperatura = lerTemperatura(); // Função para ler a temperatura

    Serial.print("Temperatura: ");
    Serial.println(temperatura);

    if (buzzerAtivo) {
      // Verificar a temperatura e acionar o buzzer conforme os limites
      if (temperatura >= temperaturaFebre) {
        tocarBuzzer(100); // Apitar rapidamente (100 ms) se houver febre
      } else {
        tocarBuzzer(500); // Apitar lentamente (500 ms) se estiver abaixo de 38°C
      }
    }
  }
}

// Função para ler a temperatura do termistor (simulada)
float lerTemperatura() {
  int leituraAnalogica = analogRead(termistorPin);
  // Calcular a resistência do termistor (depende do seu circuito específico)
  float resistencia = (1023.0 / leituraAnalogica) - 1.0;
  resistencia = 10000.0 / resistencia; // Assumindo um resistor de 10kΩ

  // Fórmula para conversão de resistência para temperatura (Steinhart-Hart)
  float temperatura = resistencia / 10000.0; // Dividir por 10kΩ
  temperatura = log(temperatura);            // Logaritmo natural da resistência
  temperatura = temperatura * 3950.0;        // Constante beta
  temperatura = temperatura / (temperatura + 273.15); // Converter para Kelvin
  temperatura = (1 / temperatura) - 273.15;  // Converter para Celsius

  return temperatura;
}

// Função para controlar o buzzer
void tocarBuzzer(int intervalo) {
  tone(buzzerPin, 1000);   // Emitir som a 1kHz
  delay(intervalo);        // Aguardar por intervalo definido
  noTone(buzzerPin);       // Parar o som
  delay(intervalo);        // Aguardar entre toques
}
