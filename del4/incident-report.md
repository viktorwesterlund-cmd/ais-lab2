# ICS-CERT Incidentrapport — Lab 2 Del 4

## 1. Sammanfattning
- **Rapportdatum:** 2026-05-2X
- **Rapportör:** [DITT NAMN]
- **Miljö:** Lab 2 Del 3-sandbox (lokal)
- **Incidenttyp:** Obehörig processmanipulering via Modbus TCP från komprometterad bastion
- **Allvarlighetsgrad:** HÖG
- **Påverkade system:** Mock PLC (172.31.50.10:502), processtyrning
- **Status:** Löst, root cause identifierad

## 2. Tidslinje (UTC)
| Tid           | Händelse |
|---------------|----------|
| T+0           | Angripare initierar nmap-skan från jump-server |
| T+15s         | FC16 bulk-write till HR0..3 (Suricata sid:2000016) |
| T+25s         | FC6 maskering — Setpoint=5500 (sid:2000006) |
| T+1m          | IR-person upptäcker larmkaskaden i fast.log |
| T+1m30s       | Triage: angripar-IP 172.31.50.99 (jump-server) bekräftad |
| T+2m          | Nödåterställning Setpoint=5000 |
| T+2m15s       | Containment: jump-server kopplad från OT-nätet |
| T+5m          | Evidence-insamling klar |
| T+8m          | Recovery via historian — process tillbaka i normaldrift |
| T+12m        | Eradicate: jump-server rebyggd från ren image |

## 3. Detektion
- **Larm-mekanism:** Suricata IDS på OT-bryggan (`br-lab3-ot`)
- **Triggrade SID:** 2001000 (nya sessioner), 2000016 (FC16), 2000006 (FC6)
- **Larm-tid:** sub-sekund från attacken
- **Detektionstid (T_detect - T_attack):** ~1 minut (manuell observation av fast.log)

## 4. Påverkan
- **Processäkerhet:** Setpoint höjt till 9999 i 25 sekunder. Eftersom TankLevel inte hade tid att klättra dit, ingen verklig konsekvens denna gång. I produktion: tank-overfill möjlig.
- **Tillgänglighet:** Jump-server offline i 8 minuter (containment).
- **Konfidentialitet:** Process-register läckte via FC3-recon (inga regler larmade reads).
- **Integritet:** HR0..3 och CO0 manipulerade. Återställd via historian.

## 5. Root cause
- Stulna creds på jump-server (simulerat — i sandbox redan exponerade)
- Saknad MFA på bastion
- Saknad audit-log över SSH-sessioner
- Modbus-protokollet stödjer ingen autentisering eller kryptering

## 6. Containment-strategi (val och motivering)
**Vald åtgärd:** `docker network disconnect lab2-del3-sandbox_ot lab3-jump`

**Motivering:** "Soft kill" — angriparens väg stängs, men jump-servern överlever för forensik. HMI och historian fortsätter fungera så processövervakning bibehålls.

**Alternativ jag valde bort:**
- `docker stop lab3-jump` — förlorade bash-historiken och process-tillstånd
- Stäng hela OT-nätet — hade stoppat HMI-pollern och blindat oss

## 7. Forensiska luckor
- Ingen pcap fanns före attacken (bara alerted packets i eve.json)
- Ingen Modbus-audit log på PLC-sidan (protokollet stödjer inte)
- Jump-servers SSH-server loggar inte sessioner i denna sandbox-konfig

## 8. Rekommendationer (prioriterade)
1. **MFA på jump-server** — hade stoppat creds-stölden i sin första fas
2. **Session-recording av alla SSH-sessioner** — `asciinema rec` eller kommersiell PAM
3. **IP-allowlist på sshd** — bara från IT-zonens specifika hosts
4. **Modbus write-allowlist på Suricata** — bara historian + officiella engineering workstations får skriva
5. **FC8 + recon-detection rules** — lagt till sid 1000100 och 1000200 i Del 3+4
6. **Full pcap-fångst i OT-DMZ** — kostar disk men kritiskt för forensik
7. **Process-side safety interlocks** — PLC borde själv vägra Setpoint > 8000 (skyddslager utanför nätverksskiktet)

## 9. Lessons learned
- Detection without prevention är otillräckligt — vi såg attacken men hindrade den inte
- "Soft kill" via network disconnect var rätt val här, men hade krävts ett beslut på sekunder i en real OT-miljö
- Tidslinjen från attack till containment var 2m15s — i en producerande miljö kanske kritiskt långsamt. Behov av automation (active response).

## 10. Bevis (artefakter i `evidence/`)
- `fast.log` — alla Suricata-alerts
- `eve.json` — strukturerad version med payloads
- `jump-history.txt` — angriparens kommandon
- `plc-state.txt` — register-tillstånd vid containment
- `timeline.txt` — extrakt från fast.log
- `ot-network.json` + `compose-state.txt` — miljötillstånd
