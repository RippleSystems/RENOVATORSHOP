# Renovator Shop Tiles — root prompt (v1)

This is the one source of truth for every Renovator Lite shop category image: the 32 tiles in `out/`, plus every future subcategory, new category or variation. Change the rules here first, then in `kit.mjs` / `build.mjs`.

## Terms

| Term | Means |
|---|---|
| **Shop Tile** | One category image: 1000 × 1000, pure white, products only, no words. |
| **Hero** | The single product that says the category at a glance (the shingle bundle for Roofing). Back center, biggest. |
| **Supports** | 1–4 smaller products from the category's real subcategories, grouped tight around the Hero. |
| **Tile Layout** | The fixed slot plan: `hero`, `L`, `R`, `L2`, `R2`, `C` (below). |
| **House Label** | The packaging label on every bag, box, can and pail: paper panel, ink band, copper rule, fake-type bars, one pictogram. Never real words, never a brand. |
| **Kit** | The code that draws the products (`kit.mjs` primitives + `products.mjs` products). |

## The rules (every tile, no exceptions)

1. **Canvas**: 1000 × 1000 px, background `#FFFFFF` edge to edge. No border, gradient, floor, room or scene.
2. **No words**: no text, numbers, logos, brand names, watermarks or price tags. The shop UI prints the category name under the tile.
3. **Real products only**: every item is a product a buyer can find in that category's subcategories (`system_material_subcategories_v3`). No people, hands or tools in use.
4. **Grouping (Tile Layout)**: 1 Hero + 1–4 Supports, max 5 items. The Hero stands at back center. Supports overlap the Hero's edge by **35% of their own width** and step forward (`L`/`R` first, then `L2`/`R2` further out and forward; `C` small and front center). Nothing floats or stands apart.
5. **Size by visual area**: the Hero = 1.0; Supports usually 0.3–0.75 of the Hero (sized by area, so long thin and tall items balance).
6. **Fill**: the group fills **86% of the width or 80% of the height**, whichever hits first, centered (the renderer does this; never hand-size a tile).
7. **Camera**: product-catalog 3/4 view. Depth runs up and to the right (oblique: 0.5 across, 0.36 up per unit of depth). Front faces are square to the camera.
8. **Light**: one soft key light from the top left. Tops lightest, fronts mid, right-hand faces darkest. Round bodies get a highlight a quarter in from the left.
9. **Shadow**: one soft contact shadow under each product plus a faint shared shadow under the group. No hard cast shadows, no reflections.
10. **Line**: a thin edge line on every shape, in a darker shade of that shape's own color (constant 2 px on the final image).
11. **Color**: true material colors (lumber, galvanized steel, PVC, copper, concrete, vinyl). Packaging uses the House Label. Avoid colors that are another company's trademark for that product type (pink fiberglass insulation, pink or blue foam board, printed house wrap).
12. **Mobile check**: at 160 px wide the Hero must still say the category. If it doesn't, change the Hero, not the Supports.

## Adding a subcategory, new category or variation (in the Kit)

1. Pull the real names first (Renovator V2):
   ```sql
   select c.name_pro, ms.name as subcategory, ms.slug
   from system_categories_v3 c
   join system_material_subcategories_v3 ms on ms.category_id = c.id and ms.is_active
   where c.is_active and c.name_pro = '<Category>'
   order by ms.sort_order;
   ```
2. Pick the Hero (the product that says it) and 1–4 Supports from those rows.
3. Reuse products from `products.mjs`. If one is missing, draw it there with the primitives in `kit.mjs` (`box`, `cyl`, `endCyl`, `sideCyl`, `bag`, `carton`, `label`). Local coordinates: ground at y = 0, start at x = 0, roughly real proportions.
4. Add an entry to `categories.mjs`:
   ```js
   { slug: 'roofing-shingles', phase: 4, name_pro: 'Shingles', name_client: 'Roof Shingles',
     shows: ['Shingles', 'Starter & Ridge Cap'],
     items: [[p.shingleBundle, 'hero', 1], [() => p.nailBox('screw'), 'R', 0.4]] }
   ```
   For a subcategory tile, the Hero is the subcategory's own product and the Supports are its micro-categories or the parent's closest companions.
5. Render: `node shop/category-images/build.mjs roofing-shingles` (see README for setup) and check the PNG at full size and at 160 px.

**Variations** (seasonal, sale, dark mode, a different aspect ratio): keep rules 2–11 and change only the canvas (rule 1) and fill (rule 6), in `build.mjs`. A dark variation swaps the background only; products, light and shadow stay the same.

## Root prompt for AI image tools

Use this when you want a photographic or AI-made tile that matches the set. Fill the `{…}` slots from the database rows; leave everything else word for word.

```
Product catalog image for an online building-supply store category: {CATEGORY}.
Pure white background (#FFFFFF), edge to edge, no floor, no scene, no border.
One hero product, {HERO}, standing at back center and largest.
Grouped tightly around it, overlapping its edges slightly: {SUPPORT_1}, {SUPPORT_2}, {SUPPORT_3}.
All items are real, unbranded building products. No text, no numbers, no logos, no brand names,
no labels with words, no watermark, no people, no hands.
Three-quarter catalog view from slightly above, depth running up and to the right.
Soft single key light from the top left; tops lightest, right sides darkest.
Soft contact shadow under each product, no hard shadows, no reflections.
Clean vector-illustration style with smooth gradient shading and a thin darker outline on every shape,
true material colors. The group fills about 86% of the width, centered. Square 1:1, 1000 x 1000.
Must read clearly as "{CATEGORY}" on a phone at 160 pixels wide.
```

Negative prompt (if the tool takes one): `text, letters, numbers, logo, brand, trademark, watermark, label words, price tag, people, hands, room, floor, scene, cluttered, cropped products, hard shadow, reflection, pink insulation, pink foam board, blue foam board, printed house wrap`.

For a subcategory, set `{CATEGORY}` to `"<Subcategory> (<Category>)"`, e.g. `"Underlayment (Roofing)"`.

## Copyright and ownership

- **Kit tiles** (`out/`): every shape is drawn from scratch in code in this repo. No stock art, photos, traced images, clip art, fonts, brand names, logos or trade dress are used, and labels carry no words. The repo history shows the authorship. The drawings were created with an AI assistant for the owner. Anthropic's terms assign those outputs to the user. In the US, purely AI-generated material may not be registrable for copyright, while human selection, arrangement and editing can be. If registration matters, have a person make or approve the final edits (arrangement, colors, product choice) and keep the commit history as the record.
- **AI image tool tiles** (made with the prompt above): use a tool whose terms grant you commercial rights to outputs. Never name a brand, a product line or "in the style of" a person or company. Before publishing, check every image for stray logos or lettering. The same registration caveat applies.
- **Never** drop in supplier or manufacturer product photos (Home Depot, Lowe's, brand sites, Firecrawl results). The shop does not own those.

This is practical guidance, not legal advice. For a trademark or registration question, ask an IP attorney.
