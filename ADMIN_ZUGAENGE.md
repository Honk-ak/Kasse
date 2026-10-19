# Eigene Zugänge für Kasse und Verwaltung

Die App unterstützt neben dem bestehenden Hauptadministrator weitere verantwortliche Personen mit jeweils eigener E-Mail-Adresse, eigenem Passwort und eigener Firebase-UID. Passwörter werden niemals geteilt oder in Dateien eingetragen.

## Sicherheitsmodell

- Der bisherige Hauptadministrator bleibt dauerhaft freigeschaltet.
- Weitere Verantwortliche werden in der Firebase Realtime Database unter `adminAccess/<UID>/active` freigegeben.
- Jede Person meldet sich mit ihrem eigenen Firebase-Konto an.
- Die App liest nur den eigenen Freigabeeintrag. Andere freigeschaltete Personen sind für normale App-Benutzer nicht sichtbar.
- Die App kann die Freigabeliste nicht selbst verändern. Einträge werden ausschließlich in der Firebase-Konsole gepflegt.
- Private Standgeräte verwenden weiterhin die zeitlich begrenzte Gerätefreigabe und gehören nicht in diese Liste.

## 1. Benutzerkonto anlegen

Für jede verantwortliche Person einzeln:

1. Firebase-Konsole öffnen und das Projekt `veranstaltungskasse` auswählen.
2. **Authentication → Users** öffnen.
3. **Add user / Nutzer hinzufügen** auswählen.
4. Die persönliche E-Mail-Adresse der Person und ein einmaliges, zufälliges Startpasswort eintragen.
5. Den neuen Benutzer anlegen.
6. Beim Benutzer das Drei-Punkte-Menü öffnen und **Reset password / Passwort zurücksetzen** auswählen.
7. Die Person legt über die erhaltene E-Mail ihr eigenes Passwort fest. Das endgültige Passwort wird nicht an andere weitergegeben.
8. Die in der Benutzerliste angezeigte **User UID** kopieren.

## 2. UID freigeben

1. In Firebase **Realtime Database → Data / Daten** öffnen.
2. Auf der obersten Ebene den Knoten `adminAccess` anlegen, falls er noch nicht existiert.
3. Unter `adminAccess` einen neuen Kindknoten anlegen. Als Schlüssel exakt die kopierte Firebase-UID verwenden.
4. Unter dieser UID folgende Werte anlegen:

```json
{
  "active": true,
  "name": "Martin"
}
```

Die entstehende Struktur sieht beispielsweise so aus:

```text
adminAccess
└── FIREBASE_UID_DER_PERSON
    ├── active: true
    └── name: "Martin"
```

`name` ist nur eine interne Bezeichnung. Für die Berechtigung ist ausschließlich `active: true` maßgeblich.

## 3. Aktuelle Datenbankregeln veröffentlichen

1. **Realtime Database → Rules / Regeln** öffnen.
2. Den vollständigen Inhalt aus `firebase/firebase-database-rules-device-access.json` einsetzen.
3. **Publish / Veröffentlichen** auswählen.

Die Regeln erlauben freigeschalteten Verantwortlichen Kasse und Verwaltung. Die bestehende Hauptadministration bleibt erhalten. Private Standgeräte behalten ausschließlich ihren zeitlich und veranstaltungsbezogen begrenzten Zugriff.

## 4. Anmeldung testen

1. Die Kassen-App auf dem Gerät der Person neu öffnen.
2. Unter **Kasse / Verwaltung** die persönliche E-Mail-Adresse und das persönliche Passwort eingeben.
3. Auf einem neuen Gerät **Kasse / Verwaltung** als Geräteart wählen.
4. Die Verwaltungsansicht öffnen und die Verwaltungs-PIN eingeben.
5. Prüfen, ob Veranstaltung, Kassenbestand, Produkte, Gerätefreigaben und Auswertung erreichbar sind.

## Zugang sperren

Für eine sofortige Sperrung:

1. Unter `adminAccess/<UID>/active` den Wert auf `false` setzen.
2. Den Benutzer zusätzlich unter **Authentication → Users** deaktivieren.

Durch `active: false` verweigern die Datenbankregeln weitere Verwaltungszugriffe. Die Deaktivierung des Benutzerkontos verhindert außerdem künftige Anmeldungen.

## Empfohlene Besetzung

Für den laufenden Betrieb genügen üblicherweise der Hauptadministrator und bis zu drei weitere verantwortliche Personen. Es sollten ausschließlich Personen freigeschaltet werden, die Kassenbestand, Veranstaltungen, Produktkatalog, Datenlöschung und private Standgeräte verwalten dürfen.
