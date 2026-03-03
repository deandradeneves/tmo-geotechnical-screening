# TMO — Geotechnical Screening

## Overview
TMO is part of **FIT Holding** — civil engineering inspections that back the **seguro quinquenal** (5-year structural damage insurance post-construction, required by Brazilian Civil Code Art. 618).

This project provides a **geotechnical screening API** that derives engineering properties from SoilGrids 250m satellite data. It enables risk-based insurance underwriting without requiring field sampling as a first pass.

---

## API

### Endpoint
```
POST https://us-central1-terra-verde-gee.cloudfunctions.net/getSoilData
Content-Type: application/json
```

### Parameters
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `lat` | float | Yes | — | Latitude (decimal, negative for South) |
| `lon` | float | Yes | — | Longitude (decimal, negative for West) |
| `depth` | string | No | `0-30cm` | Depth interval |
| `profile` | string | No | `agricultural` | `agricultural`, `geotechnical`, or `full` |

### Profiles

- **`agricultural`** — Original soil properties + agricultural derived metrics (texture, AWC, OM). Backward-compatible, no `geotechnical` key.
- **`geotechnical`** — Topsoil (0-30cm) + subsoil (30-100cm) + full geotechnical assessment. Response includes `topsoil`, `subsoil`, and `geotechnical` dicts.
- **`full`** — Same as geotechnical (reserved for future expansion with additional profiles).

### Usage Examples

```bash
# Agricultural (default, backward compatible)
curl -s -X POST https://us-central1-terra-verde-gee.cloudfunctions.net/getSoilData \
  -H 'Content-Type: application/json' \
  -d '{"lat": -23.55, "lon": -46.63}'

# Geotechnical screening
curl -s -X POST https://us-central1-terra-verde-gee.cloudfunctions.net/getSoilData \
  -H 'Content-Type: application/json' \
  -d '{"lat": -23.55, "lon": -46.63, "profile": "geotechnical"}'
```

---

## Geotechnical Response Format

```json
{
  "latitude": -23.55,
  "longitude": -46.63,
  "profile": "geotechnical",
  "topsoil": { "depth": "0-30cm", "ph": 5.8, "clay_pct": 32.5, ... },
  "subsoil": { "depth": "30-100cm", "ph": 5.4, "clay_pct": 45.1, ... },
  "geotechnical": {
    "hydraulic":        { "ksat_mm_h": 2.4, "permeability_class": "Slow" },
    "erodibility":      { "k_factor": 0.38, "erosion_risk": "High" },
    "classification":   { "uscs_group": "CL", "uscs_description": "Lean Clay" },
    "plasticity":       { "liquid_limit_est": 42, "plastic_limit_est": 22, "plasticity_index_est": 20 },
    "shrink_swell":     { "class": "Moderate", "score": 4 },
    "compaction":       { "bd_critical": 1.65, "compaction_ratio": 0.82, "compaction_risk": "Low" },
    "bearing":          { "cbr_estimated": 5.2, "subgrade_class": "Fair" },
    "foundation":       { "shallow_ok": true, "recommended": ["Sapata corrida", "Radier", "Sapata isolada"] },
    "layer_comparison": { "clay_jump_pct": 12.6, "impediment_layer": true },
    "risk_assessment":  { "risk_score": 8, "risk_class": "Moderate", "premium_factor": 1.15, "flags": [...], "inspection_checklist": [...] }
  }
}
```

---

## Risk Score Interpretation

| Score | Class | Premium Factor | Action |
|-------|-------|---------------|--------|
| 0-4 | Low | 1.00 | Standard inspection |
| 5-8 | Moderate | 1.15 | Enhanced inspection recommended |
| 9-12 | Moderate-High | 1.35 | Mandatory SPT/CPT + geotechnical review |
| 13-16 | High | 1.60 | Full geotechnical investigation required |

**8 risk flags** (2 points each):
1. Low permeability (Ksat < 1.5 mm/h)
2. High erodibility (K > 0.35)
3. High plasticity (PI > 25)
4. Shrink-swell (Moderate-High or High)
5. Compaction risk (ratio >= 0.9)
6. Low CBR (< 5)
7. Impediment layer between topsoil/subsoil
8. Problematic USCS group (CH, MH, OH, OL, PT)

---

## Architecture

```
Shared Cloud Function (Terra Verde GEE)
├── main.py          — HTTP handler, GEE queries, profile routing
├── geotech.py       — Pure Python geotechnical derivations (10 functions)
├── requirements.txt — earthengine-api, functions-framework
├── deploy.sh/.bat   — GCP deploy scripts
```

The geotechnical module (`geotech.py`) is pure Python with zero GEE dependency. All pedotransfer functions use well-established references (Saxton & Rawls 2006, Williams/EPIC 1995, Reichert 2009).

---

## Roadmap

### Phase 1 (Current) — API MVP
- [x] SoilGrids geotechnical derivations via Cloud Function
- [x] Risk scoring for insurance underwriting

### Phase 2 — Enhanced Data
- [ ] HiHydroSoil integration (Ksat measured, not estimated)
- [ ] SRTM slope/aspect for drainage assessment
- [ ] Historical flood risk overlay (ANA data)

### Phase 3 — Agent Integration
- [ ] n8n MCP integration (TMO-specific workflow)
- [ ] AI agent for inspection report generation
- [ ] PDF report generation with maps

### Phase 4 — Full Platform
- [ ] GoSellers ecosystem integration (Chatwoot, Forms API)
- [ ] Client portal for construction companies
- [ ] Integration with Brazilian insurance regulatory data (SUSEP)

---

*Last updated: 2026-03-03*
