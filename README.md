# TMO Geotechnical Screening

Screening geotécnico remoto para seguro quinquenal (Art. 618, Código Civil Brasileiro).

## O que é

API que deriva propriedades geotécnicas a partir de dados satelitais **SoilGrids 250m** (ISRIC) + contexto geológico **SGB/CPRM**, sem necessidade de amostragem de campo como triagem inicial.

Parte do ecossistema **FIT Holding** — inspeções de engenharia civil para seguradoras.

## Dados Primários

- **14 propriedades do solo** via SoilGrids 250m (ISRIC/Google Earth Engine)
- **Contexto geológico** via SGB/CPRM WFS (litoestratigrafia 1:1M)
- **3 camadas**: superfície (0-30cm), fundação (30-100cm), profundo (100-200cm)

## Derivações Geotécnicas (10 PTFs)

| Derivação | Referência |
|-----------|-----------|
| Condutividade Hidráulica (Ksat) | Saxton & Rawls (2006) |
| Erodibilidade (K-factor) | Williams/EPIC (1995) |
| Classificação USCS | ASTM D2487 simplificada |
| Limites de Atterberg (LL, LP, IP) | Siltação + CEC |
| Potencial Expansivo | Seed et al. (1962) |
| Compactação (densidade crítica) | Reichert et al. (2009) |
| CBR Estimado | De Graft-Johnson & Bhatia (1969) |
| Recomendação de Fundação | NBR 6122:2019 (simplificada) |
| Comparação entre Camadas | Detecção de camada impeditiva |
| Scoring de Risco | 8 flags × 2 pts (0-16) |

## Correção Mineralógica

Formações geológicas brasileiras têm mineralogia específica que afeta o comportamento da argila:

- **Caulinita** (Caiuá, Bauru, Barreiras): argila inerte, activity_ratio 0.30-0.40
- **Montmorillonita** (Guabirotuba): argila expansiva, activity_ratio 1.50
- **Óxidos de Fe/Al** (Latossolos): estabilizam, activity_ratio 0.30

A API aplica `effective_clay = clay × activity_ratio` antes das PTFs quando a formação é identificada.

## Scoring de Risco

| Score | Classe | Fator Prêmio | Ação |
|-------|--------|--------------|------|
| 0-4 | Baixo | 1.00 | Inspeção padrão |
| 5-8 | Moderado | 1.15 | Inspeção reforçada |
| 9-12 | Moderado-Alto | 1.35 | SPT/CPT obrigatório |
| 13-16 | Alto | 1.60 | Investigação geotécnica completa |

## Modelo Complementar

> Screening remoto **não substitui** investigação de campo (NBR 6122:2019, NBR 8036:1983).

O modelo é **complementar**: triagem remota direciona a inspeção de campo, reduzindo custo e aumentando cobertura.

```
Screening Remoto (API) → Inspeção Dirigida (Campo) → Laudo Final (Engenheiro)
```

## Relatórios

- [Viabilidade de Dados Primários](https://deandradeneves.github.io/tmo-geotechnical-screening/) — análise das 14 propriedades + 10 derivações
- [Documentação Técnica v2.0](https://deandradeneves.github.io/tmo-geotechnical-screening/documentacao.html) — arquitetura, PTFs, correção mineralógica, modelo complementar
- [Validação v2.0 — 3 Vias](https://deandradeneves.github.io/tmo-geotechnical-screening/resultados/validacao_v2.html) — API v1 × API v2 (CPRM) × Laudo de Campo
- [Validação v1.0 — Caso de Estudo](https://deandradeneves.github.io/tmo-geotechnical-screening/resultados/validacao_agua_azul.html) — diagnóstico pré-CPRM e cascata de erros

## Stack

- Python 3.12 (Google Cloud Functions)
- Google Earth Engine (SoilGrids 250m)
- SGB/CPRM WFS (contexto geológico)
- Zero dependências externas no módulo geotécnico

## Limitações

- Resolução 250m (não diferencia lotes individuais)
- Áreas urbanas densas retornam null (sem cobertura SoilGrids)
- PTFs são estimativas de screening, não substituem SPT/CPT
- Limites do Brasil: lat [-35, 6], lon [-75, -34]

---

*TMO — FIT Holding | Engenharia Civil & Seguros*
