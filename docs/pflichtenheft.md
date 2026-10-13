# taggd – Pflichtenheft

**Team:** Erik, Almir, Lukas, Julian

---

## 1. Pflichtenheft

### 1.1 Ausgangssituation

Jugendliche und junge Erwachsene verbringen einen Großteil ihrer Freizeit vor Bildschirmen.
Dadurch gehen sie selten raus und machen kaum Sport. Es fällt ihnen schwer sich mit anderen zu vernetzen.

### 1.2 Istzustand

- Wenn Fangspiele greifen Gruppen auf Messenger-Apps zurück (z. B. Standortbild über WhatsApp).
- Ob ein Fang zählt, wer sich wo aufhält und ob alle fair spielen, wird von den Spielern selbst ausgehandelt.

### 1.3 Problemstellung

- **GPS zu ungenau:** Nur 25 Meter Genauigkeit.
- **Sicherheit:** Spieler achten weniger auf den Verkehr. Gefährliche Bereiche (Autobahnen, Gleise, Gewässer) müssen erkannt werden.
- **Camping:** Wer sich das ganze Spiel zu Hause versteckt, darf damit keinen Vorteil haben.
- **Datenschutz:** Standortdaten sind personenbezogene Daten und dürfen nicht dauerhaft gespeichert werden.
- **Handy in der Hosentasche:** Das Spiel muss auch bei gesperrtem Bildschirm zuverlässig weiterlaufen.

### 1.4 Aufgabenstellung

taggd ist ein ortsbasiertes Spiel (Location-based Game): „Fangen", aber in der Größe eines Stadtteils. Gespielt wird ausschließlich zu Fuß.

Es wird eine Mobile-App (Android und iOS) samt Server entwickelt, mit der Spieler in zwei Rollen gegeneinander antreten:

- **Hunter:** Sieht Hunted als **Pings** auf der Karte, standardmäßig alle 2 min, schneller bei Annäherung
- **Hunted:** Sieht die Live-Positionen der Hunter. Versucht so lange wie möglich zu überleben.

#### 1.4.1 Funktionale Anforderungen

##### Spielmodi

- **Privat:** Private Lobbys für Freundesgruppen mit konfigurierbaren Spielregeln (Ping-Intervalle, Bots, Rundendauer, ...).
- **Public:** Möglichkeit zum Beitritt in laufende Runden und Auffüllen kleinerer Runden durch Bots (Bots sind klar gekennzeichnet).
- **Ranked:** Wettbewerbsmodus mit Elo-Wertung und Ranglistensystem.

##### Kernmechaniken

- **Hunter-Anteil:** standardmäßig sind 30 % der Spieler Hunter.
- **Taggen:** Wenn der Hunter in der Reichweite von einem Hunted ist kann er getaggt werden.
- **Win-Condition:**
  - **Hunter gewinnen**, wenn alle Hunted getaggt wurden.
  - **Hunted gewinnen**, wenn mindestens einer von ihnen bis zum Ende des Timers überlebt.
- **Spiel-Timer:** max. 60 min.

##### Dynamische Pings

- Jeder Hunter bekommt Pings für jeden Hunted.
- Das Intervall hängt von der Distanz zwischen diesem Hunter und diesem Hunted ab.

##### Zusatzmechaniken

- **Bewegliche Safe-Zonen:** Hunted können darin nicht getaggt werden.
- **Bewegliche No-Go-Zonen:** Kein Spieler darf rein.
- **Schrumpfender Sturm:** treibt die Spieler zusammen und beendet Spiele.
  - Sein Mittelpunkt wandert während des Spiels, damit kein fixer Ort sicher bleibt.
  - Ein Spieler außerhalb des Sturmkreises wird disqualifiziert.
- **Zuschauermodus:** Der Zuschauer-Feed ist verzögert, damit Zuschauer den Spielern keine Positionen verraten können.
- **Power-ups:** im Shop verfügbar: Unsichtbarkeit, Extra-Reichweite, Ghost-Ping.

##### Fortschritt & Soziales

- Accounts mit Levelsystem.
- Achievements.
- Freunde per Freundescode oder QR-Code.
- Coins verdient durch:
  - zurückgelegte Distanz
  - Taggen von Spielern
  - Abschließen eines Spiels
  - Nahe an Hunter sein
- Shop für Power-ups und Kosmetik

##### Spielersicherheit

- Spieler, die gefährliche Bereiche betreten (Autobahnen, Gleise, Gewässer, …), bekommen sofort eine Warnung. Sie werden disqualifiziert wenn sie in diesen Bereichen bleiben.
- Sicherheitshinweise (auf den Verkehr achten, auf dem Gehsteig bleiben, …).

#### 1.4.2 Nichtfunktionale Anforderungen

##### Akku & Performance

- Die Karte wird nur bei eingeschaltetem Bildschirm gezeichnet.
- Benachrichtigungen statt dauerhaft offener App.

##### Plattform

- Die App soll auf iOS und Android laufen.

### 1.5 Ziele

- **Förderung von Bewegung und Gesundheit:** Jugendliche und junge Erwachsene werden durch spielerische Motivation dazu bewegt, sich aktiv zu Fuß im Freien aufzuhalten.
- **Stärkung sozialer Interaktionen:** Gemeinsame Erlebnisse und Teamgeist; man kann mit Freunden spielen und neue Personen kennenlernen.
- **Faire, überprüfbare Fänge:** Ein Tag zählt nur, wenn GPS und Bluetooth die Nähe bestätigen und der Server ihn prüft.
- **Sicheres Spielen:** Gefährliche Bereiche werden erkannt, Spieler gewarnt und bei Missachtung disqualifiziert.

### 1.6 Mengengerüst

- Standortübertragung pro Spieler
- Aufbewahrung Tag-Koordinaten / Cheat-Nachweise

### 1.7 Rahmenbedingungen

**Organisatorisch**

- Entwicklerteam: Erik, Almir, Lukas und Julian.
- Das Projekt muss in 1,5 Jahren fertiggestellt werden.
- Während der Entwicklung fallen keine Kosten an: bestehende Hardware (Lukas Server), ausschließlich freie Open-Source-Tools und freie Geodaten.

**Technisch**

- **Zielplattformen:** Android und iOS; Release im Google Play Store und App Store optional.
- **Hardware:** GPS für die grobe Ortung, Bluetooth für das Taggen.
