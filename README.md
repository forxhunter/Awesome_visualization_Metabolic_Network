# Escher Maps for BiGG Models

Automatically generated, Escher-compatible metabolic maps for the models in the
[BiGG Models database](http://bigg.ucsd.edu/) — one map per functional cluster, plus a
whole-model map for each reconstruction.

Every map in this repository was produced by **MetaCarto**, a layout engine that draws a
metabolic network the way a curator would rather than the way a force-directed algorithm does.
**MetaCarto will be released soon**; this repository is its output, published ahead of the code.

---

## Citation

**If you use or modify these maps — in a paper, a figure, a talk, a poster, a database, or
derived software — you must cite this repository.**

```bibtex
@misc{wu_escher_maps_bigg,
  author       = {Wu, Tianyu},
  title        = {Escher Maps for BiGG Models: automatically generated metabolic
                  pathway maps},
  year         = {2026},
  howpublished = {\url{https://github.com/forxhunter/escher_maps_BiGG}},
  note         = {Generated with MetaCarto}
}
```

Plain text:

> Wu, T. *Escher Maps for BiGG Models: automatically generated metabolic pathway maps.*
> https://github.com/forxhunter/escher_maps_BiGG (generated with MetaCarto).

This applies to modified maps as well: if you edit a map in Escher and publish the result, the
layout is still derived from this work. A MetaCarto citation will be added here on release —
please use both from that point on.

Please also cite the underlying model from
[BiGG Models](http://bigg.ucsd.edu/) and, where the maps are displayed,
[Escher](https://doi.org/10.1371/journal.pcbi.1004321).

---

## Using the maps

### In the browser

**[→ Open the Escher viewer](https://forxhunter.github.io/escher/)**

- **Map ▸ Load map from library…** browses this repository directly: pick a model, then a
  pathway. Nothing to download.
- **Map ▸ Load map JSON** opens a file you have saved locally.

### In your own code

The files are plain [Escher](https://escher.github.io/) JSON and load anywhere Escher runs —
the Python package, the Jupyter widget, or an embedded `escher.Builder`.

```python
import escher, json, urllib.request

url = ('https://raw.githubusercontent.com/forxhunter/escher_maps_BiGG/main/'
       'e_coli_core/Glycolysis_Gluconeogenesis.json')
with urllib.request.urlopen(url) as response:
    map_json = json.load(response)

escher.Builder(map_json=json.dumps(map_json)).display_in_notebook()
```

---

## Repository structure

```
map_index.json                     every model, with map counts
{Model_ID}/
    model_index.json               every map in this model, with sizes
    {Model_ID}_Combined.json       whole model, pathways tiled and captioned
    {Cluster}.json                 one functional cluster
```

Cluster names come from the model's own `subsystem` annotation where it has one
(`Glycolysis_Gluconeogenesis`, `Citric_Acid_Cycle`, …). Many BiGG reconstructions carry no
subsystem annotation at all; for those, clusters are inferred from network structure and named
`Cluster_N`.

The two index files exist so applications can list the collection without cloning it. They are
regenerated together with the maps.

---

## How the maps are made

MetaCarto is constructive and deterministic — no annealing, no random seed. The same model
always produces the same map. Five ideas do most of the work:

1. **Primary-compound reduction.** A reaction such as
   `pyruvate + CoA + NAD⁺ → acetyl-CoA + CO₂ + NADH` becomes *one* directed edge, between the
   substrate/product pair that shares the most molecular skeleton — here pyruvate → acetyl-CoA.
   Everything else is drawn as a side branch. Without this reduction every reaction node has
   degree 4–8 and no clean orthogonal drawing exists, which is why naive layouts of metabolic
   networks come out as hairballs.

2. **Direction from flux.** Reconstructions store reversible reactions in whichever direction
   the curator happened to write them, so glycolysis is often recorded partly backwards. Parsimonious
   FBA decides the drawn direction wherever a reaction carries flux.

3. **Cycles drawn as cycles.** The TCA cycle, the urea cycle and the Calvin cycle are detected
   and placed on a ring, rotated so the ring's entry arc faces the pathway feeding it.

4. **Layered drawing.** A Sugiyama layering with Brandes–Köpf coordinate assignment gives the
   vertical flow and the straight pathway backbones. Edges are routed orthogonally.

5. **Two scales.** The whole-model map runs the same algorithm again one level up: each cluster
   drawing becomes a tile, tiles are ordered by metabolic flow and packed into a captioned poster.

Cofactors are drawn as curved side branches off the reaction arrow, paired on one side, the way
curated maps draw them. Labels sit beside the thing they name, shrinking to a legible minimum
rather than drifting away from it.

---

## Quality

Every map is scored on edge orthogonality, edge crossings, node separation, label collisions and
local density. Across the collection the median map is fully axis-aligned with no edge crossings
and no label overlaps.

These are automatic layouts, and they are not uniformly perfect. Known limits:

- A few reactions per genome-scale model have no drawable primary pair — typically small
  inorganic chemistry such as catalase or CO₂ transport — and are omitted from the map.
- Very dense fans, biomass reactions above all, still produce occasional label collisions.
- The whole-model maps are dense by construction; the per-cluster maps are the ones to read.

Corrections and improved layouts are welcome — open an issue or a pull request.

---

## License

MIT (see [LICENSE](LICENSE)). The licence governs reuse of the files; the citation requirement
above is the condition for using them in scholarly or published work.

*Created by Tianyu Wu (GitHub: [forxhunter](https://github.com/forxhunter)), University of
Illinois Urbana-Champaign.*
