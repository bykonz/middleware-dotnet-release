# Middleware MODBUS and NMEA - IoTLog

NMEA and MODBUS integration middleware for IoTLog via standard MQTT protocol

### Install and run

Need to install [.NET 6](https://dotnet.microsoft.com/download/dotnet/6.0)


Download the latest version of [Middleware.WorkerService.App](https://github.com/bykonz/middleware-dotnet-release/releases)

Extract the compressed files into a directory.

Configure environment variables in `appsettings.json`
```
{
  "NMEA": { // NMEA connection settings
    "Mode": "SERIAL", // connection mode (use SERIAL or TCP)
    "Serial": { // settings for SERIAL mode, inform only if `Mode: SERIAL`
      "Port": "COM2", // Serial Port
      "Baudrate": 9600, // Baudrate
      "Limiter": "\n" // Caracter limitador
    },
    "TCP": { // settings for TCP mode, inform only if `Mode: TCP`
      "Endpoint": "localhost", // TCP Endpoint
      "Port": 3000 // TCP Port
    },
    "IntervalSave": 1000 // interval to save in database
  },
  "MODBUS": { // MODBUS connection settings
    "Mode": "TCP", // connection mode (use SERIAL or TCP)
    "TCP": { //  settings for TCP mode, inform only if `Mode: TCP`
      "Endpoint": "127.0.0.1", // Endpoint TCP
      "Port": 502 // Porta TCP
    },
    "Serial": { //settings for SERIAL mode, inform only if `Mode: SERIAL`
      "Port": "COM2", // Serial port
      "Baudrate": 9600, // Baudrate
      "Type": "RTU" // Allow options (RTU or ASCII)
    },
    "SlaveID": 1, // Slave id
    "IntervalSave": 1000 // interval to save in database
  },
  "MQTT": { // MQTT connection settings for sending signals to the IoTLog broker
    "Endpoint": "siot-broker-mqtt-dev.konztec.com.br", // MQTT endpoint
    "Port": 8883, // MQTT port
    "TLS": true // using SSL/TLS? (true ou false),
    "Auth": {
      "Username": "maquina_id", // ID machine in IoTLog
      "Password": "maquina_token" // Token machine in IoTLog 
    }
  },
  "BUFFER": { // sensor buffer settings
    "CronSend": "0/30 * * * * ?", // time interval in CRON format to check the buffer and send the signals of sensors
    "TotalFiles": 10000 // limit of files to send in range
  },
  "IntervalSend": 10000 // interval for sending signals to IoTLog
  "DB": {
    "DaysRetainInDB": 3, // Number of days to delete data of the database
    "Provider": "SQLLite", // Database type, in this version is available (SQLLite and MYSQL)
    "Connection": "Data Source=E:\\Data\\teste.db" // Database string connection
  }
}
```

Configure `config.modbus.json` case the protocol is 'MODBUS':
```
{ 
  "idMachine": "maquina1", // ID machine in IoTLog
  "items": [
    {
      "address": 5, // MODBUS address
      "registers": 2, // Number of registers
      "idSensor": "temperatura_agua", // ID sensor in IoTLog
      "typeAddress": "register", // Type of address (use "register" or "coil")
      "isConvertToFloat": false // (* Optional) Value read is in hexadecimal and will be converted to float
    },
    {
       "idSensor": "gps", // ID sensor in IoTLog
       "fieldsCompose": [ // Composite fields that create an object as a signal value
        {
          "propName":  "latitude", // object property name
          "address": 10, // MODBUS address
          "registers": 2, // Number of registers
          "typeAddress": "register", // Type of address (use "register" or "coil")
          "isConvertToFloat": false // (* Optional) Value read is in hexadecimal and will be converted to float
        },
        {
          "propName":  "longitude", // object property name
          "address": 12, // MODBUS address
          "registers": 2, // Number of registers
          "typeAddress": "register", // Type of address (use "register" or "coil")
          "isConvertToFloat": false // (* Optional) Value read is in hexadecimal and will be converted to float
        }
      ] // in this case as an example generate signal value is [15.0000, 45.155 ]
    }
  ]
}
```

Configure `config.nmea.json` case the protocol is 'NMEA' using the sentences listed below:
If you need a new sentence, please enter in contact with our team.
```
{ 
  "idMachine": "maquina1", // ID machine in IoTLog
  "items": [
    {
      "sentence": "MTW", // Sentence type in NMEA (Use the ones listed below)
      "idSensor": "temperatura_agua", // ID sensor in IoTLog
      "variable": "Degrees" // Propriedade da setença NMEA  NMEA Sentence Property (Use the sentence property listed below)
    },

    { // Conditioned model
      "sentence": "MWV", // Sentence type in NMEA (Use the ones listed below)
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

[Codecs NMEA](https://gpsd.gitlab.io/gpsd/NMEA.html) available up to the current version and their properties/variable type:
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

To run as a service on Windows
```
sc create MiddlewareIoTLog binpath="C:\middleware\Middleware.WorkerService.App.exe C:\middleware" start=auto
```

 Get to know IoTLog by accessing [Bykonz](https://www.bykonz.com/)
