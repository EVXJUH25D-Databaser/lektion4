---
author: Lektion 4 - Relationer och joins
date: MMMM dd, YYYY
paging: "%d / %d"
---

# Lektion 4

## Dagens agenda

1. Frågor och repetition
2. Introduktion till relationer
3. Introduktion till joins
4. Gruppövning
5. Eget arbete med handledning

---

# Hur sparas listor av saker i SQL-databaser?

En kolumn kan endast spara ett värde och inte flera som en array. Istället hanteras listor av saker med "relationer".

En relation bildas när en rad i en tabell har en länk till en annan rad, oftast i en annan tabell.

---

# Exempel relation: Människor <-> Husdjur

```md
humans:

| name   | age |
| ------ | --- |
| Bob    | 30  |
| Rachel | 27  |

pets:

| name    | owner  |
| ------- | ------ |
| Charlie | Bob    |
| Oskar   | Bob    |
| Nevad   | Rachel |
```

---

# Primary keys och foreign keys

En relation är en länk mellan två rader. För att skapa en länk används primary keys och foreign keys.

- **Primary key**: en identifierande kolumn. Den är självstående och är typiskt sätt en `SERIAL` eller `UUID`.
- **Foreign key**: en refererande kolumn. Refererar till en primary key.
- Foreign keys har alltid samma värde som en primary key
- Primary keys är unika

I exemplet med människor och husdjur är `humans:name` en primary key och `pets:owner` en foreign key.

---

# Typer av relationer

## One-to-One

En rad refererar till exakt en rad. Förekommer sällan.

## One-to-Many och Many-to-One

En rad refererar till flera (0-n) rader. One-to-Many och Many-to-One är samma sak från olika perspektiv.

Exempel: människor <-> husdjur, användare <-> kommentarer

## Many-to-Many

Flera rader refererar till flera rader. Om One-to-Many går åt båda hållen så är det Many-to-Many. Skapas med "junction" tabeller.

Exempel: användare <-> git repos, spelare <-> spel

---

# Many-to-Many exempel: Spelare <-> Spel

```md
players:
| id | name    |
| -- | ------- |
| 1  | Alice   |
| 2  | Bob     |
| 3  | Charlie |

games:
| id | title      |
| -- | ---------- |
| 1  | Minecraft  |
| 2  | Fortnite   |

player_games (junction table):
| player_id | game_id |
| --------- | ------- |
| 1         | 1       |
| 1         | 2       |
| 2         | 1       |
| 3         | 2       |
```

Alice spelar både Minecraft och Fortnite.
Minecraft spelas av både Alice och Bob.

---

# Skapa relationer i PostgreSQL

```sql
CREATE TABLE humans (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    age INTEGER
);

CREATE TABLE pets (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    owner_id INTEGER REFERENCES humans(id)
);
```

`REFERENCES` skapar en foreign key constraint som säkerställer att `owner_id` måste matcha ett existerande `humans.id`.

---

# Vad är en JOIN?

En JOIN kombinerar data från flera tabeller baserat på en relation.

Utan JOIN:
```sql
SELECT * FROM pets WHERE owner_id = 1;
```

Visar bara husdjur, inte ägarens info.

Med JOIN:
```sql
SELECT * FROM pets
JOIN humans ON pets.owner_id = humans.id;
```

Kombinerar data från både pets OCH humans i resultatet.

---

# INNER JOIN

Den vanligaste typen av join. Returnerar endast rader där det finns matchningar i BÅDA tabellerna.

```sql
SELECT humans.name, pets.name AS pet_name
FROM humans
INNER JOIN pets ON pets.owner_id = humans.id;
```

Resultat:
```md
| name   | pet_name |
| ------ | -------- |
| Bob    | Charlie  |
| Bob    | Oskar    |
| Rachel | Nevad    |
```

---

# LEFT JOIN

Returnerar ALLA rader från vänster tabell, även om det inte finns matchning i höger tabell.

```sql
SELECT humans.name, pets.name AS pet_name
FROM humans
LEFT JOIN pets ON pets.owner_id = humans.id;
```

Om Alice finns i humans men inte har husdjur:
```md
| name   | pet_name |
| ------ | -------- |
| Bob    | Charlie  |
| Bob    | Oskar    |
| Rachel | Nevad    |
| Alice  | NULL     |
```

---
# RIGHT JOIN och FULL JOIN

**RIGHT JOIN**: Alla rader från höger tabell, även utan matchning i vänster.

**FULL JOIN**: Alla rader från BÅDA tabellerna, även utan matchningar.

```sql
-- RIGHT JOIN
SELECT * FROM pets
RIGHT JOIN humans ON pets.owner_id = humans.id;

-- FULL JOIN
SELECT * FROM pets
FULL JOIN humans ON pets.owner_id = humans.id;
```

RIGHT JOIN används sällan (använd LEFT JOIN istället genom att byta ordning på tabellerna).

---

# Many-to-Many med junction-tabell

För att skapa en Many-to-Many relation behövs en mellanliggande tabell (junction table).

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE repos (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE user_repos (
    user_id INTEGER REFERENCES users(id),
    repo_id INTEGER REFERENCES repos(id),
    PRIMARY KEY (user_id, repo_id)
);
```

---

# JOIN med junction-tabell

För att hämta alla repos en användare har tillgång till:

```sql
SELECT users.name, repos.name AS repo_name
FROM users
JOIN user_repos ON users.id = user_repos.user_id
JOIN repos ON user_repos.repo_id = repos.id
WHERE users.name = 'Bob';
```

Man kan kedja flera JOINs för att navigera genom relationer.


---

# Sammanfattning

- Relationer länkar samman rader mellan tabeller
- Primary key = unik identifierare
- Foreign key = referens till primary key
- JOIN kombinerar data från flera tabeller
- INNER JOIN = endast matchningar
- LEFT JOIN = alla från vänster + matchningar
- Many-to-Many = kräver junction-tabell

---

# Gruppövning: Relationer och Joins i PostgreSQL

## Scenario: Musikstreamingtjänst

Ni ska arbeta med en databas för en musikstreamingtjänst liknande Spotify. Databasen innehåller information om artister, album, låtar och användarnas spellistor.

## Del 1: Skapa databasen (15 minuter)

### Steg 1: Skapa tabellerna

```sql
-- Tabell för artister
CREATE TABLE artists (
    artist_id SERIAL PRIMARY KEY,
    artist_name VARCHAR(100) NOT NULL,
    country VARCHAR(50),
    formed_year INTEGER
);

-- Tabell för album
CREATE TABLE albums (
    album_id SERIAL PRIMARY KEY,
    album_title VARCHAR(150) NOT NULL,
    artist_id INTEGER REFERENCES artists(artist_id),
    release_year INTEGER,
    genre VARCHAR(50)
);

-- Tabell för låtar
CREATE TABLE songs (
    song_id SERIAL PRIMARY KEY,
    song_title VARCHAR(150) NOT NULL,
    album_id INTEGER REFERENCES albums(album_id),
    duration_seconds INTEGER,
    track_number INTEGER
);

-- Tabell för användare
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100),
    created_at DATE DEFAULT CURRENT_DATE
);

-- Tabell för spellistor (många-till-många relation)
CREATE TABLE playlists (
    playlist_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id),
    playlist_name VARCHAR(100) NOT NULL,
    created_at DATE DEFAULT CURRENT_DATE
);

-- Kopplingsstabell mellan spellistor och låtar
CREATE TABLE playlist_songs (
    playlist_id INTEGER REFERENCES playlists(playlist_id),
    song_id INTEGER REFERENCES songs(song_id),
    added_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (playlist_id, song_id)
);
```

### Steg 2: Fyll i testdata

_Fyll gärna på mer ännu fler värden om ni vill._

```sql
-- Lägg till artister
INSERT INTO artists (artist_name, country, formed_year) VALUES
('ABBA', 'Sweden', 1972),
('Roxette', 'Sweden', 1986),
('The Beatles', 'UK', 1960),
('Avicii', 'Sweden', 2008);

-- Lägg till album
INSERT INTO albums (album_title, artist_id, release_year, genre) VALUES
('Arrival', 1, 1976, 'Pop'),
('The Visitors', 1, 1981, 'Pop'),
('Look Sharp!', 2, 1988, 'Pop Rock'),
('Abbey Road', 3, 1969, 'Rock'),
('True', 4, 2013, 'EDM');

-- Lägg till låtar
INSERT INTO songs (song_title, album_id, duration_seconds, track_number) VALUES
('Dancing Queen', 1, 230, 2),
('Money, Money, Money', 1, 185, 3),
('One of Us', 2, 245, 1),
('The Look', 3, 236, 1),
('Listen to Your Heart', 3, 318, 4),
('Come Together', 4, 259, 1),
('Something', 4, 182, 2),
('Wake Me Up', 5, 247, 1),
('Hey Brother', 5, 255, 2);

-- Lägg till användare
INSERT INTO users (username, email) VALUES
('anna_svensson', 'anna@example.com'),
('erik_karlsson', 'erik@example.com'),
('maria_andersson', 'maria@example.com');

-- Lägg till spellistor
INSERT INTO playlists (user_id, playlist_name) VALUES
(1, 'Svenska favoriter'),
(1, 'Klassiker'),
(2, 'Träningslista'),
(3, 'Chill');

-- Lägg till låtar i spellistor
INSERT INTO playlist_songs (playlist_id, song_id) VALUES
(1, 1), (1, 2), (1, 4), (1, 8),
(2, 1), (2, 6), (2, 7),
(3, 8), (3, 9),
(4, 5), (4, 7);
```

## Del 2: Övningsuppgifter (45 minuter)

Lös följande övningar i er grupp. Diskutera vilken typ av join som passar bäst för varje fråga.

### Grundläggande Joins

**1. INNER JOIN: Lista alla låtar med deras albumnamn och artistnamn**
```sql
-- Skriv din query här
```

**2. LEFT JOIN: Visa alla album, inklusive de som inte har några låtar**
```sql
-- Skriv din query här
```

**3. INNER JOIN: Hitta alla låtar som är längre än 4 minuter med artist- och albuminfo**
```sql
-- Skriv din query här
```

### Mellansvåra Joins

**4. Multiple Joins: Lista alla spellistor med användarnamn och antal låtar i varje spellista**
```sql
-- Tips: Använd COUNT och GROUP BY
```

**5. Hitta alla svenska artister och deras låtar**
```sql
-- Skriv din query här
```

**6. Visa alla användare och deras spellistor (även användare utan spellistor)**
```sql
-- Vilken join behövs här?
```

### Avancerade Joins

**7. Många-till-många: Lista alla låtar i 'Svenska favoriter'-spellistan med fullständig information**
```sql
-- Behöver flera joins genom kopplingstabellen
```

**8. Aggregering: Visa varje artists totala antal låtar och genomsnittlig låtlängd**
```sql
-- Använd COUNT, AVG och GROUP BY
```

**9. Subquery och Join: Hitta alla album som har fler än 2 låtar**
```sql
-- Tips: Kan lösas med HAVING eller subquery
```

**10. Complex Join: Visa vilka artister som förekommer i flest spellistor**
```sql
-- Behöver joins genom flera tabeller och COUNT
```

## Del 3: Diskussionsfrågor (15 minuter)

Diskutera följande i er grupp:

1. **Vad är skillnaden mellan INNER JOIN och LEFT JOIN?** När ska man använda vilken?

2. **Varför behövs kopplingstabellen `playlist_songs`?** Vad händer om vi försöker skapa en direkt relation mellan playlists och songs?

3. **Foreign keys och integritet:** Vad händer om vi försöker ta bort en artist som har album?

4. **Performance:** Hur skulle indexes kunna förbättra prestandan för dessa queries?
