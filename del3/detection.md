# Lab 2 — Del 3: Detektion med Suricata

## Sammanfattning
Suricata IDS på OT-bryggan med fyra startregler + 2 egna.
Triggar alarm på samtliga Modbus-writes (FC5/FC6/FC16) samt på
nya TCP-sessioner till port 502. Egna regler täcker FC8 Diagnostics
och setpoint-värden utanför säkert intervall.

## Övervakad attackytan
- 100% av Modbus-writes detekteras (det finns ingen legitim
  write-trafik i miljön — alla writes är per definition misstänkta)
- Nya sessioner till 502 detekteras — ger spårbarhet för recon
- DoS-attacker via FC8 detekteras
- Värden utanför säkert intervall fångas innan PLC:n agerar

## Regler — översikt
| SID     | Typ        | Trigger | Allvarlighet |
|---------|-----------|---------|--------------|
| 2000005 | FC5       | Force Single Coil | High (priority 2) |
| 2000006 | FC6       | Write Single Register | High (priority 1) |
| 2000016 | FC16      | Write Multiple Registers | Critical (priority 1) |
| 2001000 | TCP SYN   | Ny session :502 | Medium |
| 1000100 | FC8 (min) | Diagnostics — potential DoS | Critical |
| 1000101 | FC6 (min) | Setpoint > 8000 (unsafe) | Critical |

## Bevis
- `del3/fast.log` (<img width="950" height="60" alt="attack-fc6 py outpu" src="https://github.com/user-attachments/assets/7b083f0c-2f52-49b6-98d4-85a10c06b8d4" />) — alla attacker fångade
- `del3/alert-anatomy.json` () — exempel-eve för FC6
- `screenshots/fast-log-attacks.png` ()
- `del3/ot.rules` (<img width="705" height="1182" alt="del3ot rules" src="https://github.com/user-attachments/assets/e9a5fdc0-6933-4a0c-8147-579a86caede0" />
) — min utökade regeluppsättning
- `del3/verify-output.txt` (<img width="679" height="400" alt="verify" src="https://github.com/user-attachments/assets/f3c4ee87-e78a-4e69-b524-8cfc6b5fe5c9" />
) — output från `./verify-detection.sh`

## Tradeoffs och false positives
- Vi alarmerar på *alla* writes — fungerar här eftersom det inte
  finns legitima writes. I produktion behövs en allowlist av kända
  writers (t.ex. via `flow.src` per regel) annars drunknar SOC:en
  i larm från legitima engineering-stationer.
- Setpoint-värdes-regeln (1000101) är processpecifik — den måste
  uppdateras när processgränserna ändras.
- Nya-session-regeln (2001000) larmar även på HMI:ns omanslutning
  efter omstart — kan göra om till `dsize:0` eller liknande.

## Vad detta INTE skyddar mot
- Kompromissad HMI som börjar göra FC6 — ser ut som legitim trafik
- Reads-baserade angrepp (skanning, datalek)
- Modbus-over-TLS — vi inspekterar inte krypterat lager
- Lateral rörelse inne i OT — vi sniffar bara mot PLC:n

## Källor
- [Suricata Modbus keyword](https://docs.suricata.io/en/latest/rules/modbus-keyword.html)
- Sandbox: `r87-e/ais-lab2-sandboxes/del3/`
