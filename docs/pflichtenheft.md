# taggd – Pflichtenheft

**Team:** Erik, Almir, Lukas, Julian

---

## 1. Pflichtenheft „Was mache ich"

### 1.1 Ausgangssituation

Jugendliche und junge Erwachsene verbringen einen Großteil ihrer Freizeit bewegungsarm vor Bildschirmen.

taggd ist ein **ortsbasiertes Spiel (Location-based Game, LBG)**: „Fangen", aber in der Größe eines Stadtteils. Gespielt wird **ausschließlich zu Fuß**.

### 1.2 Istzustand

- Wenn Fangspiele im städtischen Raum organisiert werden, greifen Gruppen auf Messenger-Apps zurück (z. B. Standortbild über WhatsApp).
- Ob ein Fang zählt, wer sich wo aufhält und ob alle fair spielen, wird von den Spielern selbst ausgehandelt.
- Für taggd wurde bereits ein **Proof of Concept** umgesetzt, der zeigt, dass sich Handys über Bluetooth gegenseitig erkennen und der Server sie den richtigen Spielern zuordnen kann.

### 1.3 Problemstellung

- **GPS ist für einen echten Fang zu ungenau:** zwei Handys können jeweils 5–10 m danebenliegen. Ein Fang allein über GPS wäre unfair.
- **Kein neutraler Schiedsrichter:** Niemand kann prüfen, ob ein Fang gültig war, ob jemand mit Fahrrad, Scooter oder Auto unterwegs ist oder ob geschummelt wird.
- **Sicherheit im öffentlichen Raum:** Im Eifer des Spiels achten Spieler weniger auf den Verkehr. Gefährliche Bereiche (Autobahnen, Gleise, Gewässer) müssen erkannt werden.
- **Camping:** Wer sich das ganze Spiel zu Hause versteckt, darf damit keinen Vorteil haben.
- **Datenschutz:** Standortdaten sind personenbezogene Daten und dürfen nicht dauerhaft gespeichert werden.
- **Handy in der Hosentasche:** Das Spiel muss auch bei gesperrtem Bildschirm zuverlässig weiterlaufen.

### 1.4 Aufgabenstellung

Es wird eine Mobile-App (Android und iOS) samt Server entwickelt, mit der Spieler in zwei Rollen gegeneinander antreten:

|Rolle|Sieht|Ziel|
|---|---|---|
|**Hunter**|Hunted-Spieler als **Pings** (Standort-Momentaufnahmen), standardmäßig alle 2 min, schneller bei Annäherung (siehe [Dynamische Pings](#dynamische-pings))|Alle Hunted-Spieler taggen|
|**Hunted**|Hunter jederzeit **live**|So lange wie möglich überleben|

Jeder sieht seinen **eigenen Live-Standort** und die Pings/Positionen, die seine Rolle erlaubt.

#### 1.4.1 Funktionale Anforderungen

##### Spielmodi

| |Privat|Public|Ranked|
|---|---|---|---|
|Wer tritt bei|Nur eingeladene Freunde|Matchmaking|Matchmaking|
|Einstellungen|Host kann alles ändern|Fix|Fix|
|Verlassen|Jederzeit|Jederzeit|Nicht erlaubt → zählt als Niederlage + Elo-Strafe|
|Coins|❌ keine|✅ bleiben auch bei frühem Verlassen erhalten|✅|
|Dynamisches Matchmaking|❌|✅ laufenden Spielen beitreten / hinzugefügt werden (nur in den ersten 15 min)|❌|
|Bots|Optional, Host entscheidet|✅ füllen kleine Lobbys auf|❌|
|Elo|❌|❌|✅ gewinnen / verlieren, Ranked-Rangliste|
|Fokus|Freunde|Kennenlernen|Wettbewerb|

- Bots sind immer klar als Bots gekennzeichnet.
- **Public-Spiele:** Verlassen alle Hunter das Spiel, übernimmt ein Bot-Hunter, damit das Spiel weiterlaufen kann.
- Der Elo-Gewinn ist relational, wie beim Coin-System: Siege gegen stärkere Spieler bringen mehr (siehe [Elo](#elo)).

##### Kernmechaniken

- **Hunter-Anteil:** standardmäßig sind 30 % der Spieler Hunter.
- **Taggen:** Der Hunter nähert sich an und **lockt sich ein** (Lock-on), indem er den Tag-Button gedrückt hält. Der Tag zählt nur, wenn **Bluetooth** bestätigt, dass die beiden Handys nur wenige Meter voneinander entfernt sind. Der Hunted bekommt eine **Lock-on-Warnung**. Details: [Taggen (Lock-on)](#taggen-lock-on).
- **Bluetooth Pflicht:** Bluetooth muss eingeschaltet sein, um einem Spiel beizutreten. Die App prüft das in der Lobby.
- **Getaggte Spieler** scheiden aus und behalten ihre verdienten Coins.
- **Siegbedingungen:**
    - **Hunter gewinnen**, wenn alle Hunted getaggt wurden.
    - **Hunted gewinnen**, wenn mindestens einer von ihnen bis zum Ende des Timers überlebt.
- **Spiel-Timer:** max. 60 min. Ein großer Timer ist das Zentrum des Spielbildschirms.

##### Dynamische Pings

Jeder Hunter bekommt Pings für jeden Hunted. Das Intervall hängt von der Distanz zwischen diesem Hunter und diesem Hunted ab:

|Distanz|Ping alle|
|---|---|
|> 500 m|2 min (Standard)|
|250–500 m|1 min|
|100–250 m|30 s|
|< 100 m|15 s|

Merkhilfe: **halbe Distanz → doppelt so schnelle Pings.** Es geht nie unter 15 s, damit der Hunted auch auf kurze Distanz noch entkommen kann.

##### Taggen (Lock-on)

GPS allein ist für einen echten Tag zu ungenau. Daher läuft das Taggen in zwei Schritten. Merkhilfe: **GPS sagt „gleiche Straße", Bluetooth sagt „Armlänge".**

1. **Tag-Button wird aktiv:** sobald ein Hunted laut GPS innerhalb von **30 m** um den Hunter ist. Ziel ist der nächstgelegene Hunted.
2. **Lock-on startet:** Der Hunter **hält** den Tag-Button gedrückt.
3. **Lock-on-Warnung:** Das Handy des Hunted vibriert sofort, spielt einen Warnton ab und zeigt „🔒 Jemand lockt sich auf dich ein!", der Hunter wird auf der Karte hervorgehoben. Das ist die letzte Chance zu flüchten.
4. **Bluetooth-Prüfung:** Während des gesamten Lock-ons müssen die Handys laut Bluetooth nah beieinander bleiben (≈ 3 m).
5. **Getaggt:** Nach **2 s** durchgehendem Lock-on bestätigt der Server den Tag.

**Der Lock-on bricht ab** (→ „Lock verloren" für beide Spieler), wenn:

- der Hunter den Button loslässt,
- die Handys sich zu weit voneinander entfernen,
- der Hunted eine Safe-Zone betritt oder Immunität aktiviert.

Nach einem abgebrochenen Lock-on wartet der Hunter **5 s**, bevor er sich erneut einlocken kann, damit der Button nicht gespammt werden kann.

**Kein Lock-on möglich** auf einen Hunted, der sich in einer Safe-Zone befindet oder immun ist.

**Anforderungen an Bluetooth**

- Während eines Spiels sendet jedes Handy einen **anonymen, regelmäßig wechselnden Code**. Nur der Server kann ihn einem Spieler zuordnen.
- Taggen muss zwischen allen Kombinationen von Android und iPhone funktionieren, auch wenn das Handy des Hunted gesperrt in der Hosentasche steckt.
- Die Nah-Distanz wird pro Gerätetyp kalibriert, auch für Handys in Hosentaschen und Taschen.

##### Zusatzmechaniken

- **Bewegliche Safe-Zonen:** Hunted können darin nicht getaggt werden.
- **Bewegliche No-Go-Zonen:** Ein Spieler darin bekommt einen **30-s**-Countdown, um sie zu verlassen. Ist er danach noch drin → disqualifiziert. Das gilt auch für Zonen, die sich über einen Spieler bewegen.
- **Schrumpfender Sturm:** treibt die Spieler zusammen und beendet Spiele. Sein Mittelpunkt wandert während des Spiels, damit kein fixer Ort sicher bleibt (siehe „Camping" in [Anti-Cheat](#anti-cheat)). Ein Spieler außerhalb des Sturmkreises bekommt denselben **30-s**-Countdown → disqualifiziert.
- **Zuschauermodus:** Der Zuschauer-Feed ist verzögert, damit Zuschauer den Spielern keine Positionen verraten können („Ghosting").
- **Power-ups**, max. eines pro Typ und Spiel:

|Power-up|Für|Wirkung|
|---|---|---|
|Immunität|Hunted|1 min nicht taggbar|
|Unsichtbarkeit|Hunter, Hunted|5 min versteckt|
|Extra-Reichweite|Hunter|1 min: Lock-on funktioniert aus größerer Distanz (60 m statt 30 m, Bluetooth ≈ 8 m statt 3 m)|
|Ghost-Ping|Hunted|Der Hunted setzt eine Markierung auf die Karte, max. 500 m von seiner echten Position entfernt. Jeder Hunter erhält **einen** Ping an dieser falschen Position, der genau wie ein echter Ping aussieht.|
|No-Go-Zone|Hunter|Der Hunter setzt eine No-Go-Zone auf die Karte, z. B. über ein Gebäude. Spieler darin bekommen den üblichen **30-s**-Countdown zum Verlassen. Zwingt Hunted aus ihren Häusern. Größe und Dauer noch offen.|

- **Onboarding:** Willkommensbildschirme, die das Spiel erklären.
- **Voice Lines** für wichtige Aktionen.

##### Fortschritt & Soziales

- Accounts mit **Levelsystem**.
- **Achievements**.
- **Freunde** per Freundescode oder QR-Code.
- **Coins**, verdient durch:
    - **zurückgelegte Distanz** (nicht überlebte Zeit, damit Campen zu Hause nichts bringt)
    - Taggen von Spielern
    - Abschließen eines Spiels
    - **Close Calls:** **10 s** lang innerhalb von **75 m** zu einem Spieler der Gegenrolle bleiben, ohne getaggt zu werden. Höchstens eine Belohnung pro Spielerpaar alle 2 min, damit es nicht gefarmt werden kann.
- **Shop** für Power-ups und Kosmetik:
    - Kosmetik für den Standortmarker
    - Kosmetik für Account-Lobby / Profil (wie bei Discord)
    - Rang-Abzeichen aus dem Elo-System
- **In-Game-Livechat**.

##### Elo

Standard-Elo, angepasst für Teams: Jeder Spieler wird mit dem **durchschnittlichen Elo des gegnerischen Teams** verglichen.

|Ergebnis|Wertung|
|---|---|
|Hunted, überlebt|Sieg|
|Hunted, getaggt|Niederlage|
|Hunter|Anteil der getaggten Hunted (z. B. 3 von 4 → 75 %)|
|Ranked-Spiel verlassen|Niederlage|

- In den ersten 20 Ranked-Spielen (Platzierung) ändert sich das Elo doppelt so schnell wie danach.
- Merkhilfe: **Sieg gegen Stärkere → großer Gewinn; Niederlage gegen Schwächere → großer Verlust.**
- **Rollenbalance:** Die Siegquoten pro Rolle werden beobachtet. Gewinnt eine Rolle deutlich öfter, wird die Wertung pro Rolle angepasst.

##### Monetarisierung

Alle Optionen sind **potenziell**. Das endgültige Modell ist noch offen.

|Option|Idee|Status|
|---|---|---|
|Coins kaufen|Echtgeld → Coins für den Shop|Potenziell|
|Battle Pass|Saisonale Belohnungsleiste mit Gratis- und Bezahlstufe (Kosmetik, Coins)|Potenziell|
|Pro-Abo|Monatliche Vorteile, z. B. exklusive Kosmetik, Bonus-Coins, erweiterte Einstellungen für private Spiele, Spielverlauf|Potenziell|
|Lootboxen|Zufällige Kosmetik|Potenziell|

- Rechtliche Einschränkungen zu Lootboxen: siehe [Rahmenbedingungen](#17-rahmenbedingungen).

##### Spielersicherheit

- Spieler, die gefährliche Bereiche betreten (Autobahnen, Gleise, Gewässer, …), bekommen sofort eine Warnung und einen **15-s**-Countdown; sind sie danach noch drin → disqualifiziert. Der Countdown verhindert Fehl-Disqualifikationen durch GPS-Ungenauigkeit neben einer Straße.
- Sicherheitshinweise (auf den Verkehr achten, auf dem Gehsteig bleiben, …).

##### Anti-Cheat

- **Nur zu Fuß:** Wer schneller als **25 km/h** unterwegs ist (Fahrrad, Scooter, Fahrzeug), wird gemeldet.
- **Tag-Prüfung:** Der Server prüft jeden Tag: GPS-Nähe, gültige Bluetooth-Nachweise für die vollen 2 s, und dass das Ziel weder immun noch in einer Safe-Zone war.
- **Manipulation:** Weitergegebene oder abgelaufene Bluetooth-Codes, unplausible Signalwerte und Handys, die absichtlich keine Bluetooth-Daten liefern, werden erkannt und gemeldet.
- Wiederholte Meldungen → Verwarnung, dann Bann.
- **Camping:** Sich das ganze Spiel zu Hause zu verstecken, soll keine brauchbare Strategie sein.
    - Coins gibt es für zurückgelegte Distanz, nicht für überlebte Zeit ([Fortschritt & Soziales](#fortschritt--soziales)).
    - Der Mittelpunkt des Sturms wandert, damit kein fixer Ort sicher bleibt ([Zusatzmechaniken](#zusatzmechaniken)).
    - Hunter haben ein **No-Go-Zonen**-Power-up, um Hunted aus einem Gebäude zu zwingen ([Zusatzmechaniken](#zusatzmechaniken)).
- Schutz der App gegen Manipulation.
- Banns und Chat-Moderation.

##### Hintergrund & Benachrichtigungen

Das Spiel muss bei ausgeschaltetem Bildschirm und mit dem Handy in der Hosentasche weiterlaufen.

- **Live-Spielbenachrichtigung** am Sperrbildschirm (Android) bzw. Live Activity mit Dynamic Island (iOS):
    - der Spiel-Timer als Countdown
    - eigene Rolle und verbleibende Spieler
    - Hunter: Countdown bis zum nächsten Ping; Hunted: „Hunter innerhalb von 250 m"
- **Lock-on-Warnung:** eigene, auffällige Benachrichtigung (Pop-up + Vibration), getrennt von der ruhigen Live-Benachrichtigung.
- **Push-Benachrichtigungen:** Spielstart, „ein Hunter ist in der Nähe", getaggt, Spielende, Freundeseinladungen.

##### Spielparameter

Alle Balance-Werte an einem Ort:

|Wert|Einstellung|Abschnitt|
|---|---|---|
|Tag-Button aktiv ab (GPS)|30 m|[Taggen (Lock-on)](#taggen-lock-on)|
|Bluetooth-Nah-Distanz|≈ 3 m, pro Gerätetyp kalibriert|[Taggen (Lock-on)](#taggen-lock-on)|
|Lock-on-Dauer|2 s|[Taggen (Lock-on)](#taggen-lock-on)|
|Lock-on-Cooldown nach Abbruch|5 s|[Taggen (Lock-on)](#taggen-lock-on)|
|Power-up Extra-Reichweite|60 m, Bluetooth ≈ 8 m|[Zusatzmechaniken](#zusatzmechaniken)|
|Ping-Intervalle|2 min / 1 min / 30 s / 15 s|[Dynamische Pings](#dynamische-pings)|
|Countdown No-Go-Zone / Sturm|30 s|[Zusatzmechaniken](#zusatzmechaniken)|
|Countdown gefährlicher Bereich|15 s|[Spielersicherheit](#spielersicherheit)|
|Geschwindigkeitslimit (zu Fuß)|25 km/h|[Anti-Cheat](#anti-cheat)|
|Max. Distanz Ghost-Ping|500 m|[Zusatzmechaniken](#zusatzmechaniken)|
|Close Call|75 m für 10 s, 2 min Cooldown pro Paar|[Fortschritt & Soziales](#fortschritt--soziales)|
|Elo-Platzierung|erste 20 Ranked-Spiele|[Elo](#elo)|
|Aufbewahrung von Daten|30 Tage (Tags, Cheat-Nachweise)|[Datenschutz (DSGVO)](#datenschutz-dsgvo)|

#### 1.4.2 Nichtfunktionale Anforderungen (NFA)

##### Fairness & Sicherheit

- **Der Server entscheidet.** Die App meldet nur Positionen und Aktionen; ob ein Tag zählt, wer gewinnt und wer disqualifiziert wird, entscheidet der Server.
- **Jeder bekommt nur, was er sehen darf.** Hunter erhalten nie Live-Positionen der Hunted, nur Pings. Eine manipulierte App kann nicht verraten, was sie nie erhalten hat.
- Accounts sind durch eine sichere Anmeldung geschützt.

##### Datenschutz (DSGVO)

Standortdaten sind personenbezogene Daten, daher wird **keine Standorthistorie gespeichert**. Nur was ein Feature tatsächlich braucht, wird aufbewahrt:

|Daten|Wozu|Aufbewahrung|
|---|---|---|
|Live-Positionen|Spielbetrieb|Gelöscht bei Spielende|
|Beide Positionen bei einem Tag|Nachweis, dass ein Tag gültig war|Koordinaten nach 30 Tagen entfernt|
|Bluetooth-Codes und -Meldungen|Prüfung der Nähe beim Taggen|Gelöscht bei Spielende|
|Nachweise zu einer Cheat-Meldung|Prüfung von Meldungen, Einsprüche gegen Banns|30 Tage oder bis der Einspruch erledigt ist|
|Gelaufene Distanz pro Spiel|Statistiken und Achievements|Eine einzelne Zahl, keine Koordinaten|
|Chatnachrichten|In-Game-Chat|Gelöscht bei Spielende|
|Gemeldete Chatnachrichten|Moderation|Bis die Meldung erledigt ist|

- **Bluetooth ist anonym:** Andere können höchstens erkennen, dass **jemand in der Nähe taggd spielt**, aber nicht wer.
- Bluetooth ist **nur während eines aktiven Spiels** aktiv, es gibt kein Koppeln der Handys, und die Berechtigungsdialoge erklären, dass Bluetooth nur zum Taggen verwendet wird.

##### Akku

- Die Karte wird nur bei eingeschaltetem Bildschirm gezeichnet.
- Benachrichtigungen statt dauerhaft offener App.

##### Skalierbarkeit

- Das System muss mit steigender Spielerzahl wachsen können, indem zusätzliche Server gestartet werden. Für die Entwicklung und das erste Release reicht ein Server.

##### Benutzbarkeit

- Verspielter Stil, angelehnt an das Design von Duolingo.
- Onboarding und Voice Lines (siehe [Zusatzmechaniken](#zusatzmechaniken)).
- Bots sind immer klar als Bots gekennzeichnet.

##### Qualitätssicherung

- **Simulierte Spieler:** Fake-Spieler laufen vorgegebene Routen ab. Die Bots in Public-Spielen verwenden denselben Mechanismus.
- **Automatisierte Tests:** Unit-Tests, Integrationstests und komplette Spiele mit simulierten Spielern, die Tags, Pings, Sturm und Wertung prüfen.
- **Lasttests** mit vielen simulierten Spielern.
- **Bluetooth-Feldtests** mit verschiedenen Handys, in der Hand, Hosentasche und Tasche, um die Nah-Distanz zu kalibrieren.

### 1.5 Ziele

- **Förderung von Bewegung und Gesundheit:** Jugendliche und junge Erwachsene werden durch spielerische Motivation dazu bewegt, sich aktiv zu Fuß im Freien aufzuhalten.
- **Stärkung sozialer Interaktionen:** Gemeinsame Erlebnisse und Teamgeist; man kann mit Freunden spielen und neue Personen kennenlernen.
- **Faire, überprüfbare Fänge:** Ein Tag zählt nur, wenn GPS und Bluetooth die Nähe bestätigen und der Server ihn prüft.
- **Sicheres Spielen:** Gefährliche Bereiche werden erkannt, Spieler gewarnt und bei Missachtung disqualifiziert.
- **Datensparsamkeit:** Keine Standorthistorie; Live-Daten werden bei Spielende gelöscht.

### 1.6 Mengengerüst

|Größe|Wert|
|---|---|
|Spieldauer|max. 60 min|
|Hunter-Anteil|30 % der Spieler (Standard)|
|Beitritt zu laufenden Public-Spielen|nur in den ersten 15 min|
|Matchmaking-Umkreis|5 km|
|Standortübertragung pro Spieler|alle 3–10 s|
|Aufbewahrung Tag-Koordinaten / Cheat-Nachweise|30 Tage|
|Spieler pro Spiel, gleichzeitige Spiele|noch offen|

### 1.7 Rahmenbedingungen

**Organisatorisch**

- Entwicklerteam: Erik, Almir, Lukas und Julian.
- Das Projekt muss in 1,5 Jahren fertiggestellt werden.
- Während der Entwicklung fallen keine Kosten an: bestehende Hardware, ausschließlich freie Open-Source-Tools und freie Geodaten.

**Technisch**

- **Zielplattformen:** Android und iOS; Release im Google Play Store und App Store optional.
- **Hardware:** GPS für die grobe Ortung, Bluetooth für das Taggen.
- **Kartendaten** stammen aus **OpenStreetMap**; die Karte muss den Hinweis „© OpenStreetMap contributors" zeigen. Die genutzten freien Kartendienste haben Fair-Use-Grenzen und keine Verfügbarkeitsgarantie.
- Technische Details zur Umsetzung: siehe [Projektspezifikation](project-specification.md#10-architecture).

**Rechtlich**

- **DSGVO:** Standortdaten dürfen nur temporär für die Dauer eines Spiels verarbeitet werden (siehe [Datenschutz (DSGVO)](#datenschutz-dsgvo)). Eine Datenschutzerklärung und eine In-App-Einwilligung für Standort und Bluetooth sind nötig und werden vor dem öffentlichen Release geprüft.
- **Mindestalter:** 14 Jahre (digitales Einwilligungsalter in Österreich laut DSGVO); jüngere Spieler brauchen die Zustimmung der Eltern.
- **Lootboxen:** Bezahlte Lootboxen sind in Belgien verboten und wurden in österreichischen Gerichtsverfahren als Glücksspiel eingestuft. Vor einer Umsetzung ist rechtliche Beratung einzuholen. Falls verwendet: nur Kosmetik, mit angezeigten Drop-Raten.
- **App Stores:** Die dauerhafte Standortberechtigung wird im App Store streng geprüft.
- **Kosten bei Veröffentlichung:** Google Play Console (ca. 25 € einmalig), Apple Developer Program (ca. 99 €/Jahr).
