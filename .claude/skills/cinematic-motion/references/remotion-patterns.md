# Remotion Code-Bausteine für den cinematischen Look

Diese Bausteine in das Projekt übernehmen (z. B. als `src/cinematic/*.tsx`) und anpassen. Alle Werte sind Startpunkte, keine Gesetze.

Benötigte Pakete (alle gratis):
```bash
npx remotion add @remotion/transitions @remotion/motion-blur @remotion/google-fonts
```
Falls `npx remotion add` nicht verfügbar ist: `npm i @remotion/transitions @remotion/motion-blur @remotion/google-fonts` und darauf achten, dass die Versionsnummer exakt der `remotion` Version in `package.json` entspricht.

---

## 1. Easing-Kurven

```tsx
// src/cinematic/easing.ts
import { Easing, interpolate } from "remotion";

export const ease = {
  cinematic: Easing.bezier(0.45, 0, 0.15, 1),   // Standard, weich rein und raus
  snap: Easing.bezier(0.16, 1, 0.3, 1),          // schnelle Ankunft, langes Auslaufen
  anticipate: Easing.bezier(0.68, -0.4, 0.32, 1),// kurzes Ausholen vor der Bewegung
  slowIn: Easing.bezier(0.7, 0, 0.84, 0),        // beschleunigt (für Abgänge)
};

// Kurzform: Wert von a nach b zwischen Frame start und end
export const tween = (
  frame: number,
  start: number,
  end: number,
  from: number,
  to: number,
  easing = ease.cinematic
) =>
  interpolate(frame, [start, end], [from, to], {
    easing,
    extrapolateLeft: "clamp",
    extrapolateRight: "clamp",
  });
```

---

## 2. Virtuelle Kamera

```tsx
// src/cinematic/CameraRig.tsx
import React from "react";
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from "remotion";
import { tween, ease } from "./easing";

type Props = {
  children: React.ReactNode;
  zoomFrom?: number;   // z. B. 1.0
  zoomTo?: number;     // z. B. 1.08 (Push-in) oder 0.95 (Pull-out)
  panX?: number;       // seitliche Fahrt in px über die ganze Szene
  panY?: number;
  rotate?: number;     // Grad, sehr klein halten (0.5 bis 1.5)
  shake?: number;      // Handheld-Amplitude in px (0 = aus, 1 bis 3 = subtil)
};

export const CameraRig: React.FC<Props> = ({
  children,
  zoomFrom = 1,
  zoomTo = 1.06,
  panX = 0,
  panY = 0,
  rotate = 0,
  shake = 0,
}) => {
  const frame = useCurrentFrame();
  const { durationInFrames } = useVideoConfig();
  const end = durationInFrames - 1;

  const scale = tween(frame, 0, end, zoomFrom, zoomTo, ease.cinematic);
  const x = tween(frame, 0, end, 0, panX, ease.cinematic);
  const y = tween(frame, 0, end, 0, panY, ease.cinematic);
  const r = tween(frame, 0, end, 0, rotate, ease.cinematic);

  // deterministisches Handheld-Wackeln (Sinus-Überlagerung, kein Math.random)
  const sx = shake * (Math.sin(frame * 0.13) + 0.5 * Math.sin(frame * 0.37));
  const sy = shake * (Math.cos(frame * 0.11) + 0.5 * Math.sin(frame * 0.29));

  return (
    <AbsoluteFill
      style={{
        transform: `translate(${x + sx}px, ${y + sy}px) scale(${scale}) rotate(${r}deg)`,
        transformOrigin: "50% 50%",
      }}
    >
      {children}
    </AbsoluteFill>
  );
};
```

Hinweis: `useVideoConfig().durationInFrames` liefert innerhalb einer `<Sequence>` die Dauer dieser Sequenz. So passt sich die Kamerafahrt automatisch an die Szenenlänge an.

---

## 3. Parallax-Ebenen und Tiefenschärfe

```tsx
// src/cinematic/Layer.tsx
import React from "react";
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from "remotion";
import { tween } from "./easing";

type Props = {
  children: React.ReactNode;
  depth: number;      // 0.2 = weit hinten, 1 = Hauptebene, 1.4 = Vordergrund
  travel?: number;    // Gesamtfahrt der Szene in px
  blur?: number;      // Tiefenunschärfe in px
};

export const Layer: React.FC<Props> = ({ children, depth, travel = 120, blur = 0 }) => {
  const frame = useCurrentFrame();
  const { durationInFrames } = useVideoConfig();
  const x = tween(frame, 0, durationInFrames - 1, 0, -travel * depth);
  return (
    <AbsoluteFill
      style={{
        transform: `translateX(${x}px) scale(${1 + (depth - 1) * 0.05})`,
        filter: blur ? `blur(${blur}px)` : undefined,
      }}
    >
      {children}
    </AbsoluteFill>
  );
};
```

Rack-Focus: `blur` animieren, z. B. Hintergrund von 0 auf 10 und gleichzeitig Vordergrund von 10 auf 0.

---

## 4. Atmosphäre: schwebende Partikel / Bokeh

```tsx
// src/cinematic/Particles.tsx
import React from "react";
import { AbsoluteFill, random, useCurrentFrame, useVideoConfig } from "remotion";

export const Particles: React.FC<{ count?: number; color?: string; bokeh?: boolean }> = ({
  count = 60,
  color = "#FFD7A8",
  bokeh = false,
}) => {
  const frame = useCurrentFrame();
  const { width, height } = useVideoConfig();
  return (
    <AbsoluteFill style={{ pointerEvents: "none" }}>
      {new Array(count).fill(0).map((_, i) => {
        const size = bokeh ? 20 + random(`s${i}`) * 80 : 1 + random(`s${i}`) * 3;
        const speed = 0.1 + random(`v${i}`) * 0.4;
        const x = random(`x${i}`) * width + Math.sin(frame * 0.01 + i) * 20;
        const y = (random(`y${i}`) * height - frame * speed + height) % height;
        const opacity = (bokeh ? 0.08 : 0.25) + random(`o${i}`) * 0.35;
        return (
          <div
            key={i}
            style={{
              position: "absolute",
              left: x,
              top: y,
              width: size,
              height: size,
              borderRadius: "50%",
              background: color,
              opacity,
              filter: bokeh ? `blur(${size * 0.15}px)` : "blur(0.5px)",
              boxShadow: `0 0 ${size * 2}px ${color}`,
            }}
          />
        );
      })}
    </AbsoluteFill>
  );
};
```

Wichtig: Immer `random()` von Remotion verwenden, nie `Math.random()`. Sonst flackert das Video beim Rendern.

---

## 5. Licht: Hintergrund, Light Sweep, Strahlen

```tsx
// src/cinematic/Light.tsx
import React from "react";
import { AbsoluteFill, useCurrentFrame } from "remotion";
import { tween, ease } from "./easing";

// Hintergrund mit motivierter Lichtquelle statt flacher Farbe
export const LitBackground: React.FC<{ base: string; light: string; x?: string; y?: string }> = ({
  base,
  light,
  x = "70%",
  y = "30%",
}) => (
  <AbsoluteFill
    style={{
      background: `radial-gradient(ellipse 60% 50% at ${x} ${y}, ${light}55 0%, transparent 70%),
                   radial-gradient(ellipse 120% 80% at 50% 120%, ${light}18 0%, transparent 60%),
                   ${base}`,
    }}
  />
);

// Glanzstreifen über Text oder Logo. Als Kind über das Element legen,
// Eltern-Element braucht position: relative und overflow: hidden.
export const LightSweep: React.FC<{ start: number; duration?: number; color?: string }> = ({
  start,
  duration = 24,
  color = "rgba(255,255,255,0.55)",
}) => {
  const frame = useCurrentFrame();
  const x = tween(frame, start, start + duration, -60, 160, ease.cinematic);
  return (
    <div
      style={{
        position: "absolute",
        inset: 0,
        background: `linear-gradient(105deg, transparent ${x - 12}%, ${color} ${x}%, transparent ${x + 12}%)`,
        mixBlendMode: "overlay",
        pointerEvents: "none",
      }}
    />
  );
};

// Volumetrische Lichtstrahlen von oben
export const GodRays: React.FC<{ color?: string }> = ({ color = "#FFE9C4" }) => {
  const frame = useCurrentFrame();
  const sway = Math.sin(frame * 0.02) * 3;
  return (
    <AbsoluteFill
      style={{
        background: `repeating-conic-gradient(from ${200 + sway}deg at 60% -20%,
          ${color}22 0deg 4deg, transparent 4deg 11deg)`,
        maskImage: "linear-gradient(to bottom, black 0%, transparent 85%)",
        WebkitMaskImage: "linear-gradient(to bottom, black 0%, transparent 85%)",
        mixBlendMode: "screen",
        filter: "blur(6px)",
      }}
    />
  );
};

// Glow für wichtige Elemente
export const glow = (color: string) =>
  `drop-shadow(0 0 8px ${color}) drop-shadow(0 0 24px ${color}88) drop-shadow(0 0 60px ${color}44)`;
```

---

## 6. Finish: Grain, Vignette, Letterbox, Flash, Aberration

```tsx
// src/cinematic/Finish.tsx
import React from "react";
import { AbsoluteFill, useCurrentFrame } from "remotion";
import { tween, ease } from "./easing";

export const FilmGrain: React.FC<{ opacity?: number }> = ({ opacity = 0.08 }) => {
  const frame = useCurrentFrame();
  return (
    <AbsoluteFill style={{ opacity, mixBlendMode: "overlay", pointerEvents: "none" }}>
      <svg width="100%" height="100%">
        <filter id="grain">
          <feTurbulence type="fractalNoise" baseFrequency="0.85" numOctaves={3} seed={frame % 12} />
          <feColorMatrix type="saturate" values="0" />
        </filter>
        <rect width="100%" height="100%" filter="url(#grain)" />
      </svg>
    </AbsoluteFill>
  );
};

export const Vignette: React.FC<{ strength?: number }> = ({ strength = 0.55 }) => (
  <AbsoluteFill
    style={{
      background: `radial-gradient(ellipse at center, transparent 45%, rgba(0,0,0,${strength}) 100%)`,
      pointerEvents: "none",
    }}
  />
);

// Kino-Balken (2.39:1) für 16:9, fahren sanft ein
export const Letterbox: React.FC<{ inFrames?: number }> = ({ inFrames = 20 }) => {
  const frame = useCurrentFrame();
  const h = tween(frame, 0, inFrames, 0, 12.2, ease.cinematic); // Prozent der Höhe
  const bar: React.CSSProperties = { position: "absolute", left: 0, right: 0, height: `${h}%`, background: "#050505" };
  return (
    <AbsoluteFill style={{ pointerEvents: "none" }}>
      <div style={{ ...bar, top: 0 }} />
      <div style={{ ...bar, bottom: 0 }} />
    </AbsoluteFill>
  );
};

// Heller Blitz auf einem Musik-Hit (at = Frame des Hits)
export const LightFlash: React.FC<{ at: number; color?: string }> = ({ at, color = "#FFF6E8" }) => {
  const frame = useCurrentFrame();
  const o =
    frame < at ? tween(frame, at - 2, at, 0, 1) : tween(frame, at, at + 8, 1, 0, ease.snap);
  return <AbsoluteFill style={{ background: color, opacity: o, mixBlendMode: "screen" }} />;
};

// Leichte chromatische Aberration: Inhalt dreifach mit Farbversatz rendern
export const Aberration: React.FC<{ children: React.ReactNode; px?: number }> = ({ children, px = 2 }) => (
  <AbsoluteFill>
    <AbsoluteFill style={{ transform: `translateX(${-px}px)`, mixBlendMode: "screen", filter: "sepia(1) saturate(8) hue-rotate(-50deg)", opacity: 0.5 }}>
      {children}
    </AbsoluteFill>
    <AbsoluteFill style={{ transform: `translateX(${px}px)`, mixBlendMode: "screen", filter: "sepia(1) saturate(8) hue-rotate(160deg)", opacity: 0.5 }}>
      {children}
    </AbsoluteFill>
    <AbsoluteFill>{children}</AbsoluteFill>
  </AbsoluteFill>
);
```

Reihenfolge im Root (unten nach oben): Szenen, `Vignette`, `FilmGrain`, `Letterbox`.
`Aberration` nur sparsam einsetzen (z. B. bei Höhepunkt oder Glitch-Moment), sie kostet Renderzeit.

---

## 7. Typografie: Fonts und Text-Reveals

```tsx
// src/cinematic/fonts.ts
import { loadFont as loadDisplay } from "@remotion/google-fonts/BebasNeue";
import { loadFont as loadText } from "@remotion/google-fonts/Inter";

export const display = loadDisplay().fontFamily;
export const text = loadText().fontFamily;
```

```tsx
// src/cinematic/TextReveal.tsx
import React from "react";
import { useCurrentFrame } from "remotion";
import { tween, ease } from "./easing";

// Masken-Reveal: jedes Wort schiebt sich aus einer unsichtbaren Linie hoch
export const MaskReveal: React.FC<{
  text: string;
  start?: number;
  stagger?: number;      // Frames zwischen Wörtern
  style?: React.CSSProperties;
}> = ({ text, start = 0, stagger = 3, style }) => {
  const frame = useCurrentFrame();
  return (
    <div style={{ display: "flex", flexWrap: "wrap", gap: "0.25em", ...style }}>
      {text.split(" ").map((w, i) => {
        const s = start + i * stagger;
        const y = tween(frame, s, s + 18, 110, 0, ease.snap);
        return (
          <span key={i} style={{ overflow: "hidden", display: "inline-block", lineHeight: 1 }}>
            <span style={{ display: "inline-block", transform: `translateY(${y}%)` }}>{w}</span>
          </span>
        );
      })}
    </div>
  );
};

// Blur-zu-Scharf pro Wort, mit leichtem Scale
export const BlurReveal: React.FC<{ text: string; start?: number; stagger?: number; style?: React.CSSProperties }> = ({
  text,
  start = 0,
  stagger = 4,
  style,
}) => {
  const frame = useCurrentFrame();
  return (
    <div style={{ display: "flex", flexWrap: "wrap", gap: "0.3em", ...style }}>
      {text.split(" ").map((w, i) => {
        const s = start + i * stagger;
        const b = tween(frame, s, s + 20, 16, 0);
        const o = tween(frame, s, s + 14, 0, 1);
        const sc = tween(frame, s, s + 24, 1.15, 1);
        return (
          <span key={i} style={{ filter: `blur(${b}px)`, opacity: o, transform: `scale(${sc})`, display: "inline-block" }}>
            {w}
          </span>
        );
      })}
    </div>
  );
};

// Tracking-Animation: Buchstabenabstand zieht sich zusammen (Trailer-Klassiker)
export const TrackingIn: React.FC<{ text: string; start?: number; style?: React.CSSProperties }> = ({
  text,
  start = 0,
  style,
}) => {
  const frame = useCurrentFrame();
  const ls = tween(frame, start, start + 60, 0.8, 0.12, ease.cinematic);
  const o = tween(frame, start, start + 20, 0, 1);
  return <div style={{ letterSpacing: `${ls}em`, opacity: o, textTransform: "uppercase", ...style }}>{text}</div>;
};
```

---

## 8. Übergänge

```tsx
// Szenen mit Übergängen aneinanderreihen
import { TransitionSeries, linearTiming, springTiming } from "@remotion/transitions";
import { fade } from "@remotion/transitions/fade";
import { slide } from "@remotion/transitions/slide";
import { wipe } from "@remotion/transitions/wipe";
import { ease } from "./cinematic/easing";

<TransitionSeries>
  <TransitionSeries.Sequence durationInFrames={75}>
    <SceneHook />
  </TransitionSeries.Sequence>
  <TransitionSeries.Transition
    presentation={wipe({ direction: "from-left" })}
    timing={linearTiming({ durationInFrames: 12, easing: ease.cinematic })}
  />
  <TransitionSeries.Sequence durationInFrames={45}>
    <SceneBuild />
  </TransitionSeries.Sequence>
  <TransitionSeries.Transition
    presentation={slide({ direction: "from-right" })}
    timing={springTiming({ config: { damping: 200 }, durationInFrames: 10 })}
  />
  <TransitionSeries.Sequence durationInFrames={90}>
    <SceneClimax />
  </TransitionSeries.Sequence>
</TransitionSeries>
```

Hard Cut = einfach keinen `Transition` zwischen zwei Sequenzen setzen.

**Zoom-Through** (Kamera fliegt in ein Element hinein): am Ende der alten Szene `CameraRig` mit `zoomTo` 8 bis 20 und `ease.slowIn` auf das Zielelement (`transformOrigin` auf dessen Position), die neue Szene beginnt mit `zoomFrom` 1.3 auf 1.0 und `ease.snap`.

**Whip-Pan:** alte Szene in den letzten 6 Frames per `translateX` um -1.5 Bildbreiten weg, neue Szene in den ersten 6 Frames von +1.5 Bildbreiten herein, beides in `CameraMotionBlur` gewickelt, plus Whoosh-Sound.

---

## 9. Motion Blur

```tsx
import { CameraMotionBlur } from "@remotion/motion-blur";

<CameraMotionBlur shutterAngle={180} samples={10}>
  <SchnelleSzene />
</CameraMotionBlur>
```

Nur um schnelle Szenen legen (Whip-Pan, Fast-Zoom), denn `samples` vervielfacht die Renderzeit. Beim Testen `samples={4}`, final 8 bis 12.

---

## 10. Musik, Beat-Raster und Sound-Effekte

```tsx
// src/cinematic/beat.ts
export const beatFrames = (bpm: number, fps: number) => (fps * 60) / bpm;

// Frame des n-ten Beats (n beginnt bei 0), offset = Frames bis zum ersten Beat in der Musik
export const onBeat = (n: number, bpm: number, fps: number, offset = 0) =>
  Math.round(offset + n * beatFrames(bpm, fps));
```

```tsx
// Audio einbinden (Dateien liegen im Ordner public/)
// Neuere Remotion-Versionen: import { Audio } from "@remotion/media";
// Ältere: import { Audio } from "remotion";
import { Sequence, staticFile, interpolate, useVideoConfig } from "remotion";
import { Audio } from "remotion";

export const Soundtrack: React.FC = () => {
  const { durationInFrames } = useVideoConfig();
  return (
    <>
      <Audio
        src={staticFile("music.mp3")}
        volume={(f) =>
          interpolate(f, [0, 15, durationInFrames - 45, durationInFrames], [0, 1, 1, 0], {
            extrapolateLeft: "clamp",
            extrapolateRight: "clamp",
          })
        }
      />
      <Sequence from={118}><Audio src={staticFile("sfx/whoosh.mp3")} volume={0.6} /></Sequence>
      <Sequence from={150}><Audio src={staticFile("sfx/riser.mp3")} volume={0.5} /></Sequence>
      <Sequence from={210}><Audio src={staticFile("sfx/impact.mp3")} volume={0.7} /></Sequence>
    </>
  );
};
```

Vor dem Einbauen prüfen, welche `Audio` Variante im Projekt existiert (`package.json` und vorhandenen Code ansehen).

---

## 11. Beispiel: eine komplette cinematische Szene

```tsx
import { AbsoluteFill } from "remotion";
import { CameraRig } from "./cinematic/CameraRig";
import { Layer } from "./cinematic/Layer";
import { Particles } from "./cinematic/Particles";
import { LitBackground, GodRays, LightSweep, glow } from "./cinematic/Light";
import { MaskReveal, TrackingIn } from "./cinematic/TextReveal";
import { display, text } from "./cinematic/fonts";

export const SceneHero: React.FC = () => (
  <AbsoluteFill>
    <CameraRig zoomFrom={1.0} zoomTo={1.08} panX={-30} rotate={0.6} shake={1}>
      <Layer depth={0.2} blur={8}>
        <LitBackground base="#0B1A22" light="#FF8A3D" x="72%" y="28%" />
      </Layer>
      <Layer depth={0.4}>
        <GodRays color="#FFD7A8" />
      </Layer>
      <Layer depth={0.6} blur={2}>
        <Particles count={40} color="#FFD7A8" bokeh />
      </Layer>
      <Layer depth={1}>
        <AbsoluteFill style={{ justifyContent: "center", paddingLeft: "10%" }}>
          <TrackingIn
            text="Ab heute"
            start={6}
            style={{ fontFamily: text, color: "#FFD7A8", fontSize: 34, marginBottom: 18 }}
          />
          <div style={{ position: "relative", overflow: "hidden" }}>
            <MaskReveal
              text="ALLES ANDERS"
              start={14}
              style={{ fontFamily: display, fontSize: 220, color: "#F5F2EA", filter: glow("#FF8A3D") }}
            />
            <LightSweep start={40} />
          </div>
        </AbsoluteFill>
      </Layer>
      <Layer depth={1.4} blur={6}>
        <Particles count={12} color="#FFFFFF" />
      </Layer>
    </CameraRig>
  </AbsoluteFill>
);
```

Im Root dann: `TransitionSeries` mit allen Szenen, darüber `Vignette`, `FilmGrain`, optional `Letterbox`, plus `Soundtrack`.
