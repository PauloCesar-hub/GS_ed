🌡️📡 Monitoramento Ambiental com ESP32, DHT22, MQTT e Node-RED Dashboard

Este projeto implementa um sistema IoT completo para monitoramento em tempo real de temperatura e umidade utilizando ESP32, sensor DHT22, MQTT (HiveMQ) e Node-RED Dashboard.

O sistema coleta dados do ambiente, envia para a nuvem via MQTT e exibe gráficos e indicadores no painel web do Node-RED.

📌 Objetivos do Projeto

Medir temperatura e umidade com precisão usando DHT22.

Enviar dados ao broker MQTT HiveMQ a cada 2 segundos.

Processar e exibir informações em tempo real no Node-RED Dashboard.

Demonstrar conceitos de IoT, Edge Computing e telemetria remota.

🛠️ Tecnologias Utilizadas
Camada	Ferramenta
Microcontrolador	ESP32
Sensor	DHT22
Protocolo	MQTT
Broker	HiveMQ Public Broker
Backend/Lógica	Node-RED
Frontend	Node-RED Dashboard
Simulação (opcional)	Wokwi
🚀 Funcionamento Geral do Sistema
flowchart LR
    A[DHT22] --> B[ESP32]
    B -->|MQTT Publish| C[HiveMQ Broker]
    C -->|MQTT Subscribe| D[Node-RED]
    D --> E[Dashboard Web UI]

🔧 Hardware Necessário

ESP32 DevKit V1

Sensor DHT22

Cabos jumper

(Opcional) Simulação no Wokwi

📡 Configurações do MQTT

Broker: broker.hivemq.com

Porta: 1883

Tópico de publicação:

esp32/monitoramento/ambiente

📜 Código Completo do ESP32
  #include <WiFi.h>
#include <PubSubClient.h>
#include "DHT.h"

#define DHTPIN 15
#define DHTTYPE DHT22
DHT dht(DHTPIN, DHTTYPE);

// ======== CONFIG WIFI ========
const char* ssid = "Wokwi-GUEST";
const char* password = "";

// ======== CONFIG MQTT ========
const char* mqtt_server = "broker.hivemq.com";
const int   mqtt_port = 1883;
const char* mqtt_topic = "esp32/monitoramento/ambiente";

WiFiClient espClient;
PubSubClient client(espClient);

unsigned long lastMsg = 0;
const long interval = 2000; // 2 segundos

void reconnect() {
  // Reconeção automática
  while (!client.connected()) {
    Serial.print("Tentando conectar ao MQTT...");
    if (client.connect("ESP32-Monitor-Node")) {
      Serial.println("Conectado!");
    } else {
      Serial.print("Falhou, rc=");
      Serial.print(client.state());
      Serial.println(" tentando novamente em 2s...");
      delay(2000);
    }
  }
}

void setup() {
  Serial.begin(115200);
  dht.begin();

  WiFi.begin(ssid, password);
  Serial.print("Conectando ao WiFi...");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi conectado!");
  Serial.print("IP: ");
  Serial.println(WiFi.localIP());

  client.setServer(mqtt_server, mqtt_port);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > interval) {
    lastMsg = now;

    float temp = dht.readTemperature();
    float umid = dht.readHumidity();

    if (isnan(temp) || isnan(umid)) {
      Serial.println("Falha ao ler do sensor DHT!");
      return;
    }

    String payload = "{\"temperatura\": " + String(temp, 2) + ", \"umidade\": " + String(umid, 2) + "}";
    client.publish(mqtt_topic, payload.c_str());
    Serial.println("Publicado: " + payload);
  }
}


🧩 Fluxo do Node-RED
✔️ Nó MQTT IN

Server: broker.hivemq.com

Topic: esp32/monitoramento/ambiente

✔️ Função para Extrair Temperatura
let data = JSON.parse(msg.payload);
msg.payload = data.temperatura;
return msg;

✔️ Função para Extrair Umidade
let data = JSON.parse(msg.payload);
msg.payload = data.umidade;
return msg;

✔️ Elementos do Dashboard

Gauge: Temperatura

Gauge: Umidade

Chart: Histórico das duas variáveis

🖥️ Interface do Dashboard

O painel apresenta:

🌡️ Temperatura (°C) em tempo real

💧 Umidade (%) em tempo real

📈 Gráficos históricos

📊 Atualização automática a cada 2 segundos

Exemplo:

---------------------------------------
|   🌡️ Temperatura   |   💧 Umidade   |
|      25.3°C         |      40%       |
---------------------------------------
|    📈 Histórico de valores          |
---------------------------------------

🧪 Como Executar o Projeto
1️⃣ ESP32

Colar o código no Arduino IDE

Instalar bibliotecas:

DHT sensor library

PubSubClient

Selecionar placa ESP32 Dev Module

Upload para o ESP32

2️⃣ Node-RED

Instalar Node-RED:

npm install -g node-red


Instalar Dashboard:

npm install node-red-dashboard


Importar o fluxo JSON

Iniciar:

node-red


Acessar:

http://localhost:1880/ui
link do projeto https://wokwi.com/projects/447354816042772481
📽️ Vídeo Demonstrativo (https://youtube.com/shorts/DmSgEp4C5Vo)

O vídeo apresenta:

O problema abordado

A solução IoT criada

Funcionamento ESP32 + Node-RED


📍 Resultados

Monitoramento remoto confiável

Atualização contínua via MQTT

Dashboard claro e responsivo

Base ideal para automação residencial/industrial

🧑‍💻 Autores

Paulo Cesar de Govea Junior — RM: 566034

Guilherme Vilela Perez — RM: 564422

Gustavo Panham Dourado — RM: 563904
