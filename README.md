# Sistema de Monitoramento e Controle — Horta Urbana

Projeto desenvolvido para a disciplina **Introducao a Computacao** — UniCEUB  
**Integrantes:** Rhyan Portilho, Pedro Shiokawa

---

## Sobre o projeto

Sistema IoT de baixo custo para monitorar e automatizar a irrigacao de hortas em apartamentos.  
O ESP32 le sensores locais, consulta a previsao do tempo e decide se deve acionar a bomba d'agua, publicando os dados em um dashboard na nuvem via MQTT.

---

## Hardware necessario

| Componente | Modelo |
|---|---|
| Microcontrolador | ESP32 DevKit V1 |
| Sensor de solo | Capacitivo v1.2 |
| Sensor de ar | DHT11 |
| Atuador | Mini Bomba Submersivel 3V-6V |
| Interface de potencia | Modulo Rele 5V 1 Canal |
| Alimentacao | Fonte 5V/2A Micro-USB |

---

## Como configurar

1. Clone o repositorio
2. Copie `credentials.example.h` para `credentials.h`
3. Preencha suas credenciais de Wi-Fi, MQTT e OpenWeatherMap em `credentials.h`
4. Abra `horta_urbana.ino` no Arduino IDE ou VS Code com PlatformIO
5. Instale as bibliotecas abaixo e faca o upload para o ESP32

---

## Bibliotecas necessarias (Arduino IDE)

- `PubSubClient` — MQTT
- `ArduinoJson` — parse do JSON da API
- `DHT sensor library` — sensor DHT11
- `WiFiClientSecure` — ja inclusa no pacote ESP32

---

## Logica de funcionamento

```
A cada 15 min (ou 5 min se temp > 32 C):
  1. Le umidade do solo, temperatura e umidade do ar
  2. Publica os dados no broker MQTT
  3. Se solo seco (< 30%):
       Consulta API OpenWeatherMap
       Se API OK e chuva < 20%  -> Aciona bomba (max 30s)
       Se API OK e chuva >= 20% -> Adia rega
       Se API falhar            -> Fallback: aciona bomba pelo sensor local
  4. Se umidade do ar < 20%: envia alerta de estresse hidrico
```

---

## Seguranca implementada

- Comunicacao MQTT com **TLS 1.2** (porta 8883)
- Credenciais em arquivo separado fora do versionamento (`.gitignore`)
- **Watchdog timer** de 30 segundos para reinicializacao automatica em caso de travamento
- **Fallback local**: opera sem internet usando apenas os sensores
- **Validacao de dados**: todos os retornos da API sao verificados antes de qualquer atuacao fisica

---

## Topicos MQTT

| Topico | Direcao | Descricao |
|---|---|---|
| `horta/solo/umidade` | publish | Umidade do solo (%) |
| `horta/ar/temperatura` | publish | Temperatura (C) |
| `horta/ar/umidade` | publish | Umidade relativa do ar (%) |
| `horta/bomba/status` | publish | Status da bomba (ON/OFF) |
| `horta/alerta` | publish | Alertas do sistema |
| `horta/bomba/cmd` | subscribe | Comando manual (ON/OFF) |
