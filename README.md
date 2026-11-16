# WISO Steuer unter Debian Linux (mit Bottles)
Installation der Wiso Steuer 2027 Windows Version unter Linux mit Bottles / Wine. Incl. Druckvorschau PDF/Web und Anbindung an das lokale Filesystem und Netzwerk.

Diese Anleitung beschreibt die Installation und Konfiguration der Desktop-Version von WISO Steuer unter Debian Linux mittels Bottles (Flatpak). Sie umfasst auch die Behebung von Anzeigefehlern (WebView2), den Zugriff auf lokale CIFS-Shares und die Integration in das KDE Plasma Startmenü.

Für die Offline-Desktop-Version folgen Sie dieser Anleitung.

## 1. Einrichtung von WISO Steuer über Bottles

### 1.1 Bottles installieren (über Flathub)
Installieren Sie Bottles auf Ihrem Debian-System am einfachsten als Flatpak. Öffnen Sie das Terminal:

```bash
flatpak install flathub com.usebottles.bottles
```

### 1.2 Neue Bottle erstellen
Starten Sie Bottles und legen Sie eine neue Umgebung ("Bottle") an:
- **Name:** z.B. *WISO_Steuer*
- **Umgebungstyp:** Anwendung (Application)
- **Architektur:** 64-Bit (WISO Steuer ist nur noch 64-Bit)
- **Windows-Version:** Windows 10

### 1.3 Abhängigkeiten (Dependencies) installieren
Wechseln Sie in die Einstellungen der neu erstellten Bottle und klicken Sie auf **Abhängigkeiten**. Installieren Sie zwingend:
- `allfonts` (verhindert kryptische oder fehlende Schriften)
- `mono` (Wine-Mono, wird für .NET-Komponenten benötigt)

### 1.4 WISO Steuer installieren
1. Laden Sie die offizielle Windows-Installationsdatei (z.B. `wiso_steuer_2024.exe`) herunter.
2. Klicken Sie in Bottles auf **Führe ausführbare Datei aus** und wählen Sie das Setup.
3. Folgen Sie dem normalen Installationsassistenten. Das Programm-Icon erscheint danach in der Bottles-Übersicht unter "Programme".

---

## 2. Behebung der Edge/WebView2-Abhängigkeit (Leere/Schwarze Fenster)

WISO Steuer nutzt für die Druckvorschau und UI-Elemente Microsoft Edge WebView2. Fehlt dies, führt das unter Wine oft zu leeren Fenstern.

### 2.1 WebView2 über Bottles installieren
1. Navigieren Sie in Bottles zu den **Abhängigkeiten** Ihrer WISO-Bottle.
2. Suchen Sie nach `webview2` und klicken Sie auf das Download-Symbol daneben.

*(Fallback: Scheitert der Download, laden Sie den **Evergreen Standalone Installer (x64)** direkt bei Microsoft herunter und führen Sie die `.exe` in der Bottle als ausführbare Datei aus).*

### 2.2 Hardwarebeschleunigung für WebView2 anpassen
Selbst wenn WebView2 installiert ist, kann es zu schwarzen Flächen kommen.
1. Installieren Sie in den Bottles-Abhängigkeiten zusätzlich `d3dcompiler_47`.
2. Wechseln Sie in die Einstellungen der Bottle und stellen Sie sicher, dass unter **Grafik** der **DXVK**-Renderer aktiviert ist.

---

## 3. CIFS-Shares als Windows-Laufwerk einbinden

Um lokal gemountete CIFS-Shares (z.B. `/home/meinuser/meinefinanzen`) als Laufwerksbuchstaben in Bottles nutzbar zu machen, muss die Flatpak-Sandbox geöffnet werden.

### 3.1 Sandbox-Rechte mit Flatseal erweitern
1. Starten Sie **Flatseal** und wählen Sie links in der Liste **Bottles** aus.
2. Scrollen Sie rechts zu **Dateisystem** (Filesystem) -> **Andere Dateien** (Other files).
3. Klicken Sie auf das **+** Symbol und tragen Sie den exakten Mountpoint ein: 
   `/home/meinuser/meinefinanzen`
4. Schließen Sie Flatseal und **beenden Sie Bottles komplett** (prüfen Sie, dass kein Symbol mehr im KDE-Systemabschnitt/Taskleiste aktiv ist), bevor Sie es neu starten.

*(Alternativ im Terminal: `flatpak override --user --filesystem=/home/meinuser/meinefinanzen com.usebottles.bottles`)*

### 3.2 Laufwerksbuchstaben in Bottles anlegen
1. Öffnen Sie die **Einstellungen** Ihrer WISO-Bottle.
2. Scrollen Sie zu **Laufwerke** (Drives) und klicken Sie auf das **+** Symbol.
3. Wählen Sie einen freien Buchstaben (z.B. `X:`) und geben Sie als Pfad exakt `/home/meinuser/meinefinanzen` an.

> **Wichtig zu CIFS-Rechten:** Wine läuft unter Ihrem lokalen Linux-Benutzer. Der CIFS-Share muss mit Ihren Benutzerrechten gemountet sein (z.B. via Optionen `uid=1000,gid=1000`), da WISO Steuer sonst beim Speichern mangels Schreibrechten auf dem Share eine Fehlermeldung ausgibt.

---

## 4. Startmenü-Eintrag (KDE Plasma) erstellen

1. Öffnen Sie Ihre WISO-Bottle und scrollen Sie nach unten zu **Programme**.
2. Klicken Sie rechts neben dem Eintrag "WISO Steuer" auf das Menü-Symbol (**⋮**).
3. Wählen Sie **Zum Desktop hinzufügen** (Add to Desktop).
4. Öffnen Sie Ihr KDE-Startmenü und suchen Sie nach "WISO". Das Programm kann nun direkt von hier gestartet oder an die Taskleiste angeheftet werden, ohne zuerst die Bottles-GUI öffnen zu müssen.
