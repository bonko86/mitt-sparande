# Designsystem

## Arkitektur

Design tokens → återanvändbara UI-komponenter → funktionskomponenter → sidor.

- `src/styles/`: gemensam visuell grund.
- `src/components/ui/`: generiska byggblock utan kunskap om sparande.
- `src/features/`: framtida domänspecifika komponenter och logik.
- `src/pages/`: kompletta vyer som komponerar funktioner och UI.

Mapparna innehåller tills vidare endast `.gitkeep`. Inga produktfunktioner,
sidor, navigering eller komponentbibliotek har införts.

## SCSS och tokens

`src/styles/index.scss` laddas en gång från `src/main.tsx`.
Den laddar tokens, minimal reset och globala standardvärden i den ordningen.
Sass är ett utvecklingsberoende; Vite kompilerar SCSS till CSS.

| Fil i styles/tokens | Ansvar |
| --- | --- |
| _colors.scss | Primitiva färger och semantiska färgroller |
| _typography.scss | Typsnitt, storlekar, vikter och radhöjder |
| _spacing.scss | Återanvändbar avståndsskala |
| _radius.scss | Återanvändbara hörnradier |
| _shadows.scss | Återanvändbara skuggor |
| _breakpoints.scss | Sass-värden för responsiva brytpunkter |
| _index.scss | Samlar kategorier med @forward |

`_reset.scss` hanterar box-sizing, marginal på body, ärvda typsnitt och
textfärger i formulärkontroller samt flexibla bilder.
`_global.scss` använder tokens för grundtypografi, bakgrund, text och fokus.

Primitiva tokens beskriver värden, exempelvis `--gray-900`.
Semantiska tokens beskriver syftet:
`--color-text-primary: var(--gray-900)`.
Komponenter väljer normalt den semantiska rollen.
Färgerna är neutrala platshållare; statusfärger anger enbart funktion.
Skalor för avstånd och storlekar behöver inte dubbleras med meningslösa alias.

Återanvändbara tokens exponeras som CSS custom properties. De kan ändras vid
körning och ärvs genom DOM-trädet. SCSS används för filorganisation,
komponentstilar och brytpunkter, inte som ersättning för runtime-tokens.

Bra:
```scss
.example {
  background: var(--color-primary);
  color: var(--color-on-primary);
  padding: var(--spacing-md);
  border-radius: var(--radius-md);
}

.example-text {
  color: var(--color-text-primary);
}
```

Undvik att duplicera gemensamma designbeslut:
```scss
.example {
  background: #536dfe;
  color: #171717;
  padding: 16px;
  border-radius: 8px;
}
```

## Nya komponenter

Lägg generiska komponenter i egna mappar:
```text
components/ui/RoundButton/
  RoundButton.tsx
  RoundButton.scss
```

TSX ansvarar för API, beteende och struktur och importerar komponentens SCSS.
SCSS ansvarar för utseendet. Använd unika klassnamn, exempelvis
`.round-button`, eftersom vanliga SCSS-importer inte isolerar selektorer.

Ett framtida API kan användas så här:
```tsx
<RoundButton onClick={handleAdd} aria-label="Lägg till sparande">
  +
</RoundButton>
```

Komponenten äger standardutseendet. Exponera vanliga HTML-egenskaper,
tillgängliga namn, tangentbordsbeteende och endast motiverade varianter.
En vanlig åtgärdsknapp bör ha `type="button"` som standard.
Kräv inte att varje användning skickar färg, radie, bredd eller padding.

`RoundButton`, `Card` och `ProgressBar` hör hemma i UI.
En framtida `features/goals/SavingsGoalCard` kan kombinera dessa och känna
till sparmål. Sidor komponerar funktioner och UI. UI ska inte importera
domänlogik eller sidor. Föredra komposition framför stora monolitiska komponenter.

## Mobile-first och brytpunkter

Skriv grundstilar för mobil. Lägg till större layouter med min-width.
Alla nya brytpunkter definieras i `_breakpoints.scss`:
sm = 40rem, md = 48rem och lg = 64rem.

Importera enbart brytpunktsmodulen i komponentstilar; att importera hela
tokenindexet där skulle kunna duplicera global CSS.
Exempel från `components/ui/Example/Example.scss`:
```scss
@use '../../../styles/tokens/breakpoints' as breakpoints;

.example {
  display: grid;
  gap: var(--spacing-md);

  @media (min-width: breakpoints.$md) {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}
```

Brytpunkter är Sass-variabler eftersom CSS custom properties inte kan användas
som villkor i media queries. Inga nya app-layouter eller Capacitor har införts.

## Regler för nya tokens och stilar

1. Använd befintliga tokens för återanvändbara färger, avstånd och typografi.
2. Föredra semantiska färgroller framför primitiva färger.
3. Skapa nya tokens endast för gemensamma designbeslut och i rätt kategorifil.
4. Håll komponentens stilar intill komponenten.
5. Duplicera inte typografiska definitioner.
6. Lägg inte till stylingramverk eller komponentbibliotek utan uttryckligt skäl.

Engångsvärden som beskriver en komponentimplementation får vara lokala.
En avsiktligt 56 × 56 px stor RoundButton kan ha `width: 56px` och
`height: 56px`, samt `border-radius: 50%`, i sin egen SCSS.
Skapa inte globala `--round-button-width` eller `--round-button-height`
utan ett verkligt behov av gemensam konfiguration.

## Framtida teman

Samla framtida temaöverskrivningar i färgmodulen och ändra semantiska roller,
så att komponenternas CSS fortsätter fungera. Exempel, inte implementerat:
```scss
[data-theme='dark'] {
  --color-background: var(--gray-900);
  --color-surface: var(--gray-800);
  --color-text-primary: var(--gray-50);
  --color-text-secondary: var(--gray-300);
  --color-border: var(--gray-600);
  --color-primary: var(--gray-100);
  --color-primary-hover: var(--gray-300);
  --color-on-primary: var(--gray-900);
  --color-focus: var(--gray-100);
}
```

När ett tema byggs behöver samtliga roller och kontraster kontrolleras.
Ingen temaväljare eller komplett mörkt tema byggs i denna grund.

## Befintlig startsida

`src/index.css` och `src/App.css` behålls för den befintliga Vite-startsidan.
`main.tsx` laddar den nya grunden före startsidans gamla CSS, så att dess
befintliga design och färgscheman behålls. Äldre färger och brytpunkter där
är inte förebilder för nya komponenter. Nya komponenter använder denna
tokenarkitektur. Startsidan migreras först när en sådan ändring beställs.

## Verifiering

Kör `npm run lint` och `npm run build`.
Kontrollera att byggd CSS innehåller exempelvis `--color-primary` och
`--spacing-md`, och att HTML-filen refererar den genererade stilmallen.
