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

**Stand des Projekts: Bluetooth-Proof-of-Concept**

Eine kleine App testet den Bluetooth-Teil des Taggens, bevor die eigentliche App gebaut wird.

|Teil|Umgesetzt mit|
|---|---|
|App|Kotlin Multiplatform, gemeinsame Compose-UI und Ktor-Client; Android-Target|
|Server|Flask (Python) als Ersatz für das Spring-Boot-Backend: vergibt rotierende Codes (30 s), löst empfangene Codes zu Spielern auf|

**Funktion:** Umschalter Hunter / Hunted; das Handy sendet seinen Code und scannt nach anderen taggd-Handys. Erkennt es eines, vibriert es und zeigt Debug-Infos: Code, Signalstärke, ungefähre Distanz, ob der Code aus der Scan Response oder über GATT kam, und zu welchem Spieler und welcher Rolle der Server ihn aufgelöst hat.

**Bisherige Ergebnisse**

- Erkennung und Auflösung durch den Server funktionieren zwischen einem **Pixel 7 Pro** (Android 17) und einem **vivo V2109** (Android 12). Die App ist auch auf einem **Galaxy A17** und einem **Galaxy A37** (Android 16) installiert.
- Erkenntnis: Android kappt den Netzwerkzugriff minimierter Apps, daher ist ein **Foreground Service** nötig ([[#Hintergrundbetrieb & Benachrichtigungen]]). Damit funktioniert die App auch im Hintergrund.
- Das gemeinsame Schema aus [[#Taggen (Lock-on)]] (fixe UUID, Scan Response, GATT) ist auf Android und am Server umgesetzt, aber **noch nicht im Feld getestet**.

**Noch offen:** die iOS-Version (benötigt Xcode auf einem Mac), die Kalibrierung der Signalstärke und der Lock-on selbst.

### 1.3 Problemstellung

- **GPS ist für einen echten Fang zu ungenau:** zwei Handys können jeweils 5–10 m danebenliegen. Ein Fang allein über GPS wäre unfair.
- **Kein neutraler Schiedsrichter:** Ohne Server kann niemand prüfen, ob ein Fang gültig war, ob jemand mit Fahrrad, Scooter oder Auto unterwegs ist oder ob eine App manipuliert wurde.
- **Sicherheit im öffentlichen Raum:** Im Eifer des Spiels achten Spieler weniger auf den Verkehr. Gefährliche Bereiche (Autobahnen, Gleise, Gewässer) müssen erkannt werden.
- **Camping:** Wer sich das ganze Spiel zu Hause versteckt, darf damit keinen Vorteil haben.
- **Datenschutz:** Standortdaten sind personenbezogene Daten und dürfen nicht dauerhaft gespeichert werden.
- **Plattformgrenzen:** iOS schränkt Bluetooth stark ein, besonders bei gesperrtem iPhone; Android beendet den Netzwerkzugriff minimierter Apps.

### 1.4 Aufgabenstellung

Es wird eine Mobile-App (Android und iOS) samt Backend entwickelt, mit der Spieler in zwei Rollen gegeneinander antreten:

|Rolle|Sieht|Ziel|
|---|---|---|
|**Hunter**|Hunted-Spieler als **Pings** (Standort-Momentaufnahmen), standardmäßig alle 2 min, schneller bei Annäherung (siehe [[#Dynamische Pings]])|Alle Hunted-Spieler taggen|
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
- Der Elo-Gewinn ist relational, wie beim Coin-System: Siege gegen stärkere Spieler bringen mehr (siehe [[#Elo]]).

##### Kernmechaniken

- **Hunter-Anteil:** standardmäßig sind 30 % der Spieler Hunter.
- **Taggen:** Der Hunter nähert sich an und **lockt sich ein** (Lock-on), indem er den Tag-Button gedrückt hält. Der Tag zählt nur, wenn **Bluetooth** während des gesamten Lock-ons bestätigt, dass die beiden Handys nur wenige Meter voneinander entfernt sind. Der Hunted bekommt eine **Lock-on-Warnung**. Details: [[#Taggen (Lock-on)]].
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

##### Standort-Updaterate

Das ist, wie oft Handys ihre Position **senden**. Es ist unabhängig davon, wie oft andere sie **sehen**.

|Spieler|Sendet alle|Für andere sichtbar|
|---|---|---|
|Hunter|3 s|Hunted sehen Hunter live|
|Hunted, kein Hunter innerhalb von 250 m|10 s|Hunter sehen nur Pings ([[#Dynamische Pings]])|
|Hunted, ein Hunter innerhalb von 250 m|3 s|Hunter sehen nur Pings ([[#Dynamische Pings]])|

- Der Server teilt jedem Hunted-Client mit, welche Rate er verwenden soll.
- **Distanzfilter:** Ein Update wird übersprungen, wenn sich der Spieler weniger als 5 m bewegt hat, aber mindestens alle 30 s als Heartbeat gesendet.
- **Frische Position beim Lock-on:** Startet ein Lock-on, fordert der Server vom Handy des Hunted einen neuen GPS-Fix an und wartet bis zu 3 s. Kommt keiner, verwendet er die letzte Position, aber nur, wenn sie jünger als 10 s ist.

##### Taggen (Lock-on)

GPS allein ist für einen echten Tag zu ungenau: zwei Handys können jeweils 5–10 m danebenliegen. Daher läuft das Taggen in zwei Schritten. Merkhilfe: **GPS sagt „gleiche Straße", Bluetooth sagt „Armlänge".**

**Ablauf**

1. **Vorfilter (GPS):** Der Tag-Button wird aktiv, sobald der Server einen Hunted innerhalb von **30 m** um den Hunter sieht. Ziel ist der nächstgelegene Hunted. Beide Handys werden informiert (`lock_available`), damit sich ein gesperrtes iPhone vorab mit dem Handy des Hunters verbinden kann (siehe Bluetooth-Design unten).
2. **Lock-on startet:** Der Hunter **hält** den Tag-Button gedrückt.
3. **Lock-on-Warnung:** Das Handy des Hunted vibriert sofort, spielt einen Warnton ab und zeigt „🔒 Jemand lockt sich auf dich ein!", der Hunter wird auf der Karte hervorgehoben. Das ist die letzte Chance zu flüchten.
4. **Bluetooth-Prüfung:** Während des gesamten Lock-ons melden die Handys die Bluetooth-Signalstärke an den Server. Sie muss über der „Nah"-Schwelle (≈ 3 m) bleiben. Eine Meldung von **einem der beiden** Handys reicht (siehe „Wer hört wen" unten).
5. **Getaggt:** Nach **2 s** durchgehendem Lock-on prüft der Server und bestätigt den Tag.

**Der Lock-on bricht ab** (→ „Lock verloren" für beide Spieler), wenn:

- der Hunter den Button loslässt,
- das Bluetooth-Signal unter die Schwelle fällt,
- der Hunted eine Safe-Zone betritt oder Immunität aktiviert.

Nach einem abgebrochenen Lock-on wartet der Hunter **5 s**, bevor er sich erneut einlocken kann, damit der Button nicht gespammt werden kann.

**Kein Lock-on möglich** auf einen Hunted, der sich in einer Safe-Zone befindet oder immun ist.

**Bluetooth-Design**

- Der Server gibt jedem Spieler einen **zufälligen 8-Byte-Code, der sich alle 30 s ändert**. Handys stellen ihren aktuellen Code über Bluetooth bereit und sammeln die Codes von Spielern in der Nähe.
- Nur der Server weiß, zu welchem Spieler ein Code gehört. Jeder Code ist nur für das aktuelle Spiel und das aktuelle sowie das vorherige 30-s-Fenster gültig.
- **Ein Schema für Android und iOS.** Jedes Handy macht das ganze Spiel über dieselben drei Dinge:

|Teil|Inhalt|Gesendet von|
|---|---|---|
|**Advert**|Fixe taggd-Service-UUID (für alle Spieler gleich), verbindbar|Android + iOS|
|**Scan Response**|Der aktuelle Code (als Service Data)|Nur Android: iOS-Apps können keine Service Data senden|
|**GATT-Service**|Ein **schreibgeschützter** Wert: der aktuelle Code|Android + iOS|

  *GATT* ist der Teil von Bluetooth Low Energy für kurze Verbindungen: ein Handy verbindet sich mit einem anderen und liest einen Wert aus. Es ist trotzdem Bluetooth, nur eine Verbindung statt bloßem Mithören von Broadcasts.

- **Warum eine fixe UUID:** iOS schränkt ein, was Apps über Bluetooth tun können:
    - eine iOS-App kann nur einen Namen und Service-UUIDs senden, keine anderen Daten;
    - ein gesperrtes iPhone sendet nur innerhalb von Apples verstecktem „Overflow-Bereich", der keinen Code enthält, sodass Android-Handys nicht erkennen können, wessen Handy es ist;
    - ein gesperrtes iPhone kann nur nach **exakten, bekannten** UUIDs scannen.

  Daher muss die UUID fix sein, und der Code wird in der Scan Response oder über eine kurze GATT-Verbindung übertragen.
- **Zuhören:** Jedes Handy scannt nach der fixen UUID.
    - Code in der Scan Response → direkt verwenden (keine Verbindung).
    - Kein Code (das andere Handy ist ein iPhone) → verbinden, Code lesen, trennen. Der Code wird pro Gerät 10 s wiederverwendet; die Signalstärke kommt aus den folgenden Scans.
    - Ein **gesperrtes iPhone** bekommt im Hintergrund zu wenige Scan-Ergebnisse, daher hält es während eines Lock-ons die Verbindung zum Handy des Hunters offen und liest die Signalstärke darüber.
- **Keine Modi:** Kein Handy muss die Plattform des anderen kennen. Beide Richtungen laufen das ganze Spiel parallel; der Server akzeptiert eine Meldung von jedem der beiden Handys.

**Wer hört wen** (die App des Hunters ist immer offen, da der Hunter den Tag-Button hält):

|Hunter|Hunted|Wer hört wen|
|---|---|---|
|Android|Android|Beide Richtungen, Code aus der Scan Response|
|Android|iPhone, App offen|Beide Richtungen; das Android-Handy liest den Code des iPhones über GATT|
|iPhone|Android|Beide Richtungen; das Android-Handy liest den Code des iPhones über GATT|
|Android oder iPhone|iPhone, **gesperrt**|**Nur das iPhone des Hunted** hört den Hunter und liest den Code des Hunters über GATT|

- **Timing:** Ein gesperrtes iPhone kann mehrere Sekunden brauchen, um ein Handy zu entdecken, länger als der 2-s-Lock-on. Die Handys scannen während des ganzen Spiels und kennen sich daher meist schon vor dem Lock-on; bei `lock_available` baut ein gesperrtes iPhone zusätzlich die Verbindung zu diesem Hunter auf, damit sie beim Start des Lock-ons bereitsteht.
- Die „Nah"-Schwelle wird pro Gerätetyp kalibriert, auch für Handys in Hosentaschen und Taschen.

##### Zusatzmechaniken

- **Bewegliche Safe-Zonen:** Hunted können darin nicht getaggt werden.
- **Bewegliche No-Go-Zonen:** Ein Spieler darin bekommt einen **30-s**-Countdown, um sie zu verlassen. Ist er danach noch drin → disqualifiziert. Das gilt auch für Zonen, die sich über einen Spieler bewegen.
- **Schrumpfender Sturm:** treibt die Spieler zusammen und beendet Spiele. Sein Mittelpunkt wandert während des Spiels, damit kein fixer Ort sicher bleibt (siehe „Camping" in [[#Anti-Cheat]]). Ein Spieler außerhalb des Sturmkreises bekommt denselben **30-s**-Countdown → disqualifiziert.
- **Zuschauermodus:** Der Zuschauer-Feed ist verzögert, damit Zuschauer den Spielern keine Positionen verraten können („Ghosting").
- **Power-ups**, max. eines pro Typ und Spiel:

|Power-up|Für|Wirkung|
|---|---|---|
|Immunität|Hunted|1 min nicht taggbar|
|Unsichtbarkeit|Hunter, Hunted|5 min versteckt|
|Extra-Reichweite|Hunter|1 min: Lock-on funktioniert aus größerer Distanz (Vorfilter 60 m, Bluetooth-Schwelle ≈ 8 m)|
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

Standard-Elo, angepasst für Teams. Jeder Spieler wird mit dem **durchschnittlichen Elo des gegnerischen Teams** verglichen:

```
expected = 1 / (1 + 10^((opponentAvg - playerElo) / 400))
change   = K × (score - expected)
```

| |score|
|---|---|
|Hunted, überlebt|1|
|Hunted, getaggt|0|
|Hunter|Anteil der getaggten Hunted (z. B. 3 von 4 → 0,75)|
|Ranked-Spiel verlassen|0 (zählt als Niederlage)|

- **K = 32** für die ersten 20 Ranked-Spiele eines Spielers (Platzierung, ändert sich schnell), danach **K = 16**.
- Merkhilfe: **Sieg gegen Stärkere → großer Gewinn; Niederlage gegen Schwächere → großer Verlust.**
- **Rollenbalance:** Die Siegquoten pro Rolle werden beobachtet. Gewinnt eine Rolle deutlich öfter, werden die Scores oder K pro Rolle angepasst.

##### Monetarisierung

Alle Optionen sind **potenziell**. Das endgültige Modell ist noch offen.

|Option|Idee|Status|
|---|---|---|
|Coins kaufen|Echtgeld → Coins für den Shop|Potenziell|
|Battle Pass|Saisonale Belohnungsleiste mit Gratis- und Bezahlstufe (Kosmetik, Coins)|Potenziell|
|Pro-Abo|Monatliche Vorteile, z. B. exklusive Kosmetik, Bonus-Coins, erweiterte Einstellungen für private Spiele, Spielverlauf|Potenziell|
|Lootboxen|Zufällige Kosmetik|Potenziell|

- Rechtliche Einschränkungen zu Lootboxen: siehe [[#1.7 Rahmenbedingungen]].

##### Spielersicherheit

- Spieler, die gesperrte oder gefährliche Bereiche betreten (Autobahnen, Gleise, Gewässer, …), bekommen sofort eine Warnung und einen **15-s**-Countdown; sind sie danach noch drin → disqualifiziert. Der Countdown verhindert Fehl-Disqualifikationen durch GPS-Ungenauigkeit neben einer Straße. Datenquelle: OSM, siehe [[#1.7 Rahmenbedingungen]].
- Sicherheitshinweise (auf den Verkehr achten, auf dem Gehsteig bleiben, …).

##### Anti-Cheat

- **Nur zu Fuß:** Geschwindigkeitsprüfungen (Distanz ÷ Zeit zwischen Updates) melden alles über **25 km/h**, was Fahrräder, Scooter und Fahrzeuge ausschließt. Updates mit schlechter GPS-Genauigkeit werden ignoriert.
- **Tag-Validierung:** Der Server prüft den GPS-Vorfilter, ob die Bluetooth-Codes für dieses Spiel, diesen Spieler und dieses Zeitfenster gültig sind, ob das Signal die vollen 2 s über der Schwelle blieb (gemeldet von einem der beiden Handys) und ob das Ziel nicht immun oder in einer Safe-Zone war.
- **Code-Replay:** Ein Code, der abgelaufen ist oder zu einem anderen Spiel oder Spieler gehört → abgelehnt und gemeldet.
- **Code-Relay** (ein Komplize neben dem Opfer leitet dessen Code an einen weit entfernten Hunter weiter): Codes rotieren alle 30 s, der Besitzer des Codes muss sich innerhalb des GPS-Vorfilters des meldenden Spielers befinden, und das Signal muss die vollen 2 s plausibel bleiben.
- **Abgleich der Signalstärke:** Handys melden die Signalstärke selbst, daher könnte eine manipulierte Hunter-App ein stärkeres Signal melden als tatsächlich gemessen (z. B. das Opfer aus 30 m hören und Armlänge behaupten). Das Signal zwischen zwei Handys ist in beide Richtungen ungefähr gleich, daher vergleicht der Server die Werte, wenn beide Handys melden: Ein Hunter, dessen Werte durchgehend deutlich stärker sind als die des Hunted → **gemeldet**. Bei einem gesperrten iPhone meldet nur der Hunted, und der Hunted hat keinen Grund, das Signal zu übertreiben.
- **Bluetooth-Heartbeat:** Während eines Spiels sendet jedes Handy alle **5 s** einen `ble_report` mit allem, was es gehört hat. Das schließt die Lücke beim gesperrten iPhone, bei dem **nur das Handy des Hunted** den Bluetooth-Nachweis liefert (siehe [[#Taggen (Lock-on)]]): Eine manipulierte App könnte sonst einfach schweigen und wäre nicht taggbar.
    - Ein Hunted, dessen Handy nichts meldet, während laut GPS ein Hunter neben ihm steht → **gemeldet**.
    - Zusätzlicher Nachweis: Das Android-Handy des Hunters kann den versteckten „Overflow-Bereich"-Broadcast des iPhones sehen. Er verrät nicht wer, beweist aber, dass ein taggd-iPhone in der Nähe ist. Das beruht auf einem undokumentierten Apple-Format, verstärkt daher nur eine Meldung und entscheidet nie allein über einen Tag.
    - Wiederholte Meldungen → Verwarnung, dann der normale Bann-Prozess.
- **Camping:** Sich das ganze Spiel zu Hause zu verstecken, soll keine brauchbare Strategie sein.
    - Coins gibt es für zurückgelegte Distanz, nicht für überlebte Zeit ([[#Fortschritt & Soziales]]).
    - Der Mittelpunkt des Sturms wandert, damit kein fixer Ort sicher bleibt ([[#Zusatzmechaniken]]).
    - Hunter haben ein **No-Go-Zonen**-Power-up, um Hunted aus einem Gebäude zu zwingen ([[#Zusatzmechaniken]]).
- Anti-Tampering für das Frontend.
- Banns und Chat-Moderation.

##### Hintergrundbetrieb & Benachrichtigungen

Das Spiel muss bei ausgeschaltetem Bildschirm und mit dem Handy in der Hosentasche weiterlaufen.

- **Android:** Das Spiel läuft in einem **Foreground Service** (Typen `location` + `connectedDevice`). Ohne ihn kappt Android den Netzwerkzugriff minimierter Apps: Im Proof of Concept liefen Serveranfragen in einen Timeout, sobald die App minimiert wurde ([[#1.2 Istzustand]]).
- **iOS:** Hintergrundmodi `location` (hält die App am Laufen, siehe [[#Akku]]), `bluetooth-central` und `bluetooth-peripheral`.
- Android verlangt für jeden Foreground Service eine **sichtbare Benachrichtigung**. Statt sie zu verstecken, macht taggd daraus eine **Live-Spielbenachrichtigung**:
    - der Spiel-Timer als Countdown (läuft im System, keine Updates nötig)
    - eigene Rolle und verbleibende Spieler
    - Hunter: Countdown bis zum nächsten Ping; Hunted: „Hunter innerhalb von 250 m"
    - aktualisiert bei Spielereignissen, nicht jede Sekunde
- **Android 16+:** angefordert als **Live Update** (hervorgehobene Benachrichtigung am Sperrbildschirm, Statusleisten-Chip mit Timer oder 🔒). Google erlaubt das nur für laufende, zeitkritische Aktivitäten, es hängt also davon ab, ob Google eine Spielrunde akzeptiert; sonst bleibt es eine normale laufende Benachrichtigung.
- **iOS:** eine **Live Activity** (Sperrbildschirm + Dynamic Island). Updates kommen aus der App oder per APNs-Push.
- **Lock-on-Warnung:** eigener Benachrichtigungskanal mit hoher Priorität (Pop-up + Vibration), getrennt von der ruhigen Live-Benachrichtigung.
- **Push-Benachrichtigungen** (FCM / APNs): Spielstart, „ein Hunter ist in der Nähe", getaggt, Spielende, Freundeseinladungen.

##### Spielparameter

Alle Balance-Werte an einem Ort:

|Wert|Einstellung|Abschnitt|
|---|---|---|
|Lock-on-Vorfilter (GPS)|30 m|[[#Taggen (Lock-on)]]|
|Bluetooth-„Nah"-Schwelle|≈ 3 m, pro Gerätetyp kalibriert|[[#Taggen (Lock-on)]]|
|Lock-on-Dauer|2 s|[[#Taggen (Lock-on)]]|
|Lock-on-Cooldown nach Abbruch|5 s|[[#Taggen (Lock-on)]]|
|Rotation des Bluetooth-Codes|30 s|[[#Taggen (Lock-on)]]|
|Power-up Extra-Reichweite|Vorfilter 60 m, Bluetooth ≈ 8 m|[[#Zusatzmechaniken]]|
|Ping-Intervalle|2 min / 1 min / 30 s / 15 s|[[#Dynamische Pings]]|
|Standort-Senderate|Hunter 3 s, Hunted 10 s (3 s in der Nähe von Huntern)|[[#Standort-Updaterate]]|
|Countdown No-Go-Zone / Sturm|30 s|[[#Zusatzmechaniken]]|
|Countdown gefährlicher Bereich|15 s|[[#Spielersicherheit]]|
|Geschwindigkeitslimit (zu Fuß)|25 km/h|[[#Anti-Cheat]]|
|Max. Distanz Ghost-Ping|500 m|[[#Zusatzmechaniken]]|
|Close Call|75 m für 10 s, 2 min Cooldown pro Paar|[[#Fortschritt & Soziales]]|
|Elo-K-Faktor|32 (erste 20 Spiele) → 16|[[#Elo]]|
|Aufbewahrung von Daten|30 Tage (Tags, Cheat-Nachweise)|[[#Datenschutz (DSGVO)]]|

#### 1.4.2 Nichtfunktionale Anforderungen (NFA)

##### Sicherheit

- **Der Server ist die Autorität.** Clients senden nur Positionen und Aktionen; alles andere entscheidet der Server.
- **Der Server filtert, was jeder Client erhält.** Hunter bekommen nie Live-Positionen der Hunted, nur Pings. Eine manipulierte App kann nicht verraten, was sie nie erhalten hat.
- Authentifizierung mit JWT.

##### Bluetooth-Sicherheit

- **Geändert gegenüber dem ersten Entwurf:** Dort hieß es, Handys „verbinden oder koppeln sich nie". Das wurde verworfen, weil ein gesperrtes iPhone einen Code nur über eine Verbindung bekommen kann (siehe [[#Taggen (Lock-on)]]). Verbindungen sind erlaubt, aber streng begrenzt:
    - der GATT-Service bietet genau **einen schreibgeschützten Wert** (den aktuellen Code); Schreibzugriffe und alle anderen Anfragen werden abgelehnt;
    - **kein Pairing, kein Bonding**;
    - der Leser trennt die Verbindung direkt nach dem Lesen; nur ein gesperrtes iPhone in einem aktiven Lock-on hält die Verbindung offen, um die Signalstärke zu messen;
    - Verbindungsversuche sind **pro Gerät rate-limitiert**, damit niemand den Akku leeren oder Verbindungen blockieren kann.
- Das Lesen des Codes über GATT verrät nichts Neues: Derselbe Code wird ohnehin offen gesendet.
- Der Broadcast enthält **nur** die fixe taggd-UUID und den zufälligen, rotierenden Code: keinen Namen, keine Account-ID. Niemand außer dem Server kann ihn einer Person zuordnen.
- Bluetooth-Broadcast, Scan und GATT-Service laufen **nur während eines aktiven Spiels** und stoppen danach automatisch.
- Berechtigungen:
    - **Android:** „Geräte in der Nähe" (Scan, Advertise, Connect), deklariert als **nicht** für Standort verwendet; Benachrichtigungen (siehe [[#Hintergrundbetrieb & Benachrichtigungen]]).
    - **iOS:** Bluetooth.
    - Die Berechtigungsdialoge erklären, dass Bluetooth nur zum Taggen verwendet wird.

##### Datenschutz (DSGVO)

Standortdaten sind personenbezogene Daten, daher wird **keine Standorthistorie gespeichert**. Nur was ein Feature tatsächlich braucht, wird aufbewahrt:

|Daten|Wozu|Aufbewahrung|
|---|---|---|
|Live-Positionen (Redis)|Spielbetrieb|Gelöscht bei Spielende|
|Beide Positionen bei einem Tag (`tags`)|Nachweis, dass ein Tag gültig war|Koordinaten nach 30 Tagen entfernt|
|Bluetooth-Codes (Redis)|Nähe-Prüfung beim Taggen|Zufällig, rotieren alle 30 s, gelöscht bei Spielende|
|Letzter Bluetooth-Report pro Spieler (Redis)|Tag-Validierung, Bluetooth-Heartbeat-Prüfung|Nur der letzte Report, gelöscht bei Spielende|
|Nachweise zu einer Cheat-Meldung (`anticheat_flags`)|Prüfung von Meldungen, Einsprüche gegen Banns|30 Tage oder bis der Einspruch erledigt ist|
|Gelaufene Distanz pro Spiel (`game_players`)|Statistiken und Achievements|Eine einzelne Zahl, keine Koordinaten|
|Chatnachrichten (Redis)|In-Game-Chat|Gelöscht bei Spielende|
|Gemeldete Chatnachrichten (`chat_reports`)|Moderation|Bis die Meldung erledigt ist|

- **Fixe Bluetooth-UUID:** Jeder mit einem Bluetooth-Scanner kann erkennen, dass **jemand in der Nähe taggd spielt**, aber nicht wer: Die Bluetooth-Adressen der Handys ändern sich regelmäßig und nur der Server kann den Code auflösen. Sie wird nur während eines aktiven Spiels gesendet.

##### Akku

- Auf Push-Benachrichtigungen setzen; Kartendaten nur rendern, wenn der Bildschirm an ist.
- Standort-Updates werden auch bei ausgeschaltetem Bildschirm gesendet, da der Server sie braucht. Auf iOS erfordert das die Standortberechtigung „Immer"; auf Android läuft das Spiel in einem Foreground Service (siehe [[#Hintergrundbetrieb & Benachrichtigungen]]).

##### Skalierbarkeit

- Das Backend läuft als **N identische, zustandslose Instanzen** hinter einem Load Balancer. Live-Zustand liegt in Redis, permanente Daten in PostgreSQL.
- Skalieren heißt, mehr Kopien desselben Docker-Images zu starten (`docker compose up --scale app=3`). Für die Entwicklung und das erste Release reicht eine Instanz.
- Die Spielschleife eines Spiels (Sturm-Ticks, Ping-Timer) läuft auf genau einer Instanz. Fällt diese aus, übernimmt eine andere das Spiel.

##### Benutzbarkeit

- Verspielter Stil, angelehnt an das Design von Duolingo.
- Onboarding und Voice Lines (siehe [[#Zusatzmechaniken]]).
- Bots sind immer klar als Bots gekennzeichnet.

##### Qualitätssicherung

- **Standortsimulator:** spielt GPX-Tracks oder geskriptete Routen als Fake-Spieler ab. Die Bots in Public-Spielen verwenden denselben Code.
- **Unit-Tests** wie üblich.
- **Automatisierte Spieltests:** komplette Spiele mit simulierten Spielern, die Tags, Pings, Sturm und Wertung prüfen.
- **Integrationstests** mit Testcontainers (echtes PostGIS + Redis).
- **Lasttests** mit vielen simulierten Spielern, um den Durchsatz von WebSocket und Redis zu prüfen.
- **Bluetooth-Feldtests:**
    - Signalstärke vs. Distanz (1 / 3 / 5 / 8 m; in der Hand, in der Hosentasche, in einer Tasche) pro Gerätemodell, um die „Nah"-Schwelle zu kalibrieren;
    - Hintergrund-Matrix: Android / iPhone × App offen / gesperrt, gemessen wird die Zeit bis zur ersten Erkennung.

### 1.5 Ziele

- **Förderung von Bewegung und Gesundheit:** Jugendliche und junge Erwachsene werden durch spielerische Motivation dazu bewegt, sich aktiv zu Fuß im Freien aufzuhalten.
- **Stärkung sozialer Interaktionen:** Gemeinsame Erlebnisse und Teamgeist; man kann mit Freunden spielen und neue Personen kennenlernen.
- **Faire, überprüfbare Fänge:** Ein Tag zählt nur, wenn GPS und Bluetooth die Nähe bestätigen und der Server ihn validiert.
- **Sicheres Spielen:** Gefährliche Bereiche werden erkannt, Spieler gewarnt und bei Missachtung disqualifiziert.
- **Datensparsamkeit:** Keine Standorthistorie; Live-Daten werden bei Spielende gelöscht.

### 1.6 Mengengerüst

|Größe|Wert|
|---|---|
|Spieldauer|max. 60 min|
|Hunter-Anteil|30 % der Spieler (Standard)|
|Beitritt zu laufenden Public-Spielen|nur in den ersten 15 min|
|Standort-Updates pro Hunter|alle 3 s → bis zu 1.200 pro Spiel|
|Standort-Updates pro Hunted|alle 10 s bzw. 3 s → 360 bis 1.200 pro Spiel|
|Mindestrate (Heartbeat bei Stillstand)|alle 30 s → mind. 120 pro Spieler und Spiel|
|`ble_report` pro Spieler|alle 5 s → 720 pro Spiel, zusätzlich laufend während eines Lock-ons|
|Bluetooth-Codes pro Spieler|8 Byte, Rotation alle 30 s → 120 pro Spiel|
|Chatverlauf pro Spiel|die letzten 100 Nachrichten|
|Matchmaking-Umkreis|5.000 m um den Spielmittelpunkt|
|Overpass-API (öffentlich)|Fair-Use: ~10.000 Anfragen und ~1 GB pro Tag; Abfrage nur für neue Gebiete, wöchentliche Aktualisierung|
|Aufbewahrung Tag-Koordinaten / Cheat-Nachweise|30 Tage|
|Spieler pro Spiel, gleichzeitige Spiele|noch offen|

### 1.7 Rahmenbedingungen

**Organisatorisch**

- Entwicklerteam: Erik, Almir, Lukas und Julian.
- Das Projekt muss in 1,5 Jahren fertiggestellt werden.
- Während der Entwicklung fallen keine Kosten an: bestehende Hardware, ausschließlich freie Open-Source-Tools und freie Geodaten.

**Technisch**

- **Zielplattformen:** Android und iOS; Release im Google Play Store und App Store optional.
- **Hardware:** GPS für die grobe Ortung, Bluetooth Low Energy für das Taggen. Bluetooth muss zum Spielen eingeschaltet sein.
- **Technologien:**
    - **Frontend:** Kotlin Multiplatform + Compose (MVVM), Karte mit MapLibre.
    - **Backend:** Spring Boot (Kotlin) als modularer Monolith; REST für normale Daten, WebSocket für Live-Spiele.
    - **Daten:** Redis (Live-Zustand), PostgreSQL + PostGIS (permanente Daten).
    - **Betrieb:** Docker Image; Docker Compose mit App, `postgis/postgis`, Redis und nginx oder Traefik.
    - **CI** (z. B. GitHub Actions): Tests, Docker-Image-Build, App-Builds.
    - Details zur Architektur gehören in den Entwurf „Wie mache ich es" und sind nicht Teil dieses Pflichtenhefts.
- **Plattformeinschränkungen:**
    - iOS-Apps können über Bluetooth nur Namen und Service-UUIDs senden; gesperrte iPhones scannen nur nach bekannten UUIDs (siehe [[#Taggen (Lock-on)]]).
    - Android kappt den Netzwerkzugriff minimierter Apps → Foreground Service nötig.
    - Die iOS Live Activity muss in Swift (Widget Extension) geschrieben werden, da Compose sie nicht rendern kann; maximale Laufzeit ~8 h, ausreichend für 60-min-Spiele.

**Externe Dienste**

Alle Kartendaten stammen aus **OpenStreetMap** (ODbL-Lizenz). Die Karte muss den Hinweis „© OpenStreetMap contributors" zeigen; MapLibre blendet ihn automatisch ein.

|Bedarf|Dienst|Hinweise|
|---|---|---|
|Kartenkacheln (Anzeige)|**OpenFreeMap**, öffentliche Instanz|Weltweit, kostenlos, kein API-Key, keine Anfragelimits. Spendenfinanziert, keine Verfügbarkeitsgarantie.|
|Gefährliche Bereiche (Serverprüfung)|**Overpass API**, öffentliche Instanz|Weltweit. Fair-Use-Richtlinie: ~10.000 Anfragen und ~1 GB pro Tag.|
|Push-Benachrichtigungen|**FCM / APNs**| |

- Beim Erstellen eines Spiels fragt der Server Overpass für die Bounding Box des Spiels ab, gefiltert auf Autobahnen, Gleise, Gewässer usw. Das Ergebnis wird gespeichert und von späteren Spielen im selben Gebiet wiederverwendet; wöchentliche Aktualisierung. Overpass wird nur für **neue Gebiete** aufgerufen, nicht pro Spiel oder Spieler.
- Jede Anfrage sendet einen `User-Agent`-Header, der taggd identifiziert.
- **Skalierungspfad** bei steigendem Traffic:

|Schritt|Ersetzen|Durch|
|---|---|---|
|1|Öffentliches Overpass|**Selbst gehostetes Overpass** (Docker, z. B. `wiktorn/overpass-api`), befüllt mit Geofabrik-Extrakten für die Regionen, in denen gespielt wird, automatisch aktualisiert|
|2|Öffentliches OpenFreeMap|**Protomaps**: eine `.pmtiles`-Datei (ganze Welt ~120 GB oder ein Regionsausschnitt) auf Cloud-Speicher wie Cloudflare R2; kein Tile-Server nötig|

**Rechtlich**

- **DSGVO:** Standortdaten dürfen nur temporär für die Dauer eines Spiels verarbeitet werden (siehe [[#Datenschutz (DSGVO)]]). Eine Datenschutzerklärung und eine In-App-Einwilligung für Standort-Tracking und Bluetooth sind nötig und werden vor dem öffentlichen Release geprüft.
- **Mindestalter:** 14 Jahre (digitales Einwilligungsalter in Österreich laut DSGVO); jüngere Spieler brauchen die Zustimmung der Eltern.
- **Lootboxen:** Bezahlte Lootboxen sind in Belgien verboten und wurden in österreichischen Gerichtsverfahren als Glücksspiel eingestuft. Vor einer Umsetzung ist rechtliche Beratung einzuholen. Falls verwendet: nur Kosmetik, mit angezeigten Drop-Raten.
- **App Stores:** Die iOS-Standortberechtigung „Immer" wird im App Store streng geprüft. Android Live Updates sind nur möglich, wenn Google eine Spielrunde als laufende, zeitkritische Aktivität akzeptiert.
- **Kosten bei Veröffentlichung:** Google Play Console (ca. 25 € einmalig), Apple Developer Program (ca. 99 €/Jahr).
