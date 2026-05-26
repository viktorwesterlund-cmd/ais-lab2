# Identifierad attackyta — Chas OT-range

## Protokoll-nivå
1. **Modbus saknar autentisering.** Vem som helst på OT-nätet kan skriva till
   HR0 (Setpoint) lika lätt som HMI:n läser. Inga sessions-tokens, ingen
   ömsesidig autentisering, ingen integritetskontroll.

2. **Modbus är okrypterat.** Allt går i klartext. En angripare på trådsidan
   ser register-värden, function codes, och kan injicera egna paket.

3. **Reads larmar inte.** Detta är en avsiktlig designtrade-off — en angripare
   som "läser sig in" på systemet (FC1/2/3/4) syns inte i vår IDS-baseline.

## Nätverksnivå
4. **Dual-homed attacker-host.** Attacker-containern sitter på 10.0.10.99
   (IT) OCH 10.0.50.99 (OT). Detta är platt segmentering — en kompromiss
   på IT-sidan ger direkt tillgång till PLC:n. (Del 2 åtgärdar detta.)

5. **Inget skrivskydd mellan IT och OT.** Det finns ingen IT-OT-firewall,
   ingen DMZ, ingen data-diod. Bara en bridge.

## Tillämpningsnivå
6. **OpenPLC standardlösenord.** OpenPLC har en delad admin-login
   (openplc/openplc). Den som loggar in kan stoppa PLC:n eller ladda upp
   ny kod. (I rangen är /plc/ instruktör-låst, men det är en lokal mitigation.)

7. **HMI:n saknar autentisering till PLC:n.** Modbus-klienten i Node-RED
   skickar inga creds. (Naturligt — det finns inga creds i protokollet.)

## Övergripande
8. **Ingen loggning av writes på applikationsnivån.** Suricata loggar dem,
   men det finns ingen audit-log i PLC:n själv som säger "vem ändrade
   setpointen och när".

9. **Inget incident response-flöde.** Larmet i Suricata syns på en webbsida.
   Det finns ingen escalation, ingen ticket, inget pager-flöde. (Del 4
   bygger detta.)

## Vad jag rangordnar högst (med motivering)
| # | Risk            | Sannolikhet | Konsekvens | Motivering |
|---|----------------|-------------|------------|------------|
| 4 | Dual-homing    | Hög         | Hög        | Direkt vägen in |
| 1 | Saknad authn   | Hög         | Hög        | Protokollet är så |
| 6 | Default creds  | Hög         | Hög        | Kända offentligt |
| 2 | Klartext       | Medel       | Medel      | Kräver MITM-position |
| 8 | Ingen audit-log| Medel       | Medel      | Postmortem-problem |
