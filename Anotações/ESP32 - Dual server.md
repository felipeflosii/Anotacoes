## ESP32 - B
Direita +
```c++
#include <WiFi.h>
#include <WebServer.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>

const char* ssid = "Fast Net milu";
const char* password = "02061972";
const char* servidorA = "http://IP_DO_A/tempo";

WebServer server(80);

const char index_html[] PROGMEM = R"rawliteral(
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Clima ESP32</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { font-family: 'Segoe UI', sans-serif; background: #0f172a; color: #e2e8f0; min-height: 100vh; display: flex; align-items: center; justify-content: center; }
    .card { background: #1e293b; border-radius: 24px; padding: 40px; width: 320px; text-align: center; box-shadow: 0 20px 60px rgba(0,0,0,0.4); }
    .title { font-size: 14px; color: #64748b; letter-spacing: 2px; text-transform: uppercase; margin-bottom: 24px; }
    .temp { font-size: 72px; font-weight: 700; color: #38bdf8; line-height: 1; }
    .temp span { font-size: 32px; color: #94a3b8; }
    .desc { font-size: 18px; color: #94a3b8; margin-top: 12px; }
    .luz { font-size: 15px; color: #94a3b8; margin-top: 8px; }
    .status { font-size: 13px; margin-top: 6px; }
    .update { font-size: 12px; color: #475569; margin-top: 24px; }
    button { margin-top: 24px; background: #38bdf8; color: #0f172a; border: none; padding: 10px 28px; border-radius: 999px; font-size: 14px; font-weight: 600; cursor: pointer; }
  </style>
</head>
<body>
  <div class="card">
    <div class="title">☁ Estação Clima</div>
    <div class="temp" id="temp">--<span>°C</span></div>
    <div class="desc" id="desc">Carregando...</div>
    <div class="luz" id="luz">Luminosidade: --</div>
    <div class="status" id="status"></div>
    <div class="update" id="update"></div>
    <button onclick="atualizar()">Atualizar</button>
  </div>
  <script>
    const wCodes = {0:"Céu limpo",1:"Principalmente limpo",2:"Parcialmente nublado",3:"Nublado",45:"Neblina",61:"Chuva leve",63:"Chuva moderada",65:"Chuva forte",80:"Pancadas leves",95:"Tempestade"};
    function atualizar() {
      document.getElementById('desc').textContent = 'Carregando...';
      fetch('/tempo').then(r=>r.json()).then(d=>{
        document.getElementById('temp').innerHTML = d.temperatura+'<span>°C</span>';
        document.getElementById('desc').textContent = wCodes[d.weathercode]||'Código '+d.weathercode;
        document.getElementById('luz').textContent = 'Luminosidade: ' + d.luminosidade + ' / 4095';
        document.getElementById('status').textContent = d.claro ? '☀️ Ambiente claro' : '🌑 Ambiente escuro';
        document.getElementById('update').textContent = 'Atualizado às '+new Date().toLocaleTimeString('pt-BR');
      }).catch(()=>{ document.getElementById('desc').textContent='Erro ao conectar'; });
    }
    atualizar();
    setInterval(atualizar,30000);
  </script>
</body>
</html>
)rawliteral";

void handleHome() {
  server.send(200, "text/html", index_html);
}

void handleTempo() {
  HTTPClient http;
  http.begin(servidorA);
  int code = http.GET();
  if (code == 200) {
    server.send(200, "application/json", http.getString());
  } else {
    server.send(500, "text/plain", "Erro ao consultar ESP32 A");
  }
  http.end();
}

void setup() {
  delay(1000);
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nIP B: " + WiFi.localIP().toString());

  server.on("/", handleHome);
  server.on("/tempo", handleTempo);
  server.begin();
  Serial.println("Servidor B rodando!");
}

void loop() {
  server.handleClient();
}
```

## ESP32 - A
Direita -
```c++
#include <WiFi.h>
#include <WebServer.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>
#include <WiFiClientSecure.h>

const char* ssid = "Fast Net milu";
const char* password = "02061972";

const char* URL = "https://api.open-meteo.com/v1/forecast"
                  "?latitude=-23.55&longitude=-46.63"
                  "&current=temperature_2m,weathercode"
                  "&timezone=America/Sao_Paulo";

#define LDR_AO 34
#define LDR_DO 35

WebServer server(80);

String buscarTempo() {
  WiFiClientSecure client;
  client.setInsecure();

  HTTPClient http;
  http.begin(client, URL);
  int code = http.GET();
  Serial.println("Code: " + String(code));

  if (code != 200) {
    http.end();
    return "{\"erro\":\"falha na requisicao\"}";
  }

  String payload = http.getString();
  http.end();

  StaticJsonDocument<1024> doc;
  deserializeJson(doc, payload);

  float temp = doc["current"]["temperature_2m"];
  int wcode  = doc["current"]["weathercode"];
  int luz    = analogRead(LDR_AO);
  bool claro = !digitalRead(LDR_DO); // LOW = claro na maioria dos módulos

  StaticJsonDocument<256> resposta;
  resposta["temperatura"]  = temp;
  resposta["weathercode"]  = wcode;
  resposta["luminosidade"] = luz;
  resposta["claro"]        = claro;

  String json;
  serializeJson(resposta, json);
  return json;
}

void setup() {
  Serial.begin(115200);
  pinMode(LDR_AO, INPUT);
  pinMode(LDR_DO, INPUT);

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nIP A: " + WiFi.localIP().toString());

  server.on("/tempo", []() {
    server.send(200, "application/json", buscarTempo());
  });

  server.begin();
  Serial.println("Servidor A rodando!");
}

void loop() {
  server.handleClient();
}
```