# ICS-arkitektur — Chas OT-range (snabbspår)

## Sammanfattning
En enkel vattenrenings-PLC styrd via Modbus TCP. En HMI läser sensorer 1×/s
och visar dem i en dashboard. En Suricata-instans sniffar trafiken mellan
IT-segmentet och OT-segmentet.

## Komponenter
| Komponent      | Roll              | IP-adress     | Port  | Protokoll      |
|----------------|------------------|---------------|-------|----------------|
| OpenPLC        | PLC runtime       | 10.0.50.10    | 502   | Modbus TCP     |
| OpenPLC web    | PLC config        | 10.0.50.10    | 8080  | HTTP           |
| Node-RED HMI   | HMI/SCADA         | 10.0.50.20    | 1880  | HTTP (dashboard)|
| Suricata       | IDS               | host          | -     | sniffar br-ot  |
| Attacker       | (dual-homed)      | 10.0.10.99 / 10.0.50.99 | - | - |

## Nätverkssegmentering
- `br-ot` (10.0.50.0/24): PLC + HMI
- `br-it` (10.0.10.0/24): attacker-arbetsstationen
- HMI är dual-homed (10.0.50.20 + 10.0.10.20) — den behöver vara nåbar från IT
  men prata med PLC på OT-sidan. Detta är *avsiktligt designat* så.
- Attacker är dual-homed (10.0.10.99 + 10.0.50.99) — detta är *fel design* och
  representerar dålig segmentering som Lab 2 Del 2 åtgärdar.

## Modbus mapping
HR0 = Setpoint, HR1 = TankLevel, HR2 = PumpSpeed, HR3 = ChlorineLevel
Coil 0 = DosingValve, Coil 1 = Alarm

## Kommunikationsflöden
- HMI → PLC: FC3, polling 1×/s, register 0..3
- (Avsikten i normaldrift: inga writes)
- Operatör → HMI: HTTPS via nginx-proxy + basic auth

## Purdue-modellen — mappning
| Nivå | Vad det är           | Vår motsvarighet |
|------|----------------------|------------------|
| 0    | Fysisk process       | (simulerad)      |
| 1    | PLC / kontrollers    | OpenPLC          |
| 2    | HMI / SCADA          | Node-RED HMI     |
| 3    | Site operations      | (saknas)         |
| 4    | Företags-IT          | Attacker-host    |
| 5    | Internet             | (saknas)         |

## Bilder
- ![baseline HMI](../screenshots/hmi-baseline.png)
- ![baseline alerts](../screenshots/alerts-baseline.png)
