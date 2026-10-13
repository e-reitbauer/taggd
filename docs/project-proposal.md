# Projektantrag: taggd

## Inhaltsverzeichnis

1. Ausgangslage
2. Rahmenbedingungen und Randbedingungen
3. Projektziele und Systemkonzept
4. Chancen und Risiken
5. Projektplanung und Ressourcen
6. Wirtschaftlichkeit

---

## 1. Ausgangslage

Jugendliche und junge Erwachsene verbringen einen Großteil ihrer Freizeit bewegungsarm vor Bildschirmen.
Wenn Fangspiele im städtischen Raum organisiert werden, greifen Gruppen auf Messenger-Apps (z.B. Bild über WhatsApp) zurück.

---

## 2. Rahmenbedingungen und Randbedingungen

* **Organisatorische und personelle Rahmenbedingungen:**
  * Entwicklerteam: Erik, Almir, Lukas und Julian.
  * Das Projekt muss in 1,5 Jahren fertiggestellt werden.
* **Technische Bedingungen & IT-Infrastruktur:**
  * **Zielplattformen:** Android und iOS.
  * **Hardwareschnittstellen:** GPS-Sensoren für die grobe Ortung und Bluetooth Low Energy (BLE) für die Nahbereichs-Interaktion (Taggen).
* **Standards, Richtlinien und Sicherheitsauflagen:**
  * **Datenschutz (DSGVO):** Standortdaten dürfen nur temporär für die Dauer eines Matches verarbeitet werden.
  * **Physische Sicherheit im öffentlichen Raum:** Gefährliche Bereiche (Autobahnen, Zugtrassen, offene Gewässer) müssen erkannt und gekennzeichnet werden. Jeder der sich auf diesem Bereich bewegt wird aus dem Spiel gekickt.

---

## 3. Projektziele und Systemkonzept

### 3.1. Projektziele

* **Förderung von Bewegung und Gesundheit:** Jugendliche und junge Erwachsene werden durch spielerische Motivation dazu bewegt, sich aktiv zu Fuß im Freien aufzuhalten.

* **Stärkung sozialer Interaktionen:** Das Projekt ermöglicht gemeinsame Erlebnisse und Teamgeist. Man kann mit Freunden spielen und auch neue Personen kennenlernen.

### 3.2. Systemkonzept

* **Rollenkonzept:**
  * **Hunter (Jäger):** Sieht die Gejagten (Hunted) in regelmäßigen Zeitabständen (standardmäßig alle 2 Minuten, öfter bei Annäherung) über Pings auf der Karte. Ziel ist es, alle Gejagten (Hunted) zu fangen.
  * **Hunted (Gejagte):** Sehen die Jäger über Pings auf der Karte und müssen bis zum Rundenende überleben.
* **Kernmechaniken:**
  * **Fangen:** Die App verifiziert über Kombination von GPS und Bluetooth Low Energy, dass sich Jäger und Gejagter tatsächlich physisch auf wenige Meter angenähert haben.
  * **Dynamische Zonen & Rundenbegrenzung:** Runden dauern maximal 60 Minuten. Ein sich verkleinernder „Sturm“ schränkt den Bereich ein; bewegliche Safe-Zonen bieten temporären Schutz.
  * **Sicherheitssystem:** Disqualifikation bei Erkennung unzulässiger Geschwindigkeiten(z.B. Nutzung von Fahrrädern oder Autos) sowie Warnung und Rundenabbruch bei Betreten von Gefahrenzonen (Gleise, Autobahnen).
  * **Gamification:** Level, Achievements, In-Game-Münzen, Power-ups und ein rein kosmetischer Shop zur Individualisierung.
* **Spielmodi:**
  * **Privat:** Private Lobbys für Freundesgruppen mit konfigurierbaren Spielregeln und Ping-Intervallen.
  * **Public:** Automatisches Matchmaking, Möglichkeit zum Beitritt in laufende Runden und Auffüllen kleinerer Runden durch Bots.
  * **Ranked:** Wettbewerbsmodus mit Elo-Wertung und Ranglistensystem.

---

## 4. Chancen und Risiken

* **Chancen und Nutzenpotenziale:**
  * **Zielgruppe & Marktlücke:** Hohes Potenzial bei Jugendlichen und jungen Erwachsenen, die soziale Interaktion mit Bewegung im Freien verknüpfen wollen.
  * **Gesundheitlicher Mehrwert:** Spielerische Motivation zu sportlicher Aktivität abseits reiner Bildschirmzeiten.
* **Risiken und Gegenmaßnahmen:**
  * **Risiko physischer Unfälle:** Spieler könnten im Eifer des Spiels den Straßenverkehr vernachlässigen.
    *Gegenmaßnahme:* Automatische Erkennung gefährlicher Infrastrukturen via Overpass API, Einrichtung von No-Go-Zonen und sofortige Disqualifikation bei Missachtung.
  * **Risiko technischer Ortungsungenauigkeiten:** GPS-Drift in dicht bebauten Häuserschluchten könnte Fehlfänge erzeugen.
    *Gegenmaßnahme:* Zwei-Faktor-Verifikation des Fangs (GPS zur Annäherungsprüfung plus lokaler Bluetooth-Handshake zur Nahdistanzbestätigung).
  * **Risiko geringer Nutzerzahlen zu Beginn:** Öffentliche Partien könnten mangels aktiver Spieler unattraktiv sein.
    *Gegenmaßnahme:* Fokus auf private Gruppenrunden sowie Implementierung Bots zur Auffüllung öffentlicher Lobbys.

---

## 5. Projektplanung und Ressourcen

* **Grobe Meilensteine:**
  * **M1 – Projektstart (13.10.2026):** Spezifikation der Schnittstellen, Datenmodelle (ERD).
* **Ressourceneinsatz:**
  * **Personelle Ressourcen:** Vier Entwickler über den gesamten Projektzeitraum.
  * **Software & Lizenzen:** Ausschließlich kostenfreie Open-Source-Lizenzen, Open-Source-Tools und freie Geodaten.
  * **Testinfrastruktur:** Eigene mobile Testgeräte (Smartphones mit Android und iOS).

---

## 6. Wirtschaftlichkeit

* **Entwicklungsaufwand & Kosten:**
  * Während der Entwicklungsphase fallen **keine finanziellen Kosten (0 €)** an, da bestehende Hardware genutzt wird und alle Services auf freien Open-Source-Lösungen basieren.
* **Kosten im Falle einer späteren Veröffentlichung:**
  * Google Play Store/AppStore: Google Play Console (ca. 25€ einmalig) sowie für das Apple Developer Program (ca. 99€/Jahr).
  * Serverinfrastruktur: Proxmox-Cluster für das Backend bei Lukas
* **Wirtschaftlicher und strategischer Ertrag:**
  * **Monetarisierungschance:** Refinanzierung über rein kosmetische In-App-Käufe im Shop möglich, ohne Pay-to-Win-Vorteile zu bieten.
  * **Gesamtwirtschaftliches Urteil:** Das Projekt zeichnet sich durch ein äußerst vorteilhaftes Aufwand-Nutzen-Verhältnis aus, da der hohe technische Lerneffekt und gesellschaftliche Mehrwert ohne Investitionsrisiko erzielt werden.
