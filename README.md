# Middleware MODBUS e NMEA - IoTLog

For this instrucions in english, please [click here](https://github.com/bykonz/middleware-dotnet-release/blob/main/README.en.MD)

Middleware de integração de NMEA e MODBUS para IoTLog via protocolo padrão MQTT

### Instalação execução

Necessário instalar [.NET 6](https://dotnet.microsoft.com/download/dotnet/6.0)

Realizar o download da última versão do [Middleware.WorkerService.App](https://github.com/bykonz/middleware-dotnet-release/releases)

Extrair os arquivos compactados em um diretório.

Configurar as váriaveis do ambiente no `appsettings.json`

```
{
  "NMEA": { // configurações de conexão com o NMEA
    "Mode": "SERIAL", // modo de conexão (utilizar SERIAL ou TCP)
    "Serial": { // configurações para o modo SERIAL, informar apenas se o `Mode: SERIAL`
      "Port": "COM2", // Serial Port
      "Baudrate": 9600, // Baudrate
      "Limiter": "\n" // Caracter limitador
    },
    "TCP": { // configurações para o modo TCP, informar apenas se o `Mode: TCP`
      "Endpoint": "localhost", // Endpoint TCP
      "Port": 3000 // Porta TCP
    },
    "IntervalSave": 1000 // intervalo para salvar no banco de dados
  },
  "MODBUS": { // configurações de conexão com o MODBUS
    "Mode": "TCP", // modo de comunicação disponível (TCP ou SERIAL)
    "TCP": { // configurações para o modo TCP, informar apenas se o `Mode: TCP`
      "Endpoint": "127.0.0.1", // Endpoint TCP
      "Port": 502 // Porta TCP
    },
    "Serial": {  //configurações para o modo SERIAL, informar apenas se o `Mode: SERIAL`
      "Port": "COM2", // porta serial
      "Baudrate": 9600, // Baudrate
      "Type": "RTU" // opções disponíveis (RTU or ASCII)
    },
    "SlaveID": 1, // identificador do slave
    "IntervalSave": 1000 // intervalo para salvar no banco de dados
  },
  "MQTT": { // configurações da conexão MQTT para envio dos sinais para o broker do IoTLog
    "Endpoint": "siot-broker-mqtt-dev.konztec.com.br", // endpoint do MQTT
    "Port": 8883, // porta MQTT
    "TLS": true // utiliza SSL/TLS (true ou false),
    "Auth": {
      "Username": "maquina_id", // ID da máquina no IoTLog
      "Password": "maquina_token" // Token da máquina no IoTLog
    }
  },
  "BUFFER": { // configurações do buffer de sinais
    "CronSend": "0/30 * * * * ?", // intervalo de tempo no formato CRON para verificar o buffer e enviar os sinais
    "TotalFiles": 10000 // limite de arquivos a serem enviados no intervalo
  },
  "IntervalSend": 10000 // Intervalo para envio dos sinais para o IoTLog,
  "DB": {
    "DaysRetainInDB": 3, // Quantidade de dias para excluir os dados do banco
    "Provider": "SQLLite", // Tipo de banco, nessa versão está disponível (SQLLite e MYSQL)
    "Connection": "Data Source=E:\\Data\\teste.db" // Conexão do banco de dados
  }
}
```

Configurar `config.modbus.json`:

```
{
  "idMachine": "maquina1", // ID da máquina no IoTLog
  "items": [
    {
      "address": 5, // Endereço do MODBUS
      "registers": 2, // Quantidade de registros
      "idSensor": "temperatura_agua", // ID do sensor no IoTLog
      "typeAddress": "inputRegister", // Tipo de endereço ("inputRegister", "holdingRegister", "coil")
      "isConvertToFloat": false // (* Opcional) Valor lido está em hexadecimal e será convertido em float
    },
    {
      "address": 5, // Endereço do MODBUS
      "registers": 2, // Quantidade de registros
      "idSensor": "temperatura_agua", // ID do sensor no IoTLog
      "typeAddress": "inputRegister", // Tipo de endereço ("inputRegister", "holdingRegister", "coil")
      "calcRange": { // (Opcional) Cálculo de interpolação linear para valores equivalentes
          "minRange": -20, // mínimo valor esperado
          "maxRange": 20, // máximo valor esperado
          "minSignal": 0, // mínimo valor endereço de memória
          "maxSignal": 32756, // máximo valor endereço de memória
          "precision": 1, // número de casas decimais
          "isCalculateRange": true, // se ativa o cálculo de interpolação
          "sizeDecimals": 1 // atua como potência de 10 para dividir sobre o valor resultante, por exemplo se for 2 é igual a 10² = o valor resultante será divido por 100 (que é igual a 10² = 10 x 10)
          "isMultiplySizeDecimals": true // caso true, o sizeDecimals vai atuar como multiplicação e não divisão
       }
    },
    {
       "idSensor": "gps", // ID do sensor no IoTLog
       "fieldsCompose": [ // Campos compostos que criam um objeto como valor do sinal
        {
          "propName":  "latitude", // nome da propriedade do objeto
          "address": 10, // Endereço do MODBUS
          "registers": 2, // Quantidade de registros
          "typeAddress": "inputRegister", // Tipo de endereço ("inputRegister", "holdingRegister", "coil")
          "isConvertToFloat": false // (* Opcional) Valor lido está em hexadecimal e será convertido em float
        },
        {
          "propName":  "longitude", // nome da propriedade do objeto
          "address": 12, // Endereço do MODBUS
          "registers": 2, // Quantidade de registros
          "typeAddress": "inputRegister", // Tipo de endereço ("inputRegister", "holdingRegister", "coil")
          "isConvertToFloat": false // (* Opcional) Valor lido está em hexadecimal e será convertido em float
        }
      ] // nesse caso como exemplo valor do sinal seria [15.0000, 45.155 ]
    }
  ]
}
```

Configurar `config.nmea.json` utilizando as setenças listadas abaixo:

```
{
  "idMachine": "maquina1", // ID da máquina no IoTLog
  "items": [
    {
      "sentence": "MTW", // Tipo de setença NMEA (Utilizar as listadas abaixo)
      "idSensor": "temperatura_agua", // ID do sensor no IoTLog
      "variable": "Degrees" // Propriedade da setença NMEA  (Utilizar a propriedade das sentenças listadas acima)
    },

    { // Modelo condicionado
      "sentence": "MWV", // Tipo de setença NMEA (Utilizar as listadas abaixo)
      "conditions": [
        {
          "when": {
            "variable": "Reference",
            "value": "T"
          },
          "then": {
            "idSensor": "angulo_vento_verdadeiro",
            "variable": "WindAngle"
          }
        },
        {
          "when": {
            "variable": "Reference",
            "value": "R"
          },
          "then": {
            "idSensor": "angulo_vento_relativo",
            "variable": "WindAngle"
          }
        }
      ]
    }
  ]
}
```

[Codecs NMEA](https://gpsd.gitlab.io/gpsd/NMEA.html) disponíveis até a versão atual e suas propriedades/tipo de variável:

- `GGA` **_Global Positioning System Fix Data_**
  - FixTime _TimeSpan_
  - Latitude _Float_
  - Longitude _Float_
  - NumberOfSatellites _Int_
  - HorizontalDilution _Float_
  - AltitudeUnits _String_
  - FixQuality _Int_
    - Invalid = 0
    - GpsFix = 1
    - DgpsFix = 2
    - PpsFix = 3
    - Rtk = 4
    - FloatRtk = 5
    - Estimated = 6
    - ManualInput = 7
    - Simulation = 8
  - Altitude _Float_
  - HeightOfGeoId _Float_
  - HeightOfGeoIdUnits _String_
  - TimeSpanSinceDgpsUpdate _Int_
  - DgpsStationId _Int_
- `HDT` **_Heading - True_**
  - Heading _Float_
- `VTG` **_Track made good and Ground speed_**
  - TrackTrue _Float_
  - TrackMagnetic _Float_
  - SpeedKnots _Float_
  - SpeedKmph _Float_
  - FaaMode _String_
- `DBK` **_Depth Below Keel_**
  - DepthFeet _Float_
  - DepthMeters _Float_
  - DepthFathoms _Float_
- `DBS` **_Depth Below Surface_**
  - DepthFeet _Float_
  - DepthMeters _Float_
  - DepthFathoms _Float_
- `DBT` **_Depth below transducer_**
  - DepthFeet _Float_
  - DepthMeters _Float_
  - DepthFathoms _Float_
- `DPT` **_Depth of Water_**
  - Depth _Float_
  - OffsetTransducer _Float_
  - MaximumRangeScale _Float_
- `GLL` **_Geographic Position - Latitude/Longitude_**
  - Latitude _Float_
  - Longitude _Float_
  - Time _TimeSpan_
  - Status _String_
  - FaaMode _String_
- `GSA` **_Active satellites and dilution of precision_**
  - SelectionMode _Int_
    - Auto = 0
    - Manual = 1
  - FixMode _Int_
    - NotAvailable = 1
    - Fix2D = 2
    - Fix3D = 3
  - SatelliteIDs _Int[]_
  - Pdop _Float_
  - Hdop _Float_
  - Vdop _Float_
- `MTW` **_Mean Temperature of Water_**
  - Degrees _Float_
  - UnitMeasurement _String_
- `MWD` **_Wind Direction_**
  - WindAngleTrue _Float_
  - WindAngleMagnetic _Float_
  - SpeedKnots _Float_
  - SpeedMps _Float_
- `MWV` **_Wind Speed and Angle_**
  - WindAngle _Float_
  - Speed _Float_
  - Reference _String_ _(R = Relative, T = True)_
  - Units _String_ _(K = km/hr, M = m/s, N = knots, S = statute)_
  - Status _Int_
    - Valid = 0
    - Invalid = 1
- `RMC` **_Recommended Minimum Navigation Information_**
  - DateTime _DateTime_
  - Status _String_
  - Latitude _Float_
  - Longitude _Float_
  - SpeedKnots _Float_
  - TrackTrue _Float_
  - Variation _Float_
  - VariationPole _String_ _(E or W)_
  - FaaMode _String_
- `RPM` **_Revolutions_**
  - Source _Int_
    - Shaft = 0
    - Engine = 1
  - SourceValue _Float_
  - SpeedMinute _Float_
  - PropellerPitch _Float_
  - Status _Int_
    - Valid = 0
    - Invalid = 1
- `VHW` **_Water speed and heading_**
  - DegreesTrue _Float_
  - DegreesMagnetic _Float_
  - SpeedKnots _Float_
  - SpeedKmph _Float_
- `ZDA` **_Time & Date - UTC, day, month, year and local time zone_**
  - DateTime _DateTime_
  - LocalZoneHours _Int_
  - LocalZoneMinutes _Int_

<br/>
<br/>

---

#### Windows

Executando como serviço

```
sc create MiddlewareIotLog binpath="C:\middleware\Middleware.WorkerService.App.exe C:\middleware" start=auto
```

---

#### Linux

Executando como serviço

Adicionar permissão para executar o arquivo extraído

```
chmod +x Middleware.WorkerService.App
```

Criar o arquivo `MiddlewareIoTLog.service` e adicionar os dados abaixo

```
sudo nano /etc/systemd/system/MiddlewareIoTLog.service
```

```
[Unit]
Description=Middleware IoTLog

[Service]
WorkingDirectory=/home/Middleware/
ExecStart=/home/Middleware/Middleware.WorkerService.App
SyslogIdentifier=MiddlewareIoTLog
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Reiniciar daemon e ativiar o serviço

```
sudo systemctl daemon-reload
sudo systemctl start MiddlewareIoTLog
sudo systemctl enable MiddlewareIoTLog
```

---

Conheça o IoTLog, acessando [Bykonz](https://www.bykonz.com/)
