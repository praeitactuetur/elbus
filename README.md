# On és el bus? 🚌

PWA col·laborativa i anònima per seguir el bus Igualada ↔ Barcelona (Monbus / Hispano Igualadina).

## Com funciona

- **Estàs al bus?** → Selecciona la direcció i comença a compartir. La teva ubicació s'envia de forma anònima cada 20 segons.
- **Estàs esperant?** → Selecciona la direcció i la teva parada. L'app et dirà si el bus s'acosta i en quants minuts arribarà, o si ja ha passat.
- **Xat** → Parla amb altres usuaris de la línia en temps real.

## Privacitat

- Completament anònim. No es guarda cap dada personal.
- Cada sessió genera un ID aleatori que desapareix en tancar el navegador.
- Les ubicacions s'esborren automàticament als 10 minuts.

## Desplegament a Netlify

1. Connecta el repositori a Netlify.
2. Configura el directori de publicació com `.` (arrel).
3. Desplega. El `netlify.toml` ja ho configura tot.

## Base de dades (Supabase)

Taules necessàries:

```sql
-- Ubicacions dels busos
create table bus_checkins (
  id text primary key,           -- session ID anònim
  linea_bus text not null,
  lat float8 not null,
  lon float8 not null,
  sentit text not null,          -- 'BCN' o 'IGA'
  created_at timestamptz default now()
);

-- Xat
create table chat_messages (
  id bigserial primary key,
  linea_bus text not null,
  sentido text not null,
  apodo text not null,
  mensaje text not null,
  created_at timestamptz default now()
);

-- Habilita Realtime per al xat
alter publication supabase_realtime add table chat_messages;
```

Row Level Security (RLS) recomanada per `bus_checkins`:
```sql
alter table bus_checkins enable row level security;
create policy "Lectura pública" on bus_checkins for select using (true);
create policy "Escriptura anònima" on bus_checkins for insert with check (true);
create policy "Actualització pròpia" on bus_checkins for update using (true);
create policy "Eliminació pròpia" on bus_checkins for delete using (true);
```
