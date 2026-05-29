# Reflektion – Lab 2

Lab 2 gav en tydlig bild av hur OT-säkerhet skiljer sig från traditionell IT-säkerhet. Den största skillnaden jag upplevde var att 
konsekvenserna av en attack direkt påverkar en fysisk process. I labben såg jag hur ändringar i PLC-register påverkade tanknivåer, 
pumpstyrning och klorvärden. I en vanlig IT-miljö handlar incidenter ofta om data, system eller tjänster, medan en OT-incident kan leda 
till fysiska konsekvenser som produktionsstopp eller säkerhetsrisker.

En andra skillnad är att tillgänglighet och stabil drift prioriteras högre i OT-miljöer. Under incidenthanteringen blev det tydligt att 
man inte bara kan stänga av system eller starta om komponenter utan att riskera att påverka processen. I många IT-miljöer är det vanligt 
att isolera eller stänga av komprometterade system, men i OT måste man väga säkerhetsåtgärder mot driftsäkerhet.

Den tredje skillnaden är användningen av industriprotokoll som Modbus. I labben såg jag hur angriparen kunde manipulera PLC-register 
genom relativt enkla kommandon. Protokollet saknar många av de säkerhetsfunktioner som är standard i moderna IT-protokoll, vilket gör 
nätverksövervakning och segmentering extra viktiga.

Den del som lärde mig mest var Del 4, där fokus låg på incidenthantering och forensik. Att analysera Suricata-larm, identifiera 
angriparens IP-adress och undersöka PLC:ns tillstånd gjorde att jag fick en bättre förståelse för hur en verklig OT-incident kan utredas. 
Jag tyckte också att denna del kändes mest realistisk eftersom den kombinerade teknisk analys med beslutsfattande kring risker och 
bevisinsamling.

Om jag imorgon fick i uppdrag att säkra en riktig vattenrenings-PLC skulle jag börja med tre åtgärder. För det första skulle jag 
segmentera nätverket och begränsa åtkomsten till PLC:n så att endast auktoriserade system kan kommunicera med den. För det andra skulle 
jag införa övervakning och larm för Modbus-trafik, exempelvis med Suricata eller liknande IDS-lösningar, för att snabbt upptäcka 
avvikande aktivitet. För det tredje skulle jag stärka autentisering och åtkomstkontroll kring administrativa system och jump-servrar 
eftersom labben visade hur en komprometterad jump-server kan ge en angripare direkt åtkomst till OT-miljön.

Det svåraste att försvara mot i den arkitektur vi arbetade med är enligt mig legitima kommandon som skickas av en angripare som 
redan har fått åtkomst till rätt system. När angriparen använder korrekta protokoll och funktioner kan trafiken se normal ut trots att 
syftet är skadligt. I labben användes giltiga Modbus-funktioner för att manipulera processen, vilket illustrerar hur svårt det kan vara 
att skilja mellan legitim drift och ett pågående angrepp. Därför räcker det inte med traditionellt perimeterskydd; man behöver också 
övervakning, baslinjer för normalt beteende och tydliga processer för incidenthantering.

Sammanfattningsvis gav Lab 2 en praktisk förståelse för både tekniska och operativa utmaningar inom OT-säkerhet. Den visade att 
skydd av industriella system kräver ett annat perspektiv än traditionell IT-säkerhet, där säker drift och fysiska konsekvenser står i 
centrum.
