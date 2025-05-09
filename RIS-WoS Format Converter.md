# RIS-WoS Format Converter

解決EndNote文獻資料轉換至WoS使用需求

- 匯出選擇RIS格式
- 在Colab開啟程式碼，上傳匯出之RIS檔
- 完成轉換時自動下載文件

輸出：符合 Web of Science（WoS）格式的 .txt 純文字檔 修正項目： 多筆文獻逐筆處理 摘要（AB）多行合併 作者單位（C1/AD）整合 期刊名稱自動補齊（JO/T2/JF/PB） DOI 檢查與補值 中文與英文關鍵字區分為DE/ID

> [!NOTE]
>
> 上傳 .ris 檔案
>
> 讀取內容
>
> 基礎欄位處理函數
>
> 單筆 RIS → WoS 格式函數（每筆加 FN, VR）
>
> 處理所有記錄
>
> 組合輸出
>
> 輸出為檔案並下載
>
> 程序可在colab運行

```
import re
from google.colab import files

# === 上傳 .ris 檔案 ===
uploaded = files.upload()
filename = list(uploaded.keys())[0]

# === 讀取內容 ===
with open(filename, 'r', encoding='utf-8') as f:
    ris_text = f.read()

# === 基礎欄位處理函數 ===
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

# === 單筆 RIS → WoS 格式函數（每筆加 FN, VR）===
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

# === 處理所有記錄 ===
records = re.findall(r'TY  -.*?ER  -', ris_text, re.DOTALL)
converted_records = [ris_to_wos(r) for r in records]

# === 組合輸出 ===
wos_output = "\n\n".join(converted_records)

# === 輸出為檔案並下載 ===
output_filename = "converted_wos_with_FN_VR_per_record.txt"
with open(output_filename, 'w', encoding='utf-8') as f:
    f.write(wos_output)

files.download(output_filename)
```

