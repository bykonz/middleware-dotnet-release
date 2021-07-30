# Middleware MODBUS e NMEA - S.IoT
Middleware de integração de NMEA e MODBUS para SIOT via protocolo padrão MQTT

### Instalação execução

Necessário instalar [.NET 5](https://dotnet.microsoft.com/download/dotnet/5.0)


Realizar o download da última versão do [Middleware.WorkerService.App](https://github.com/konztec/middleware-dotnet-release/releases/download/2.0.7/Middleware.WorkerService.App.2.0.7.zip)

Extrair os arquivos compactados em um diretório.

Verifique a documentação do [cadastro e geração de token no SIoT](https://blog.konztec.com/cadastro-maquina-token-integracao-siot/) para obter a `maquina_id` e `maquina_token` 

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
  "MODBUS": { // configurações de conexão com o NMEA
    "Mode": "TCP", // modo de comunicação disponível (TCP ou SERIAL) 
    "TCP": { // configurações para o modo TCP, informar apenas se o `Mode: TCP`
      "Endpoint": "127.0.0.1", // Endpoint TCP
      "Port": 502 // Porta TCP
    },
    "Serial": { configurações para o modo SERIAL, informar apenas se o `Mode: SERIAL`
      "Port": "COM2", // porta serial
      "Baudrate": 9600, // Baudrate
      "Type": "RTU" // opções disponíveis (RTU or ASCII)
    },
    "SlaveID": 1, // identificador do slave
    "IntervalSave": 1000 // intervalo para salvar no banco de dados
  },
  "MQTT": { // configurações da conexão MQTT para envio dos sinais para o broker do SIoT
    "Endpoint": "siot-broker-mqtt-dev.konztec.com.br", // endpoint do MQTT
    "Port": 8883, // porta MQTT
    "TLS": true // utiliza SSL/TLS (true ou false),
    "Auth": {
      "Username": "maquina_id", // ID da máquina no SIoT
      "Password": "maquina_token" // Token da máquina no SIoT 
    }
  },
  "BUFFER": { // configurações do buffer de sinais
    "CronSend": "0/30 * * * * ?", // intervalo de tempo no formato CRON para verificar o buffer e enviar os sinais
    "TotalFiles": 10000 // limite de arquivos a serem enviados no intervalo
  },
  "IntervalSend": 10000 // Intervalo para envio dos sinais para o SIoT,
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
  "idMachine": "maquina1", // ID da máquina no SIoT
  "items": [
    {
      "address": 5, // Endereço do MODBUS
      "registers": 2, // Quantidade de registros
      "sensorId": "temperatura_agua", // ID do sensor no SIoT
      "signalId": "atual", // ID do sinal no SIOT,
      "typeAddress": "register", // Tipo de endereço ("register", "coil")
      "isConvertToFloat": false // (* Opcional) Valor lido está em hexadecimal e será convertido em float
    },
    {
       "sensorId": "gps", // ID do sensor no SIoT
       "signalId": "coordinate", // ID do sinal no SIOT,
       "fieldsCompose": [ // Campos compostos que criam um objeto como valor do sinal
        {
          "propName":  "latitude", // nome da propriedade do objeto
          "address": 10, // Endereço do MODBUS
          "registers": 2, // Quantidade de registros
          "typeAddress": "register", // Tipo de endereço ("register", "coil")
          "isConvertToFloat": false // (* Opcional) Valor lido está em hexadecimal e será convertido em float
        },
        {
          "propName":  "longitude", // nome da propriedade do objeto
          "address": 12, // Endereço do MODBUS
          "registers": 2, // Quantidade de registros
          "typeAddress": "register", // Tipo de endereço ("register", "coil")
          "isConvertToFloat": false // (* Opcional) Valor lido está em hexadecimal e será convertido em float
        }
      ] // nesse caso como exemplo valor do sinal seria { "latitude": 15.0000, "longitude": 45.155 }
    }
  ]
}
```

Configurar `config.nmea.json` utilizando as setenças listadas abaixo:
```
{ 
  "idMachine": "maquina1", // ID da máquina no SIoT
  "items": [
    {
      "sentenceId": "MTW", // Tipo de setença NMEA (Utilizar as listadas abaixo)
      "sensorId": "temperatura_agua", // ID do sensor no SIoT
      "signals": [
        {
          "signalId": "atual", // ID do sinal no SIoT
          "packet": "Degrees" // Propriedade da setença NMEA  (Utilizar a propriedade das sentenças listadas acima)
        }
      ]
    }
  ]
}
```

[Codecs NMEA](https://gpsd.gitlab.io/gpsd/NMEA.html) disponíveis até a versão atual e suas propriedades/tipo de variável:
   - `GGA` ***Global Positioning System Fix Data***
      - FixTime *TimeSpan*
      - Latitude *Float*
      - Longitude *Float*
      - NumberOfSatellites *Int*
      - HorizontalDilution *Float*
      - AltitudeUnits *String*
      - FixQuality *Int* 
        - Invalid = 0
        - GpsFix = 1
        - DgpsFix = 2
        - PpsFix = 3
        - Rtk = 4
        - FloatRtk = 5
        - Estimated = 6
        - ManualInput = 7
        - Simulation = 8
      - Altitude *Float*
      - HeightOfGeoId *Float*
      - HeightOfGeoIdUnits *String*
      - TimeSpanSinceDgpsUpdate *Int*
      - DgpsStationId *Int*
   - `HDT` ***Heading - True***
      - Heading *Float*
   - `VTG` ***Track made good and Ground speed***
      - TrackTrue *Float*
      - TrackMagnetic *Float*
      - SpeedKnots *Float*
      - SpeedKmph *Float*
      - FaaMode  *String*
   - `DBK` ***Depth Below Keel***
      - DepthFeet *Float*
      - DepthMeters *Float*
      - DepthFathoms *Float*   
   - `DBS` ***Depth Below Surface***
      - DepthFeet *Float*
      - DepthMeters *Float*
      - DepthFathoms *Float*
   - `DBT` ***Depth below transducer***
      - DepthFeet *Float*
      - DepthMeters *Float*
      - DepthFathoms *Float*
   - `DPT` ***Depth of Water***
      - Depth *Float*
      - OffsetTransducer *Float*
      - MaximumRangeScale *Float*
   - `GLL` ***Geographic Position - Latitude/Longitude***
      - Latitude *Float*
      - Longitude *Float*
      - Time *TimeSpan*
      - Status *String*
      - FaaMode *String*
   - `GSA` ***Active satellites and dilution of precision***
      - SelectionMode *Int*
        - Auto = 0
        - Manual = 1
      - FixMode *Int*
        - NotAvailable = 1
        - Fix2D = 2
        - Fix3D = 3
      - SatelliteIDs *Int[]*
      - Pdop *Float*
      - Hdop *Float*
      - Vdop *Float*
   - `MTW` ***Mean Temperature of Water***
      - Degrees *Float*
      - UnitMeasurement *String*
   - `MWD` ***Wind Direction***
      - WindAngleTrue *Float*
      - WindAngleMagnetic *Float*
      - SpeedKnots *Float*
      - SpeedMps *Float*
   - `MWV` ***Wind Speed and Angle***
      - WindAngle *Float*
      - Speed *Float*
      - Reference *String* *(R = Relative, T = True)*
      - Units *String* *(K = km/hr, M = m/s, N = knots, S = statute)*
      - Status *Int*
        - Valid = 0
        - Invalid = 1
   - `RMC` ***Recommended Minimum Navigation Information***
      - DateTime *DateTime*
      - Status *String*
      - Latitude *Float*
      - Longitude *Float*
      - SpeedKnots *Float*
      - TrackTrue *Float*
      - Variation *Float*
      - VariationPole *String* *(E or W)*
      - FaaMode *String*
   - `RPM` ***Revolutions***
      - Source *Int*
        - Shaft = 0
        - Engine = 1
      - SourceValue *Float*
      - SpeedMinute *Float*
      - PropellerPitch *Float*
      - Status *Int*
        - Valid = 0
        - Invalid = 1
   - `VHW` ***Water speed and heading***
      - DegreesTrue *Float*
      - DegreesMagnetic *Float*
      - SpeedKnots *Float*
      - SpeedKmph *Float*
   - `ZDA` ***Time & Date - UTC, day, month, year and local time zone***
      - DateTime *DateTime*
      - LocalZoneHours *Int*
      - LocalZoneMinutes *Int*

<br/>
<br/>

Para executar como serviço no Windows
```
sc create MidSiot binpath="C:\middleware\Middleware.WorkerService.App.exe C:\middleware\" start=auto
```

 Mais informações e integrações com S.IoT, acessando [Blog Documentações](https://blog.konztec.com/documentacao)

 Conheça o S.IoT, acessando [S.IoT](https://www.konztec.com/)