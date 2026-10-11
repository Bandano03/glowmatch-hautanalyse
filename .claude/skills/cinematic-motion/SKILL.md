---
name: cinematic-motion
description: Hebt Motion-Graphics-Videos (Remotion, HTML/CSS-Animation, Werbevideos, Explainer, Shorts, Reels, Produkt-Trailer, animierte Intros) auf Kino-Niveau. Immer verwenden, wenn ein animiertes Video, eine Motion-Graphic, ein Promo-Clip oder eine Remotion-Komposition erstellt oder verbessert wird.
---

# Cinematic Motion: Motion Graphics auf Trailer-Niveau

Ziel: Jedes Video soll wirken wie ein Kinotrailer, eine Apple-Keynote oder ein Titel-Intro einer Netflix-Serie. Niemand soll es für ein Template halten. Wenn ein Frame aus dem Video als Standbild nicht wie ein Filmplakat aussieht, ist es noch nicht fertig.

Dieser Skill gilt für alle Motion-Graphics-Arbeiten. Standard-Technik ist **Remotion** (React, gratis für Einzelpersonen). Fertige Code-Bausteine stehen in `references/remotion-patterns.md`: diese Datei lesen, bevor du Code schreibst.

---

## 0. Grundregeln für die Zusammenarbeit

- Der Nutzer ist Programmier-Anfänger. Erkläre am Ende jedes Arbeitsschritts in einfachen, nummerierten Schritten, was er tun muss (Vorschau öffnen, rendern, Datei finden). Keine Fachbegriffe ohne kurze Erklärung.
- Nur gratis und Open-Source-Werkzeuge. Nie kostenpflichtige Tools, APIs oder Credits verwenden ohne ausdrückliche Erlaubnis.
- Kommunikation auf Deutsch, direkt und ohne Füllwörter.

---

## 1. Verbotene Muster (der "Template-Look")

Diese Dinge machen ein Video sofort billig. Vermeide sie konsequent:

1. Lineare Bewegung (`interpolate` ohne Easing). Nichts in der echten Welt bewegt sich linear.
2. Alles erscheint gleichzeitig. Elemente kommen immer gestaffelt.
3. Simples Fade-in/Fade-out als einzige Animation.
4. Zentrierter Text auf flacher Einfarb-Fläche ohne Tiefe, Licht oder Textur.
5. Reines Schwarz `#000` oder reines Weiss `#FFF` als Fläche.
6. Standard-Systemschrift (Arial, Helvetica, Times) oder mehr als 2 Schriftfamilien.
7. Statische Kamera: das Bild steht über eine ganze Szene still.
8. Jede Szene gleich lang.
9. Wildes Hüpfen (zu viel Spring-Bounce), Drehungen ohne Grund, Regenbogenfarben.
10. Standard-Crossfade als Übergang zwischen allen Szenen.
11. Musik, die nicht zum Schnitt passt (Schnitte zwischen den Beats).

---

## 2. Pflicht-Workflow: erst denken, dann coden

Bevor eine einzige Zeile Code entsteht, erstelle ein kurzes **Treatment** und zeige es dem Nutzer (max. 15 Zeilen):

1. **Logline:** Was soll der Zuschauer nach dem Video fühlen und tun? (1 Satz)
2. **Look-Referenz:** Welcher Filmstil? z. B. "Blade Runner Neon-Nacht", "Apple Produkt-Reveal, kühl und edel", "A24 Trailer, körnig und warm", "Formel-1 Intro, aggressiv und schnell".
3. **Farbpalette:** 1 dunkle Basis, 1 Mittelton, 1 Akzentfarbe, 1 Lichtfarbe (Hex-Werte).
4. **Typografie:** 1 Display-Schrift + 1 Text-Schrift (Google Fonts).
5. **Format und Länge:** 9:16 (Shorts/Reels/TikTok) oder 16:9, Sekunden, fps.
6. **Shot-Liste:** jede Szene mit Startframe, Dauer, Kamerabewegung, Inhalt, Übergang.
7. **Audio-Plan:** Musik-Stimmung, BPM, wo Hits, Whooshes und Riser liegen.

Danach direkt umsetzen, ausser der Nutzer will etwas ändern.

---

## 3. Die 10 Säulen des cinematischen Looks

### 3.1 Virtuelle Kamera
- Die Kamera bewegt sich **immer**, auch wenn nur leicht. Mindestens ein langsamer Push-in (Scale 1.00 auf 1.06 über die Szene) oder ein seitlicher Drift.
- Baue jede Szene in einen `CameraRig` (siehe Patterns), der Scale, Translate und leichte Rotation steuert.
- Kamerabewegungen: Push-in (Spannung), Pull-out (Enthüllung), Dolly seitlich mit Parallax (Raum), Whip-Pan (Energie, Übergang), Handheld-Shake minimal (Dokumentar-Gefühl, Amplitude 1 bis 3 px).
- Kamera-Easing immer weich: `Easing.bezier(0.45, 0, 0.15, 1)`.

### 3.2 Tiefe und Parallax
- Jede Szene hat **3 bis 5 Ebenen**: Hintergrund, Atmosphäre, Hauptmotiv, Vordergrund-Elemente, Overlay.
- Ebenen bewegen sich bei Kamerafahrt unterschiedlich schnell (Hintergrund 0.2x, Mitte 0.6x, Vordergrund 1.4x).
- Tiefenschärfe: Hintergrund `blur(6-12px)`, Vordergrund-Partikel `blur(4-8px)`, nur das Hauptmotiv ist scharf.
- Rack-Focus: Schärfe wandert per Animation von einer Ebene zur anderen.
- Atmosphäre: schwebende Staub-Partikel, Bokeh-Kreise, Nebel-Verläufe, Lichtstrahlen.

### 3.3 Licht
- Licht erzeugt Stimmung. Arbeite mit radialen Verläufen als Lichtquellen, nicht mit flachen Farben.
- **Light Sweep:** ein schräger, heller Streifen gleitet über Text und Logos (Glanz-Effekt).
- **Glow/Bloom:** wichtige Elemente bekommen mehrere `drop-shadow` oder `text-shadow` Lagen in der Lichtfarbe.
- **Volumetrisches Licht:** halbtransparente Strahlen von oben oder hinten, leicht animiert.
- Licht ist motiviert: es kommt aus einer erkennbaren Richtung und bleibt in der Szene konsistent.

### 3.4 Farbe und Color Grading
- Begrenzte Palette (siehe Treatment). Bewährte Filmlooks:
  - Teal & Orange: Basis `#0B1A22`, Akzent `#FF8A3D`, Licht `#FFD7A8`
  - Noir Gold: Basis `#0A0908`, Akzent `#C9A45C`, Licht `#F3E3C0`
  - Neon Night: Basis `#07060F`, Akzent `#FF2E88`, zweiter Akzent `#2EE6FF`
  - Arctic Clean: Basis `#0C1116`, Akzent `#7FD1FF`, Licht `#EAF6FF`
- Nie reines Schwarz oder Weiss. Dunkel ist z. B. `#0A0A0F`, hell ist `#F5F2EA`.
- Finish-Ebene über dem ganzen Video (Reihenfolge von unten nach oben): Vignette, Film Grain, optional leichte chromatische Aberration an den Rändern.
- Kontrast bewusst steuern: das wichtigste Element ist immer das hellste im Bild.

### 3.5 Bewegung und Easing
- Nutze die Easing-Kurven aus den Patterns, nie linear:
  - `cinematic` (Standard): `bezier(0.45, 0, 0.15, 1)`
  - `snap` (schnelle Ankunft, weiches Auslaufen): `bezier(0.16, 1, 0.3, 1)`
  - `anticipate` (kurzes Ausholen): `bezier(0.68, -0.4, 0.32, 1)`
  - Spring nur mit `damping` 18 bis 30, damit nichts hüpft.
- **Staffelung (Stagger):** Elemente einer Gruppe starten 2 bis 4 Frames versetzt, Buchstaben 1 bis 2 Frames.
- **Motion Blur** bei schnellen Bewegungen (`@remotion/motion-blur`, `CameraMotionBlur`). Schnelle Bewegung ohne Blur sieht nach Computer aus.
- **Overlap:** die nächste Animation beginnt, bevor die vorige ganz fertig ist. Nie Stillstand zwischen Aktionen, ausser als bewusste Pause.
- Bewegung hat Gewicht: grosse Elemente bewegen sich langsamer als kleine.

### 3.6 Typografie
- Display-Schriften (gratis, `@remotion/google-fonts`): Bebas Neue, Anton, Syne, Space Grotesk, Unbounded, Playfair Display, Cormorant Garamond, Archivo Black. Text: Inter, Manrope, DM Sans.
- Starke Grössenkontraste: Hauptwort riesig (15 bis 25 % der Bildbreite), Nebentext klein und mit weitem Buchstabenabstand (`letter-spacing: 0.3em`, Grossbuchstaben).
- Text-Reveals statt Fade: Masken-Reveal (Text schiebt sich aus einer unsichtbaren Linie hoch), Tracking-Animation (Buchstabenabstand zieht sich zusammen), Wort-für-Wort mit Blur-zu-Scharf, Clip-Path-Wipe.
- Maximal 6 bis 8 Wörter gleichzeitig im Bild. Lieber mehr Schnitte.
- Text steht nie einfach in der Mitte: nutze Drittel-Regel, Anschnitte, bewusste Asymmetrie.

### 3.7 Übergänge
- Jeder Übergang hat eine Idee. Werkzeugkasten:
  - **Match Cut:** eine Form der alten Szene wird zur Form der neuen Szene (Kreis wird zur Sonne, Linie wird zum Horizont).
  - **Zoom-Through:** Kamera fliegt in ein Element hinein (z. B. in den Buchstaben "O"), dahinter liegt die neue Szene.
  - **Whip-Pan:** schneller seitlicher Schwenk mit starkem Motion Blur.
  - **Light Flash:** kurzer heller Blitz (2 bis 4 Frames) genau auf einem Musik-Hit.
  - **Mask Wipe:** eine Form wischt die neue Szene ins Bild.
  - **Hard Cut auf den Beat:** oft der stärkste Übergang überhaupt.
- Übergänge 8 bis 15 Frames, Hard Cuts 0 Frames. Nicht zwei Mal hintereinander denselben Übergang.

### 3.8 Rhythmus und Dramaturgie
- Struktur jedes Videos:
  1. **Hook (0 bis 2 s):** sofort etwas Unerwartetes, starkes Bild oder starke Aussage. Kein Logo am Anfang.
  2. **Aufbau:** Spannung steigt, Schnitte werden schneller.
  3. **Höhepunkt:** die wichtigste Aussage oder das Produkt, grösster visueller Moment, oft mit kurzer Stille davor.
  4. **Auflösung / CTA:** ruhiger, Logo, Handlungsaufforderung, 2 bis 3 s stehen lassen.
- Szenendauer variieren: z. B. 45, 30, 20, 15, 60 Frames. Nie alle gleich.
- **Atempausen:** vor dem Höhepunkt 6 bis 12 Frames Ruhe oder Dunkelheit. Stille macht den nächsten Hit doppelt so stark.

### 3.9 Sound Design
- Ton macht 50 % der Wirkung. Jedes Video bekommt Musik plus Effekte.
- Gratis Quellen (Lizenz jeweils prüfen und dem Nutzer nennen): Pixabay Music, YouTube Audio Library, Freesound.org (CC0 Filter), Mixkit.
- Schnitte und wichtige Animationen liegen **auf dem Beat**. Beat-Raster berechnen: `framesPerBeat = fps * 60 / BPM` (Helfer in den Patterns).
- Effekte: Whoosh bei Kamerafahrten und Whip-Pans, Impact/Hit beim Höhepunkt, Riser 1 bis 2 s vor dem Höhepunkt, leiser Ambient-Teppich (Room Tone) gegen tote Stille.
- Musik am Ende ausblenden, nicht abschneiden. Lautstärke von Effekten unter der Musik mischen (0.4 bis 0.7).
- Wenn keine Audiodatei vorhanden ist: Treatment mit Audio-Plan liefern und dem Nutzer Schritt für Schritt erklären, wo er passende Musik gratis herunterlädt und in welchen Ordner (`public/`) er sie legt.

### 3.10 Finish
- Film Grain über allem (Deckkraft 0.05 bis 0.12, animiert).
- Vignette (Ränder 30 bis 50 % dunkler).
- Optional Letterbox-Balken für 16:9 im Kino-Look (2.39:1), sanft ins Bild fahrend.
- Optional Light Leaks: warme, weiche Lichtflecken, die langsam über das Bild wandern.
- Endcard: Logo mit Light Sweep, kurze Zeile, ruhige Kamera.

---

## 4. Formate und technische Einstellungen

| Zweck | Auflösung | fps |
|---|---|---|
| Shorts, Reels, TikTok | 1080 x 1920 | 30 (60 bei sehr schnellen Bewegungen) |
| YouTube, Website | 1920 x 1080 | 30 oder 60 |
| Kino-Look, ruhig | 1920 x 1080 mit Letterbox | 24 |

- Bei 9:16 Safe Zones beachten: oben 250 px und unten 400 px frei von wichtigem Text (dort liegen App-Bedienelemente).
- Rendern in hoher Qualität:
  `npx remotion render <CompositionId> out/video.mp4 --codec=h264 --crf=16`
- Bilder und Logos immer als SVG oder hochauflösendes PNG, nie verpixelt. Logos als echtes SVG bauen, keine KI-Bilder.

---

## 5. Qualitätskontrolle vor der Abgabe (Pflicht)

1. Rendere Standbilder an 5 bis 8 Schlüsselmomenten:
   `npx remotion still <CompositionId> out/check-<frame>.png --frame=<frame>`
2. Schau dir jedes Bild selbst an (Bild-Datei öffnen/lesen).
3. Bewerte jedes Standbild ehrlich von 1 bis 10 in: Kamera/Tiefe, Licht, Farbe, Typografie, Komposition.
4. Frage dich: "Würde jemand dieses Standbild für einen Frame aus einem Kinotrailer halten?"
5. Alles unter 8 wird überarbeitet, dann erneut prüfen. Maximal 3 Runden, danach dem Nutzer den Stand zeigen.
6. Prüfe die Verbotsliste aus Abschnitt 1 Punkt für Punkt.
7. Erst dann das finale Video rendern.

---

## 6. Abschluss für den Nutzer

Am Ende immer eine kurze, nummerierte Anleitung in einfachen Worten:

1. Wie er die Vorschau öffnet (`npm run dev` bzw. `npx remotion studio`, dann die angezeigte Adresse im Browser öffnen).
2. Wie er das finale Video rendert (genauer Befehl zum Kopieren).
3. Wo die fertige Datei liegt (Ordner `out/`).
4. Was er einfach selbst ändern kann (Texte, Farben, Musikdatei) und in welcher Datei und Zeile.
