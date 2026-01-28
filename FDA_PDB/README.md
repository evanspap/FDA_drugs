
# FDA–PDB Ligand Mapping & UniProt Fusion Pipeline  
**Author:** Evangelos Papadopoulos  
**Version:** 4.0 — Canonical Clean Edition (2025)  
**Status:** Complete, Reproducible, GitHub‑Ready  
**License:** MIT (optional – add if desired)  

---

# 📚 Overview

This repository implements a **fully automated structural‑bioinformatics pipeline** that:

1. Downloads **FDA‑approved small molecule drugs** from ChEMBL.  
2. Extracts/normalizes SMILES + InChI.  
3. Matches FDA compounds to the **PDB Chemical Component Dictionary (CCD)** using:  
   - exact InChI match  
   - canonical match  
   - fingerprint similarity  
4. Retrieves structural occurrence information for each ligand using two PDBe APIs:  
   - **UniProt Ligand Interaction API**  
   - **PDBe in_pdb POST API**  
5. Generates **flat TSV** + **full JSON** metadata describing:  
   - PDB IDs for every drug  
   - interacting UniProt IDs  
   - target organisms  
   - entity names  
   - raw API metadata (for programmatic analysis)  

This enables:  

- Drug repurposing  
- Off‑target mapping  
- Cross‑species structural pharmacology  
- Dataset creation for ML models  
- Template selection for docking  

---

# 🗺 Pipeline Diagram

```
                    ┌─────────────────────────┐
                    │   ChEMBL FDA JSON       │
                    │  chembl_fda.json        │
                    └─────────────┬───────────┘
                                  │
                                  ▼
                    ┌─────────────────────────┐
                    │  1. extract_smiles.py   │
                    │  → FDA_OUT01/           │
                    └─────────────┬───────────┘
                                  │
                                  ▼
                    ┌─────────────────────────┐
                    │ 2. match_to_pdb_components.py
                    │    - ExactMatch_CCD
                    │    - CanonicalMatch_CCD
                    │    - FingerprintMatch_CCD
                    │ → FDA_OUT02/ matched_compounds
                    └─────────────┬───────────┘
                                  │
                                  ▼
                     ┌────────────────────────┐
                     │ 3. fusion_pdb_uniprot_compounds.py
                     │    - UniProt Ligand API
                     │    - PDBe in_pdb API
                     │ → FDA_OUT03/ fusion_output
                     └────────────────────────┘
```

---

# 📁 Directory Structure

```
project/
├── input/
│   └── chembl_fda.json
├── scripts/
│   ├── extract_smiles.py
│   ├── match_to_pdb_components.py
│   └── fusion_pdb_uniprot_compounds.py
├── output/
│   ├── FDA_OUT01/
│   ├── FDA_OUT02/
│   └── FDA_OUT03/
└── README.md
```

---

# 🧪 Scripts (Canonical Clean Versions)

Below are the **exact scripts** used in the final pipeline.

---

## 1. **extract_smiles.py**  
Extracts SMILES, InChI, names, ChEMBL IDs.

### Usage
```bash
python3 extract_smiles.py chembl_fda.json FDA_OUT01/
```

### Output
- `FDA_OUT01/fda_molecules_clean.json`
- `FDA_OUT01/fda_molecules_clean.tsv`

---

## 2. **match_to_pdb_components.py**  
Matches FDA drugs → PDB CCD 3‑letter codes.

### Usage
```bash
python3 match_to_pdb_components.py FDA_OUT01/fda_molecules_clean.tsv FDA_OUT02/
```

### Output
- `FDA_OUT02/matched_compounds.json`
- `FDA_OUT02/matched_compounds.tsv`

CCD annotations include:  
- `ExactMatch_CCD`  
- `CanonicalMatch_CCD`  
- `FingerprintMatch_CCD`  
- `FPScore`

---

## 3. **fusion_pdb_uniprot_compounds.py** (v3.0 canonical edition)  
Fuses UniProt + in_pdb API results.

### Usage
```bash
python3 fusion_pdb_uniprot_compounds.py FDA_OUT02/matched_compounds.json FDA_OUT03/
```

### Output
- `fusion_output.tsv` (flat, Excel‑friendly)  
- `fusion_output.json` (complete metadata)

### TSV Columns
| Column             | Description |
|-------------------|-------------|
| Name              | Compound name |
| ChEMBL_ID         | ChEMBL identifier |
| CCD               | 3-letter PDB CCD code |
| Number_of_Targets | # of UniProt targets |
| pdb_ids_all       | All PDB IDs (UniProt ∪ in_pdb) |
| pdb_ids_uniprot   | PDB IDs from UniProt API |
| pdb_ids_inpdb     | PDB IDs from in_pdb API |
| targets_uniprot   | All interacting UniProt IDs |
| organisms         | Scientific names only |

---

# 🌐 PDBe API Documentation

## **1. UniProt Ligand Interaction API**
```
https://www.ebi.ac.uk/pdbe/api/v2/compound/uniprot/<CCD>
```

CLI test:
```bash
curl -s "https://www.ebi.ac.uk/pdbe/api/v2/compound/uniprot/BO2" | jq .
```

---

## **2. PDBe in_pdb POST API**
```
https://www.ebi.ac.uk/pdbe/api/pdb/compound/in_pdb/
```

CLI test:
```bash
curl -s \
  -X POST \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "BO2" \
  "https://www.ebi.ac.uk/pdbe/api/pdb/compound/in_pdb/" | jq .
```

---

# 📊 Example Output (TSV Preview)

```
Name    ChEMBL_ID   CCD   Number_of_Targets   pdb_ids_all
BORTEZOMIB CHEMBL325041 BO2 22 2f16,3mg0,4fwd,4qvl,...
RUCAPARIB CHEMBL1173055 RPB 17 8hko,4bjc,9goi,...
...
```

---

# 📦 Example Output (JSON Snippet)

```json
{
  "input_record": {
    "Name": "BORTEZOMIB",
    "ChEMBL_ID": "CHEMBL325041",
    "ExactMatch_CCD": "BO2"
  },
  "ccd": "BO2",
  "uniprot_pdb_list": ["6td5","5l66","7lxt"],
  "inpdb_pdb_list": ["2f16","4qvv","9goi"],
  "pdb_all": ["2f16","4qvv","5l66","6td5","7lxt","9goi"]
}
```

---

# 💳 Donations
If you find this pipeline helpful:

### ☕ Buy me a coffee  
https://buymeacoffee.com/evanspap

### 💸 Venmo  
@EvangelosPapadopoulos

---

# 🚀 Final Notes

This repository now contains a fully documented, fully reproducible structural‑bioinformatics pipeline suitable for:

- research groups  
- drug discovery teams  
- academic teaching  
- ML dataset generation  
- automated structural analysis  

If you want, I can also provide:

- a **LICENSE file**  
- minimal **GitHub Actions CI**  
- an **example Jupyter notebook**  
- a **pipeline diagram PNG/SVG**  

Just ask and it will be added.

---

# ✔ Ready for GitHub Upload
