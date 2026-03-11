# NEXUS — Schema Explorer

A browser-based, interactive database schema visualization tool. Import your SQL schema as CSV and explore tables, columns, foreign keys, and inferred relationships on a zoomable canvas.

![Single-file HTML application — no build step, no dependencies.](https://img.shields.io/badge/zero%20dependencies-single%20file-blue)

## Features

- **Multi-database support** — Load multiple databases onto the same canvas, each with its own color theme
- **CSV import** — Paste CSV output from a schema query or drag-and-drop a `.csv` file
- **Force-directed layout** — Tables are automatically arranged using a physics simulation that clusters connected tables together
- **Foreign key visualization** — Solid blue lines for declared FKs, dashed orange lines for inferred relationships
- **Inferred relationships** — Automatically detects likely FK relationships based on column naming conventions (e.g. `Ord_Usr_Id` → `Usr_Id`)
- **Interactive canvas** — Pan, zoom, and click to explore; hover for column details
- **Search** — Filter tables and columns in real time
- **Persistent storage** — Databases are saved to IndexedDB and restored on reload
- **Performance-optimized** — Spatial grid culling, LOD rendering, and batched draw calls for smooth interaction even with large schemas
- **Collapsible sidebar** — Browse and manage loaded databases

## Getting Started

1. Open `index.html` in any modern browser.
2. Click **+ Import** and paste CSV output from your schema query, or upload a `.csv` file.
3. Explore the canvas — scroll to zoom, drag to pan, click a column to highlight its relationships.

### Expected CSV Format

The CSV should have these columns (header names are flexible):

```
schema_name, table_name, column_order, column_name, data_type, max_length, precision, scale, is_nullable, is_identity, fk_name, ref_schema, ref_table, ref_column
```

#### MSSQL query to get this for the entire database:
```
SELECT 
    s.name                          AS schema_name,
    t.name                          AS table_name,
    c.column_id                     AS column_order,
    c.name                          AS column_name,
    tp.name                         AS data_type,
    c.max_length,
    c.precision,
    c.scale,
    c.is_nullable,
    c.is_identity,
    -- FK info
    fk.name                         AS fk_name,
    rs.name                         AS ref_schema,
    rt.name                         AS ref_table,
    rc.name                         AS ref_column
FROM sys.columns c
JOIN sys.tables t                   ON c.object_id = t.object_id
JOIN sys.schemas s                  ON t.schema_id = s.schema_id
JOIN sys.types tp                   ON c.user_type_id = tp.user_type_id
-- FK joins (all LEFT so columns without FKs still show)
LEFT JOIN sys.foreign_key_columns fkc  ON fkc.parent_object_id = c.object_id
                                       AND fkc.parent_column_id = c.column_id
LEFT JOIN sys.foreign_keys fk          ON fkc.constraint_object_id = fk.object_id
LEFT JOIN sys.columns rc               ON fkc.referenced_object_id = rc.object_id
                                       AND fkc.referenced_column_id = rc.column_id
LEFT JOIN sys.tables rt                ON fkc.referenced_object_id = rt.object_id
LEFT JOIN sys.schemas rs               ON rt.schema_id = rs.schema_id
ORDER BY 
    s.name, 
    t.name, 
    c.column_id;
```

## Controls

| Action | Input |
|---|---|
| Pan | Click and drag on canvas |
| Zoom | Scroll wheel, or `+` / `−` buttons |
| Fit view | **Fit** button |
| Search | Type in the search box |
| Select column | Click a column row |
| Toggle sidebar | `◀` button |

## Tech

Single self-contained HTML file — no frameworks, no build tools, no server required. Uses:

- **Canvas 2D** for rendering
- **IndexedDB** for persistence
- **Force-directed graph layout** for automatic table positioning

## License

MIT
