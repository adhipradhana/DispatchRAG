# DispatchRAG

**DispatchRAG** (APSIPA ASC 2026 — **Best Paper Award** 🏆) is a retrieval-grounded framework for turning a dashcam accident clip into a
structured, three-audience emergency-dispatch plan (dispatcher / civilian bystander / arriving
ambulance crew), with every directive grounded in a cited regulatory rule rather than free
generation. This repository is the dataset release: 500 human-annotated accident clips and the
19-rule Japanese regulatory corpus (Road Traffic Act, FDMA urgency-triage manual, JPTEC trauma
guidelines) the annotations are grounded in.

> 🏆 **Our paper has been accepted at APSIPA ASC 2026 and won the Best Paper Award.**
> Paper: [arXiv:2607.23132](https://arxiv.org/abs/2607.23132)

See [the project page](https://adhipradhana.github.io/DispatchRAG/) for a visual overview of the
dataset and how it's structured.

**No video is included in this repository.** Clips are derived from the public
[MM-AU (LOTVS-MM-AU)](https://github.com/jeffreychou777/LOTVS-MM-AU) dataset
("Abductive Ego-View Accident Video Understanding for Safe Driving Perception",
CC-BY-NC-4.0). Each annotation file names its source video using MM-AU's own
`video_id` / `type_id` numbering (see [Provenance](#provenance--source-video) below),
so it can be matched back to the original frames by anyone who has downloaded MM-AU
directly from its authors.

---

## Contents

```
accident-dispatch-dataset/
├── annotations/     500 JSON files, one per clip (see schema below)
├── corpus/
│   └── corpus.jsonl 19-rule regulatory corpus cited by the annotations
├── docs/            GitHub Pages site (dataset overview)
└── README.md
```

## Dataset statistics (n = 500)

**Urgency tier** (FDMA 4-tier triage):

| Tier | n |
|---|---|
| Non-Emergency (`non_urgent`) | 180 |
| Low-Emergency (`less_urgent`) | 122 |
| Semi-Emergency (`urgent`) | 121 |
| Emergency (`emergent`) | 77 |

**Collision type:**

| Type | n |
|---|---|
| Vehicle-to-vehicle | 143 |
| Vehicle-to-object | 91 |
| Vehicle-to-motorcycle | 90 |
| Vehicle-to-pedestrian | 79 |
| Single-vehicle | 66 |
| Vehicle-to-bicycle | 31 |

**Environment:** 429 sunny / 24 rainy / 47 snowy · 454 day / 46 night.

**Dispatched units (label frequency):** police 499, ambulance 320, doctor_car 75, fire_rescue 63.

## Annotation schema

Each file in `annotations/` is one clip:

```jsonc
{
  "clip_id": "sft-positive-000001-000018-000050",   // internal id (see mmau_source for the original)
  "annotation_source": "checklist",                  // human annotation via a pre-checked-directive UI
  "observation_tags": {
    "collision_type": "vehicle_vehicle",              // one of 6 enum values (see table above)
    "environment": {"weather": "...", "lighting": "...", "scene_type": "...", "road_geometry": "..."},
    "scene_hazards": ["deformed", "possible_injury", "..."],
    "vru_post_status": "..."                          // pedestrian/cyclist/motorcyclist post-impact state
    // ... additional scene-tag fields
  },
  "applicable_chunks": ["jp-doukoho-72-1", "..."],    // corpus chunk_ids that ground this clip's dispatch
  "pov_structured": {
    "pov_1": {                                        // dispatcher-facing
      "incident_summary": "...",
      "urgency": "non_urgent | less_urgent | urgent | emergent",
      "units": [{"unit": "police", "text": "...", "cite": ["jp-doukoho-72-1"]}, "..."],
      "coordination": []
    },
    "pov_2": {"do": [...], "warn": [...], "dont": [...]},   // civilian/bystander instructions, each item cited
    "pov_3": {                                        // arriving ambulance/fire crew
      "mechanism": {"text": "...", "cite": [...]},
      "load_and_go": "applicable | not_applicable | judgment_call",
      "destination": "...",
      "check": [...],
      "equipment": []
    }
  },
  "notes": "",
  "annotated_at": "2026-06-11T20:00:22",
  "mmau_source": {                                    // provenance back to the original MM-AU video
    "video_id": 1,
    "type_id": 10,
    "path_hint": "CAP-DATA/10/000001/images/",
    "note": "Path relative to the MM-AU CAP-DATA root ..."
  }
}
```

Every cited item throughout `pov_structured` references a `chunk_id` in `corpus/corpus.jsonl`
via its `cite` field, so the legal/procedural grounding for every dispatch directive is
resolvable standalone from the two folders in this repo (no external lookup needed).

## Corpus schema (`corpus/corpus.jsonl`)

19 chunks from 5 Japanese regulatory sources (Road Traffic Act Art. 71/72/117, FDMA urgency-tier
and 119 sub-tier definitions, JPTEC high-risk-mechanism and severity-assessment criteria, Shonan
Medical Center field protocols). Each chunk has `chunk_id`, `text_ja` (source language) and
`text_en` (English working translation), `mandatory` (always-applicable vs. retrieved-on-relevance),
`applicable_povs`, and `retrieval_keys` used for chunk retrieval.

## Provenance / source video

This repository intentionally does **not** redistribute video. `annotations/*.json` files are
named `{video_id:06d}.json`, where `video_id` is MM-AU's own video numbering (unique across all
500 clips in this dataset; also duplicated in each file's `mmau_source` field, along with the
`type_id`). To recover the source frames:

1. Download MM-AU's **CAP-DATA** split from the [official repo](https://github.com/jeffreychou777/LOTVS-MM-AU)
   (chunked archive; see their README for merge/extract instructions).
2. Locate `CAP-DATA/<type_id>/<video_id:06d>/images/` using the `mmau_source.path_hint` field.

**Each annotation was made watching the whole source video** (every frame under that path — the
lead-up as well as the accident itself), not a trimmed window; urgency triage in particular needs
the lead-up context.

## 👍 Acknowledgement

* [MM-AU (LOTVS-MM-AU)](https://github.com/jeffreychou777/LOTVS-MM-AU) — source dataset for every
  clip's video, collision-type category, and abnormal-frame window (paper: *"Abductive Ego-View
  Accident Video Understanding for Safe Driving Perception"*). Please cite the MM-AU paper and
  consult their repository for the canonical citation if you use the source video/frames this
  dataset references.

## License

* **Annotations & corpus** (this repository) — original work, released under **CC-BY-NC-4.0**,
  matching the license of the underlying MM-AU source material.
* **MM-AU source video/frames** — CC-BY-NC-4.0, licensed by its authors; see
  [their repository](https://github.com/jeffreychou777/LOTVS-MM-AU) for full terms.
* **Regulatory corpus text** — drawn from public Japanese government/medical-society sources
  (e-Gov Japan, FDMA, JPTEC); `corpus.jsonl` records each chunk's `source_url`.

## Citation

If you use this dataset, please cite our paper, published at APSIPA ASC 2026 (**Best Paper Award**): [arXiv:2607.23132](https://arxiv.org/abs/2607.23132)

```bibtex
@inproceedings{adhipradhana2026dispatchrag,
  title={DispatchRAG: Grounding Emergency Dispatch Decisions in Real-World Protocols from Traffic Accident Video},
  author={Adhipradhana, Muhammad Sulthan and Javanmardi, Ehsan and Bao, Naren and Tsukada, Manabu},
  booktitle={Proceedings of the Asia Pacific Signal and Information Processing Association Annual Summit and Conference (APSIPA ASC)},
  address={Bangkok, Thailand},
  month=nov,
  year={2026},
  note={Best Paper Award},
  eprint={2607.23132},
  archivePrefix={arXiv}
}
```
