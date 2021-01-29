#include <Arduino.h>
#include <esp_now.h>
#include <WiFi.h>
#include "ESPAsyncWebServer.h"
#include <Arduino_JSON.h>

//Bağlanılacak WiFi adresi bilgileri
const char* ssid = "Philippe";
const char* password = "11235813";

const char* http_username = "admin";
const char* http_password = "admin";

/* NTP Server Details
const char* ntpServer = "pool.ntp.org";
const long  gmtOffset_sec = 0;
const int   daylightOffset_sec = 3600;  */

//Butonların ayrı ayrı durumları
const char* PARAM_INPUT_1 = "state";
const char* PARAM_INPUT_2 = "state2";
const char* PARAM_INPUT_3 = "state3";
const char* PARAM_INPUT_4 = "state4";

//Değişken atamalar
const int output = 2;
const int output2 = 4;
const int output3 = 12;
const int output4 = 14;

//Anlık button durumu
int buttonState = LOW;    

// Sender modülü tarafından gönderilen bilgierin eşleşip eşleimediği kontrol edilir
typedef struct struct_message {
  int id;
  float temp;
  float hum;
  unsigned int readingId;
} struct_message;

struct_message incomingReadings;

JSONVar board;

AsyncWebServer server(80);
AsyncEventSource events("/events");

void OnDataRecv(const uint8_t * mac_addr, const uint8_t *incomingData, int len) { 
  
  char macStr[18];
  Serial.print("Packet received from: ");
  snprintf(macStr, sizeof(macStr), "%02x:%02x:%02x:%02x:%02x:%02x",
  mac_addr[0], mac_addr[1], mac_addr[2], mac_addr[3], mac_addr[4], mac_addr[5]);
  Serial.println(macStr);
  memcpy(&incomingReadings, incomingData, sizeof(incomingReadings));
  board["id"] = incomingReadings.id;
  board["temperature"] = incomingReadings.temp;
  board["humidity"] = incomingReadings.hum;
  board["readingId"] = String(incomingReadings.readingId);
  String jsonString = JSON.stringify(board);
   events.send(jsonString.c_str(), "new_readings", millis());

  Serial.printf("Board ID %u: %u bytes\n", incomingReadings.id, len);
  Serial.printf("t value: %4.2f \n", incomingReadings.temp);
  Serial.printf("h value: %4.2f \n", incomingReadings.hum);
  Serial.printf("readingID value: %d \n", incomingReadings.readingId);
  Serial.println();
}



const char index_html[] PROGMEM = R"rawliteral(
<!DOCTYPE HTML>
<html>
<head>
  <title>PAMUKKALE UNIVERSITESI</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta charset="utf-8">
  <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.7.2/css/all.css" integrity="sha384-fnmOCqbTlWIlj8LyTjo7mOUStjsKC4pOpQbqyi7RrhN7udi9RwhKkMHpvLbHG9Sr" crossorigin="anonymous">
  <link rel="icon" href="data:,">
  <script src="https://code.highcharts.com/highcharts.js"></script>
  <style>
    html {font-family: Arial; display: inline-block; text-align: center;}
    p {  font-size: 1.2rem;}
    body {  margin: 0;}
    
    .topnav { overflow: hidden; background-color: #2f4468; color: white; font-size: 1.7rem; }
    .content { padding: 20px; }
    .contentbutton { padding: 20px; }
    .card { background-color: white; box-shadow: 2px 2px 12px 1px rgba(140,140,140,.5); }
    .cards { max-width: 700px; margin: 0 auto; display: grid; grid-gap: 2rem; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); }
    .reading { font-size: 2.8rem; }

    .datetime {color: #fff; background: #2f4468; font-family: "Segoe UI", sans-serif; width: 340; padding: 15px 10px;}
    //.datetime:hover{bachround: #2E94E3; box-shadow: 0 0 30px #2E94E3;}
    .date {font-size: 15px; font-weight: 600; text-align: center; letter-spacing: 3px;}
    .time {font-size: 45px; display: flex; justify-content: center; align-items: center;} 
    .time span:not(:last-child){position: relative; margin: 0 6px; font-weight: 600; text-align: center; letter-spacing: 3px;}
    .time span:last-child{background: #2E94E3; font-size: 15px; font-weight: 600; text-transform: uppercase; margin-top: 10px; padding: 0 5px; border-radius: 3px;}

    .packet { color: #bebebe; }
    .card.temperature { color: #fd7e14; }
    .card.humidity { color: #1b78e2; }
    .switch {position: relative; display: inline-block; width: 120px; height: 68px} 
    .switch input {display: none}
    .slider {position: absolute; top: 0; left: 0; right: 0; bottom: 0; background-color: #ccc; border-radius: 34px}
    .slider:before {position: absolute; content: ""; height: 52px; width: 52px; left: 8px; bottom: 8px; background-color: #fff; -webkit-transition: .4s; transition: .4s; border-radius: 68px}
    input:checked+.slider {background-color: #2196F3}
    input:checked+.slider:before {-webkit-transform: translateX(52px); -ms-transform: translateX(52px); transform: translateX(52px)}
  </style>
</head>
<body>
  <div class="topnav">
    <h2>KONTROL PANELI</h2>
  </div>

<body onload="initClock()">

  <div class="datetime">
    <div class="date">
      <span id="dayname">Day</span>,
      <span id="month">Month</span>,
      <span id="daynum">00</span>,
      <span id="year">Year</span>
    </div>
    <div class="time">
      <span id="hour">00</span>:
      <span id="minutes">00</span>:
      <span id="seconds">00</span>
      <span id="period">AM</span>
    </div>
  </div>

  <div class="content">
    <div class="cards">
      <div class="card temperature">
        <h4><i class="fas fa-thermometer-half"></i> BOARD #1 - TEMPERATURE</h4><p><span class="reading"><span id="t1"></span> &deg;C</span></p><p class="packet">Reading ID: <span id="rt1"></span></p>
      </div>
      <div class="card humidity">
        <h4><i class="fas fa-tint"></i> BOARD #1 - HUMIDITY</h4><p><span class="reading"><span id="h1"></span> &percnt;</span></p><p class="packet">Reading ID: <span id="rh1"></span></p>
      </div>
      <div class="card temperature">
        <h4><i class="fas fa-thermometer-half"></i> BOARD #2 - TEMPERATURE</h4><p><span class="reading"><span id="t2"></span> &deg;C</span></p><p class="packet">Reading ID: <span id="rt2"></span></p>
      </div>
      <div class="card humidity">
        <h4><i class="fas fa-tint"></i> BOARD #2 - HUMIDITY</h4><p><span class="reading"><span id="h2"></span> &percnt;</span></p><p class="packet">Reading ID: <span id="rh2"></span></p>
      </div>
    </div>
  </div>

  <div class="content">
    <div class="cards">
      <div class="card">
        <div class="contentbutton">
          <h3>Buton 1 - GPIO 2 - Durum</h3>
          <label class="switch"><input type="checkbox" onchange="toggleCheckbox(this)" id="output"><span class="slider"></span></label>
        </div>

        <div class="contentbutton">
          <h3>Buton 2 - GPIO 4 - Durum</h3>
          <label class="switch"><input type="checkbox" onchange="toggleCheckbox2(this)" id="output2"><span class="slider"></span></label>
        </div>

        <div class="contentbutton">
          <h3>Buton 3 - GPIO 12 - Durum</h3>
          <label class="switch"><input type="checkbox" onchange="toggleCheckbox3(this)" id="output3"><span class="slider"></span></label>
        </div>

        <div class="contentbutton">
          <h3>Buton 4 - GPIO 14 - Durum</h3>
          <label class="switch"><input type="checkbox" onchange="toggleCheckbox4(this)" id="output4"><span class="slider"></span></label>
        </div>
      </div>
    </div>
  </div>

  <div class= "content">
    <div class="cards">
      <div class="card">
        <div class='contentbutton'>
          <div id="chart-temperature" class="container"></div>
        </div>
      </div>
    </div>
  </div>

  <div class= "content">
      <div class="cards">
        <div class="card">
          <div class='contentbutton'>
            <button onclick="logoutButton()">Logout</button>
          </div>
        </div>
      </div>
  </div>

<script>function toggleCheckbox(element) {
  var xhr = new XMLHttpRequest();
  if(element.checked){
    xhr.open("GET", "/update?state=0", true);
    }
  else { 
    xhr.open("GET", "/update?state=1", true); 
    }
    xhr.send();
}

function toggleCheckbox2(element) {
  var xhr = new XMLHttpRequest();
  if(element.checked){
    xhr.open("GET", "/update?state2=0", true);
    }
  else { 
    xhr.open("GET", "/update?state2=2", true); 
    }
    xhr.send();
}

function toggleCheckbox3(element) {
  var xhr = new XMLHttpRequest();
  if(element.checked){
    xhr.open("GET", "/update?state3=0", true);
    }
  else { 
    xhr.open("GET", "/update?state3=3", true); 
    }
    xhr.send();
}

function toggleCheckbox4(element) {
  var xhr = new XMLHttpRequest();
  if(element.checked){
    xhr.open("GET", "/update?state4=0", true);
    }
  else { 
    xhr.open("GET", "/update?state4=4", true); 
    }
    xhr.send();
}

var chartT = new Highcharts.Chart({
  chart:{ renderTo : 'chart-temperature' },
  title: { text:'BME280 Temperature' },
  series: [{
    showInLegend: false,
    data: []
  }],
  plotOptions: {
    line: { animation: false,
      dataLabels: { enabled: true }
    },
    series: { color: '#059e8a' }
  },
  xAxis: { type: 'datetime',
    dateTimeLabelFormats: { second: '%H:%M:%S' }
  },
  yAxis: {
    title: { text: 'Temperature (Celsius)' }
    //title: { text: 'Temperature (Fahrenheit)' }
  },
  credits: { enabled: false }
  
});

function logoutButton() {
  var xhr = new XMLHttpRequest();
  xhr.open("GET", "/logout", true);
  xhr.send();
  setTimeout(function(){ window.open("/logged-out","_self"); }, 1000);
}

function updateClock(){
  var now = new Date();
  var dname = now.getDay(),
      mo = now.getMonth(),
      dnum = now.getDate(),
      yr = now.getFullYear(),
      hou = now.getHours(),
      min = now.getMinutes(),
      sec = now.getSeconds(),
      pe = "AM";
      
      /*if(hou == 0){
        hou = 24;
      }
      if(hou > 12){
        hou = hou - 12;
        pe = "PM";
      }*/

      Number.prototype.pad = function(digits){
        for(var n = this.toString(); n.lenght < digits; n = 0 + n);
        return n;
      }

      var months = ["January", "February", "March", "April", "May", "June", "Jule", "Augest", "September", "October", "November", "December"];
      var week = ["Sunday", "Monday", "Tuesday", "Wednsday", "Thursday", "Friday", "Saturday"];
      var ids = ["dayname", "month", "daynum", "year", "hour", "minutes", "seconds", "period"];
      var values = [week[dname], months[mo], dnum.pad(2), yr, hou.pad(2), min.pad(2), sec.pad(2), pe];
      for(var i = 0; i < ids.length; i++)
      document.getElementById(ids[i]).firstChild.nodeValue = values[i];

}

function initClock(){
  updateClock();
  window.setInterval("updateClock()", 1);
}


if (!!window.EventSource) {
 var source = new EventSource('/events');
 
 source.addEventListener('open', function(e) {
  console.log("Events Connected");
 }, false);
 
 source.addEventListener('error', function(e) {
  if (e.target.readyState != EventSource.OPEN) {
    console.log("Events Disconnected");
  }
 }, false);
 
 source.addEventListener('message', function(e) {
  console.log("message", e.data);
 }, false);
 
 source.addEventListener('new_readings', function(e) {
  console.log("new_readings", e.data);
  var obj = JSON.parse(e.data);
  document.getElementById("t"+obj.id).innerHTML = obj.temperature.toFixed(2);
  document.getElementById("h"+obj.id).innerHTML = obj.humidity.toFixed(2);
  document.getElementById("rt"+obj.id).innerHTML = obj.readingId;
  document.getElementById("rh"+obj.id).innerHTML = obj.readingId;
  var x = (new Date()).getTime(),
  y = Number(obj.temperature.toFixed(2));
  console.log(y);
  if(chartT.series[0].data.length > 40) {
    chartT.series[0].addPoint([x, y], true, true, true);
  } 
  else {
    chartT.series[0].addPoint([x, y], true, false, true);
  }
 }, false);
}
</script>
</body>
</html>)rawliteral";

const char logout_html[] PROGMEM = R"rawliteral(
<!DOCTYPE HTML><html>
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
  <p>Logged out or <a href="/">return to homepage</a>.</p>
  <p><strong>Note:</strong> close all web browser tabs to complete the logout process.</p>
</body>
</html>
)rawliteral";



void setup() {
  // Serial Monitör yüklenir
  Serial.begin(115200);

  //Esp32 çıkış pinlerinin ayarlanması
  pinMode(output, OUTPUT);
  digitalWrite(output, HIGH);

  pinMode(output2, OUTPUT);
  digitalWrite(output2, HIGH);

  pinMode(output3, OUTPUT);
  digitalWrite(output3, HIGH);

  pinMode(output4, OUTPUT);
  digitalWrite(output4, HIGH);

  // Wifi modülünün modu ayarlanır 
  WiFi.mode(WIFI_AP_STA);
  
  // Modül Wifi istasyonu olarak ayarlanır
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Setting as a Wi-Fi Station..");
  }
  Serial.print("Station IP Address: ");
  Serial.println(WiFi.localIP());
  Serial.print("Wi-Fi Channel: ");
  Serial.println(WiFi.channel());

  // ESP-NOW protokolü yüklenir
  if (esp_now_init() != ESP_OK) {
    Serial.println("Error initializing ESP-NOW");
    return;
  }
  
  esp_now_register_recv_cb(OnDataRecv);

  // Web sayfasına yönlendirilir  
  server.on("/", HTTP_GET, [](AsyncWebServerRequest *request){
    if(!request->authenticate(http_username, http_password))
      return request->requestAuthentication();
    request->send_P(200, "text/html", index_html);
  });

  server.on("/logout", HTTP_GET, [](AsyncWebServerRequest *request){
    request->send(401);
  });

  server.on("/logged-out", HTTP_GET, [](AsyncWebServerRequest *request){
    request->send_P(200, "text/html", logout_html);
  });

  // Butonların çalıştırılması durumunda SerialMonitörde göösterilecek mesaj
  server.on("/update", HTTP_GET, [] (AsyncWebServerRequest *request) {
    String inputMessage;
    String inputParam;
    String inputMessage2;
    String inputParam2;
    String inputMessage3;
    String inputParam3;
    String inputMessage4;
    String inputParam4;
  
    if (request->hasParam(PARAM_INPUT_1)) {
      inputMessage = request->getParam(PARAM_INPUT_1)->value();
      inputParam = PARAM_INPUT_1;
      digitalWrite(output, inputMessage.toInt());
      buttonState = !buttonState;
      Serial.println(inputMessage);
      request->send(200, "text/plain", "OK");
    }
    if (request->hasParam(PARAM_INPUT_2)) {
      inputMessage2 = request->getParam(PARAM_INPUT_2)->value();
      inputParam2 = PARAM_INPUT_2;
      digitalWrite(output2, inputMessage2.toInt());
      buttonState = !buttonState;
      Serial.println(inputMessage2);
      request->send(200, "text/plain", "OK");
    }
    if (request->hasParam(PARAM_INPUT_3)) {
      inputMessage3 = request->getParam(PARAM_INPUT_3)->value();
      inputParam3 = PARAM_INPUT_3;
      digitalWrite(output3, inputMessage3.toInt());
      buttonState = !buttonState;
      Serial.println(inputMessage3);
      request->send(200, "text/plain", "OK");
    }
    if (request->hasParam(PARAM_INPUT_4)) {
      inputMessage4 = request->getParam(PARAM_INPUT_4)->value();
      inputParam4 = PARAM_INPUT_4;
      digitalWrite(output4, inputMessage4.toInt());
      buttonState = !buttonState;
      Serial.println(inputMessage4);
      request->send(200, "text/plain", "OK");
    }
  });

  events.onConnect([](AsyncEventSourceClient *client){
    if(client->lastId()){
      Serial.printf("Client reconnected! Last message ID that it got is: %u\n", client->lastId());
    }
    client->send("hello!", NULL, millis(), 10000);
  });
  server.addHandler(&events);
  server.begin();
}
 
void loop() {
  static unsigned long lastEventTime = millis();
  static const unsigned long EVENT_INTERVAL_MS = 5000;
  if ((millis() - lastEventTime) > EVENT_INTERVAL_MS) {
    events.send("ping",NULL,millis());
    lastEventTime = millis();
  }
}
