# PathoReason: frozen analysis plans (pre-run timestamp)

These analysis plans were frozen before any test-tile run. This repository holds hashes and the plans only. It contains no MHIST data.

| file | SHA-256 |
|---|---|
| PLAN.md (MedGemma adapter, order test, 100 label-balanced test tiles) | fbc4856c071f7c79027c97e7842a9522cf89d6be032e9a7fff608984e61815d2 |
| PLAN_nctcrc.md (NCT-CRC-HE order experiment) | d97481e58d269953353320c145f7c7a0aa2074c03ed4fde560a7a5ccf65e1125 |
| FROZEN_CODE.sha256 (code list quoted by PLAN.md) | b8b37948ec881569d148dd4616c091b9d66c26133bb247677621f7005f52d578 |

Frozen MedGemma test tile list (100 names, kept private): SHA-256 d56ee265137924a185e257d76ff616e3e642f81769e112c2f8fe630ea24a04b8

Check: `shasum -a 256 PLAN.md PLAN_nctcrc.md FROZEN_CODE.sha256`
