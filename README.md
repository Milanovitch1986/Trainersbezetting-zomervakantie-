# Trainersbezetting zomervakantie

Een eenvoudige webapp om de trainersbezetting te regelen tijdens de zomervakantie
(regio zuid). Trainers geven zelf hun beschikbaarheid door; de coördinator maakt
daarna met één klik een eerlijke planning voor alle maandagen en woensdagen.

Gebouwd als één `index.html`-bestand, gehost op GitHub Pages, met [Supabase](https://supabase.com)
als database.

## Links

- **Voor trainers (beschikbaarheid invullen):**
  https://milanovitch1986.github.io/Trainersbezetting-zomervakantie-/
- **Voor de coördinator (planning maken):**
  https://milanovitch1986.github.io/Trainersbezetting-zomervakantie-/#planning
  (met wachtwoord)

## Wat de app doet

- Trainers vullen hun **naam** in en kiezen hun **categorie** (U14 of U16).
- Per training (elke maandag en woensdag van de vakantie) kiezen ze:
  - ✅ **beschikbaar** — ik kan
  - ⚠️ **alleen in nood** — liever niet, maar het kan
  - ❌ **niet** — ik kan niet
- De coördinator genereert een planning waarbij elke training **minimaal 2 U14 en
  2 U16** trainers heeft.

## Hoe de planning werkt

- De trainingen worden **eerlijk verdeeld**: wie tot dan toe het minst is ingezet,
  komt als eerste aan de beurt.
- "Alleen in nood" wordt pas gebruikt als het met de gewoon-beschikbare trainers
  niet lukt (die krijgen dan een ⚠).
- Lukt het minimum zelfs dan niet, dan wordt de training **rood** met "tekort".
- De **middelste 2 weken** van de vakantie worden geel gemarkeerd; dat zijn de
  krappe weken waar het minimum gegarandeerd moet worden. Bij de andere weken kun
  je met één tik extra beschikbare trainers toevoegen.
- De planning is te **printen / als PDF op te slaan** en met één knop te **kopiëren
  voor de WhatsApp-groep**.

De officiële zomervakanties van regio zuid (t/m 2030) zitten ingebouwd, dus de
coördinator kiest alleen het jaar en klikt op "Vul automatisch in".

## Voor de coördinator

1. Open de coördinatorlink (met `#planning`) en log in met het wachtwoord.
2. **Stap 1 – Periode:** kies het jaar, klik "Vul automatisch in", controleer de
   datums en klik "Periode opslaan".
3. Deel de gewone link met de trainers zodat zij hun beschikbaarheid kunnen invullen.
4. **Stap 3 – Planning:** als iedereen heeft ingevuld, klik op "Genereer planning".
   Pas eventueel handmatig aan en print of kopieer het resultaat.

---

## Technische setup (voor onderhoud)

### 1. Database (Supabase)

Draai in de Supabase **SQL Editor** eenmalig deze query om de tabellen aan te maken:

```sql
create table trainers (
  id uuid primary key default gen_random_uuid(),
  naam text not null,
  categorie text not null check (categorie in ('U14','U16')),
  beschikbaarheid jsonb not null default '{}',
  updated_at timestamptz default now()
);

create table instellingen (
  id int primary key default 1 check (id = 1),
  start_datum date,
  eind_datum date
);

-- iedereen met de link mag lezen/schrijven (club-tool, geen accounts)
alter table trainers enable row level security;
alter table instellingen enable row level security;
create policy "open trainers" on trainers for all using (true) with check (true);
create policy "open instellingen" on instellingen for all using (true) with check (true);
```

### 2. Configuratie

Bovenin `index.html` staan drie regels die je invult:

```js
const SUPABASE_URL           = "https://xxxxxxxx.supabase.co";  // Project URL (Data API)
const SUPABASE_ANON_KEY      = "sb_publishable_...";            // publishable / anon key (API Keys)
const COORDINATOR_WACHTWOORD = "kies-iets-eenvoudigs";          // zelf te verzinnen
```

De `SUPABASE_URL` moet **exact** op `.supabase.co` eindigen — geen schuine streep of
extra pad erachter.

### 3. Hosten (GitHub Pages)

Repo-instellingen → **Settings → Pages** → Source: *Deploy from a branch* →
Branch: `main`, map: `/ (root)` → Save. Na ~1 minuut staat de site live.

## Onderhoud

- **Nieuwe vakantiedatums:** de officiële datums van regio zuid zijn vastgesteld
  t/m 2030 en zitten in de lijst `ZOMERVAKANTIE_ZUID` in `index.html`. Voor latere
  jaren voeg je daar een regel toe zodra de overheid ze bekendmaakt.
- **Beveiliging:** het coördinatorwachtwoord staat in de code en is dus geen echte
  beveiliging, maar een drempel. De database staat open voor iedereen met de link
  en de anon key — prima voor een clubtool, maar geen plek voor gevoelige gegevens.

---

*Gemaakt voor AV Sprint Breda.*
