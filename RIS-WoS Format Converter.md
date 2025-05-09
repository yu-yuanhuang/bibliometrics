# RIS-WoS Format Converter

*Yu-Yuan Huang*
@[ResearchGate](https://www.researchgate.net/profile/Yu-Yuan-Huang-4?ev=hdr_xprf)

https://zenodo.org/records/15369097



> When conducting bibliometric analysis, we often encounter citation data conversion problems when processing Chinese literature database data. This document is suitable for Taiwanese researchers who use the Airiti and NDLTD databases and need to import data into CiteSpace analysis software.
>
> The processed dataset can be imported and analyzed in CiteSpace like the original WoS dataset.

## Solve the need to convert EndNote documents to WoS.

- Select RIS format for export.
- Open the code in Colab and upload the exported RIS file
- Automatically download files when the conversion is complete

Output: .txt plain text file in the Web of Science (WoS) format Modifications: Process multiple documents one by one Abstract (AB) multi-line merge Author affiliation (C1/AD) integration Journal name auto-completion (JO/T2/JF/PB) DOI check and fill Chinese and English keywords are distinguished as DE/ID

> [!NOTE]
>
> - Upload .ris file
> - Read content
> - Basic field processing function
> - Single RIS → WoS format function (add FN, VR to each transaction)
> - Process all records
> - Combined Output
> - Export to file and download
> - The program can be run in colab

```
import re
from google.colab import files

# === Upload .ris file ===
uploaded = files.upload()
filename = list(uploaded.keys())[0]

# === Read content ===
with open(filename, 'r', encoding='utf-8') as f:
    ris_text = f.read()

# === Basic field processing function ===
def extract_ris_value(tag, content):
    pattern = rf'^{tag}  - (.*)$'
    return re.findall(pattern, content, re.MULTILINE)

def extract_multiline(tag, text):
    lines = text.splitlines()
    content = []
    collecting = False
    for line in lines:
        if line.startswith(f"{tag}  -"):
            collecting = True
            content.append(line[len(tag) + 4:].strip())
        elif collecting and not re.match(r"^[A-Z][A-Z0-9]  -", line):
            content.append(line.strip())
        elif collecting:
            break
    return [" ".join(content)] if content else []

# === Single RIS → WoS format function ===
def ris_to_wos(record):
    au_list = extract_ris_value("AU", record)
    ti_list = extract_ris_value("TI", record) or extract_ris_value("T1", record)
    py_list = extract_ris_value("PY", record)
    ab_list = extract_multiline("AB", record)
    kw_list = extract_ris_value("KW", record)
    doi = extract_ris_value("DO", record)
    volume = extract_ris_value("VL", record)
    issue = extract_ris_value("IS", record)
    sp = extract_ris_value("SP", record)
    ep = extract_ris_value("EP", record)
    so_list = extract_ris_value("JO", record) or extract_ris_value("T2", record) or extract_ris_value("JF", record) or extract_ris_value("PB", record) or ["[Unknown Journal]"]
    c1_list = extract_ris_value("C1", record) or extract_ris_value("AD", record)

    wos_lines = [
        "FN RIS-WoS Format Converter by Yu-Yuan Huang",
        "VR 1.0",
        "PT J"
    ]
    wos_lines += [f"AU {a}" for a in au_list]
    if ti_list:
        wos_lines.append(f"TI {ti_list[0]}")
    wos_lines.append(f"SO {so_list[0]}")
    if c1_list:
        wos_lines.append(f"C1 {'; '.join(c1_list)}")
    if kw_list:
        zh = '; '.join([kw for kw in kw_list if not re.search(r'[a-zA-Z]', kw)])
        en = '; '.join([kw for kw in kw_list if re.search(r'[a-zA-Z]', kw)])
        if zh:
            wos_lines.append(f"DE {zh}")
        if en:
            wos_lines.append(f"ID {en}")
    if ab_list:
        wos_lines.append(f"AB {ab_list[0]}")
    if doi and doi[0].strip():
        wos_lines.append(f"DI {doi[0]}")
    else:
        wos_lines.append("DI [DOI unavailable]")
    if py_list:
        wos_lines.append(f"PY {py_list[0]}")
    if volume:
        wos_lines.append(f"VL {volume[0]}")
    if issue:
        wos_lines.append(f"IS {issue[0]}")
    if sp:
        wos_lines.append(f"BP {sp[0]}")
    if ep:
        wos_lines.append(f"EP {ep[0]}")
    wos_lines.append("ER")
    return "\n".join(wos_lines)

# === Process all records ===
records = re.findall(r'TY  -.*?ER  -', ris_text, re.DOTALL)
converted_records = [ris_to_wos(r) for r in records]

# === Combined Output ===
wos_output = "\n\n".join(converted_records)

# === Export to file and download ===
output_filename = "converted_wos_with_FN_VR_per_record.txt"
with open(output_filename, 'w', encoding='utf-8') as f:
    f.write(wos_output)

files.download(output_filename)
```

