# PLC — water_treatment.st (sammanfattning)

## Adresskarta
| Tag    | Modbus           | Variabel       | Init |
|--------|------------------|----------------|------|
| %QW0   | HR0              | Setpoint       | 5000 |
| %QW1   | HR1              | TankLevel      | 3000 |
| ...    | ...              | ...            | ...  |

## Logik (sammanfattning på 3 rader)
1. ...
2. ...
3. ...

## Säkerhetsobservationer (kort)
- Setpoint är ett vanligt holding register — ingen extra skrivskydd
- Larmet är reaktivt, inte preventivt — det går först EFTER att TankLevel skjutit över
- ...


# HMI — Node-RED Modbus-klient

## Anslutning
- Host: 10.0.50.10
- Port: 502 (Modbus TCP)
- Unit ID: 1

## Polling
- FC3 (Read Holding Registers)
- Adress 0, Quantity 4 (läser HR0–HR3 i en bulk-fråga)
- Rate: 1 sekund

## Mapping i HMI
HR0 → Setpoint gauge
HR1 → TankLevel gauge + trend
HR2 → PumpSpeed gauge
HR3 → ChlorineLevel gauge
