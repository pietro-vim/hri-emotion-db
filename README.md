# HRI Emotion Database — Relational Database for Human-Robot Emotional Interaction

A relational database designed to model emotional interactions between humans and
intelligent systems (Human-Robot Interaction). Developed as a project for the
_Databases and Information Systems_ course at Università del Piemonte Orientale
"Amedeo Avogadro", it covers the full design pipeline — conceptual, logical, and physical — for a domain focused on
emotionally-aware human-machine dialogue.

## Overview

The system models a platform where users interact with intelligent agents (robots)
through multiple channels, and where each interaction is analysed for the emotions it
expresses. Detected emotions are based on **Plutchik's model** (eight primary emotions
and their secondary combinations) and drive how the system responds, adapts its tone,
and personalises the experience for each user.

This is an **academic database design project**: the deliverable is the database itself
(schema, sample data, and a set of representative operations), together with the full
design report. It is not a deployed application — there is no application or ML layer on
top of the schema.

## What this project demonstrates

- Conceptual modelling with an Entity-Relationship diagram (both the initial and the
  restructured version)
- Logical design: translation to the relational model, restructuring decisions, and
  choice of primary keys
- Normalisation up to **Third Normal Form (3NF)**
- Physical implementation in PostgreSQL with referential integrity, `CHECK`
  constraints, and cascade rules
- Handling of N:N relationships through bridge tables
- Attribute _reification_ as a recurring design pattern (e.g. modelling `emozione` and
  `metrica` as entities rather than plain attributes)

## Tech stack

- **PostgreSQL 17.5**
- SQL (DDL + DML)
- E-R modelling

## Repository structure

| File                               | Description                                                                                                        |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `DDL.sql`                          | Schema definition: 21 tables with primary keys, foreign keys, `CHECK` and cascade constraints                      |
| `DMLPOP.sql`                       | Population script with realistic sample data for every table                                                       |
| `DMLUPD.sql`                       | 22 representative modification operations (insert / update / delete) mapped to the system's use cases              |
| `Schema_E-R_non_ristrutturato.png` | Initial (non-restructured) E-R diagram                                                                             |
| `Schema_E-R_ristrutturato.png`     | Restructured E-R diagram used for the logical design                                                               |
| `Relazione.pdf`                    | Full design report (requirements, glossary, E-R schemas, business rules, logical and physical design) — in Italian |

## Database schema

The database is made up of **21 tables**, which can be grouped as follows:

**Core entities**
`utente`, `robot`, `sessione`, `interazione`, `emozione`, `messaggio`

**Configuration & profile**
`profilo_utente`, `configurazione_robot`, `accessibilita`, `config_privacy`

**Supporting / dynamic entities**
`feedback`, `contenuto_multimediale`, `metrica`, `log_errori`, `storico_emotivo`,
`traduzione`, `interfaccia_canale`, `policy_moderazione`

**Bridge tables (N:N)**
`utente_interfaccia`, `interazione_interfaccia`, `storico_emotivo_emozione`

### Selected design choices

- **Surrogate keys** for every entity, for stability and simpler joins.
- **Plutchik-based emotions**: only primary and secondary emotions are accepted, and
  emotion intensity is constrained to the range `0.00`–`1.00`.
- **`CHECK` constraints** enforce valid value sets at the database level (e.g. robot
  operational state, message tone and type, accessibility text sizes).
- **`ON DELETE CASCADE` / `ON UPDATE CASCADE`** keep referential integrity consistent
  when users or related records are removed.
- **1:N between `utente` and `profilo_utente`**, a deliberate choice that lets a user
  hold multiple evolving psychological profiles over time.

## Design process

The schema was built with a **mixed top-down / bottom-up** approach. The top-down phase
identified the core entities of the domain (user, robot, interaction, emotion) and their
cardinalities; the bottom-up phase refined each entity with its specific attributes and
introduced entities such as `log_errori` and `policy_moderazione` driven by monitoring
and safety requirements.

During restructuring, three merges were applied to simplify the model:

- `Risposta` + `Suggerimento` → `messaggio` (with a `tipo_messaggio` discriminator)
- `Preferenze_Utente` + `Profilo_Utente` → `profilo_utente`
- `Competenza_Robot` + `Personalizzazione_Robot` → `configurazione_robot`

The direct `utente`–`interazione` relationship was also removed, since it is already
captured through `sessione`.

Full details, including all business rules and derivations, are in `Relazione.pdf`.

## Getting started

Requires PostgreSQL (developed on 17.5). The SQL files are PostgreSQL dumps.

```bash
# 1. Create a database
createdb hri_emotion_db

# 2. Load the schema
psql -d hri_emotion_db -f DDL.sql

# 3. Populate with sample data
psql -d hri_emotion_db -f DMLPOP.sql

# 4. (Optional) run the example modification operations
psql -d hri_emotion_db -f DMLUPD.sql
```

Once loaded, you can explore the data, for example:

```sql
-- Emotions detected in interactions, with their intensity
SELECT e.nome, e.intensita, e.categoria_plutchik
FROM emozione e
JOIN interazione i ON e.id_interazione = i.id_interazione
ORDER BY e.intensita DESC;
```

## Authors

- **Pietro Mezzatesta**
- **Ossama Boulakhdar**

## Academic context

Course: _Basi di dati e Sistemi Informativi_ (Databases and Information Systems)
Università del Piemonte Orientale "Amedeo Avogadro" — Academic Year 2024/2025
