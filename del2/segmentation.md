# Lab 2 — Del 2: IT/OT-nätverkssegmentering

## Sammanfattning
Tre Docker-nätverk (IT, DMZ, OT) som speglar Purdue-modellens nivåer.
Attacker isolerad till IT-zonen. Jump-server är enda vägen från IT till OT.

## Identifierad brist (från Del 1)
Sandboxen levererades med `attacker`-containern dual-homed på både `it`
och `ot`. Det innebär att en angripare i IT-zonen kunde nå
`mock-plc:502` direkt — i strid med Purdue-regel "IT pratar aldrig
direkt med OT".

## Korrigerande åtgärd
Tog bort `ot`-nätverket från `attacker`-tjänsten i `docker-compose.yml`.
Docker behåller därmed routing-isolation på L2-nivå — paket från
attacker kan inte längre nå OT-nätverket eftersom containern saknar
ett interface dit.

## Resulterande regelmatris

| Från     | Till         | Förväntat | Faktiskt |
|----------|--------------|-----------|----------|
| attacker | mock-plc:502 | BLOCK     | BLOCK ✓  |
| attacker | jump-srv:22  | ALLOW     | ALLOW ✓  |
| jump-srv | mock-plc:502 | ALLOW     | ALLOW ✓  |
| historian| mock-plc:502 | ALLOW     | ALLOW ✓  |
| historian| internet     | ALLOW     | ALLOW ✓  |
| mock-plc | internet     | BLOCK     | BLOCK ✓  |

Bevis: se `del2/verify-output.txt`

## Designval och tradeoffs
- **Docker network isolation** istället för iptables: enklare att
  resonera om, ger zone-separation på L2 utan kernel-regler. I en
  riktig miljö skulle vi också ha iptables/nftables eller en
  next-gen firewall mellan zonerna för djupare kontroll.
- **Jump-server placerad i DMZ** med ben i OT: bastion-mönstret —
  en kontrollpunkt med audit, MFA-möjlighet, sessionsinspelning.
- **Historian i DMZ**: läser produktionsdata från PLC men exponerar
  data åt IT-sidan via en separat kanal (ej implementerad här —
  Del 3-territorium).

## Vad detta INTE skyddar mot
- Komprometterad jump-server är fortfarande en bridgehead in i OT
- Inga regler stoppar OT-internt malicious-trafik (PLC → HMI)
- Saknar idag övervakning — utan IDS upptäcker vi inte överträdelser
  (det är vad Del 3 löser)

## Källor
- Purdue Enterprise Reference Architecture (kort introduktion på sista
  föreläsningen)
- ISA/IEC 62443 — zoner och conduits
