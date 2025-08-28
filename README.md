# kumar_manju_files
import xml.etree.ElementTree as ET
import csv

# ---------- CONFIG ----------
xml_file = "informatica_mapping.xml"

# Output files
sources_csv = "sources.csv"
targets_csv = "targets.csv"
connectors_csv = "connectors.csv"
logic_csv = "transformation_logic.csv"

# ---------- PARSING ----------
tree = ET.parse(xml_file)
root = tree.getroot()

# Namespace check (some XMLs have namespaces, strip them)
def strip_ns(tag):
    return tag.split('}')[-1] if '}' in tag else tag

# Collect data
sources, targets, connectors, logics = [], [], [], []

# ---- Sources ----
for src in root.findall(".//SOURCE"):
    src_name = src.attrib.get("NAME")
    for field in src.findall(".//SOURCEFIELD"):
        sources.append([src_name,
                        field.attrib.get("NAME"),
                        field.attrib.get("DATATYPE"),
                        field.attrib.get("PRECISION"),
                        field.attrib.get("SCALE")])

# ---- Targets ----
for tgt in root.findall(".//TARGET"):
    tgt_name = tgt.attrib.get("NAME")
    for field in tgt.findall(".//TARGETFIELD"):
        targets.append([tgt_name,
                        field.attrib.get("NAME"),
                        field.attrib.get("DATATYPE"),
                        field.attrib.get("PRECISION"),
                        field.attrib.get("SCALE")])

# ---- Connectors (lineage) ----
for conn in root.findall(".//CONNECTOR"):
    connectors.append([
        conn.attrib.get("FROMINSTANCE"),
        conn.attrib.get("FROMFIELD"),
        conn.attrib.get("TOINSTANCE"),
        conn.attrib.get("TOFIELD")
    ])

# ---- Transformations (logic, hardcoded values) ----
for trans in root.findall(".//TRANSFORMATION"):
    tname = trans.attrib.get("NAME")
    ttype = trans.attrib.get("TYPE")

    # Expression / Aggregator field logic
    for field in trans.findall(".//TRANSFORMFIELD"):
        expr = field.attrib.get("EXPRESSION")
        if expr:
            logics.append([tname, ttype, field.attrib.get("NAME"), expr])

    # Table attributes (filter, join, lookup)
    for attr in trans.findall(".//TABLEATTRIBUTE"):
        name = attr.attrib.get("NAME")
        value = attr.attrib.get("VALUE")
        if value:
            logics.append([tname, ttype, name, value])

# ---------- WRITE TO CSV ----------
def write_csv(filename, header, rows):
    with open(filename, "w", newline="", encoding="utf-8") as f:
        writer = csv.writer(f)
        writer.writerow(header)
        writer.writerows(rows)

write_csv(sources_csv, ["Source", "Column", "Datatype", "Precision", "Scale"], sources)
write_csv(targets_csv, ["Target", "Column", "Datatype", "Precision", "Scale"], targets)
write_csv(connectors_csv, ["FromInstance", "FromField", "ToInstance", "ToField"], connectors)
write_csv(logic_csv, ["Transformation", "Type", "Field/Attr", "Expression"], logics)

print("✅ Extraction complete!")
print(f"- Sources → {sources_csv}")
print(f"- Targets → {targets_csv}")
print(f"- Connectors → {connectors_csv}")
print(f"- Transformation Logic → {logic_csv}")