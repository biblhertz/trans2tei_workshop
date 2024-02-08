# Conversion from PAGE with inline layout to TEI

Encoding of specific textual phenomenons during the several steps:

| Phenomenon       | Example                                            | Transcription  | After replacement                    | After transformation                                                                                 |
|:-----------------|:---------------------------------------------------|:---------------|:-------------------------------------|:-----------------------------------------------------------------------------------------------------|
| Italics          | *italics*                                          | `⌠italics⌡`    | `<hi rendition="#i">italics</hi>`    | `<hi rendition="#i">italics</hi>`                                                                    |
| Supplied         | [*supplied*]                                       | `[⌠supplied⌡]` | `<supplied>supplied</supplied>`      | `<supplied>supplied</supplied>`                                                                      |
| Bold text        | **bold**                                           | `↾bold⇃`       | `<hi rendition="#b">bold</hi>`       | `<hi rendition="#b">bold</hi>`                                                                       |
| Document links   | [**F42**]                                          | `[F42]`        | `[<ref target="#doc_F42">F42</ref>]` | `[<ref target="#doc_F42">F42</ref>]`                                                                 |
| Small caps       | <span style="font-variant: small-caps;">abc</span> | `₍ABC₎`        | `<hi rendition="#k">abc</hi>`        | `<hi rendition="#k">abc</hi>`                                                                        |
| Footnote marks   | Lorem<sup>1</sup> ipsum                            | `⊂1⊃`          | `<note type="refnum">1</note>`       | `Lorem<note n="1" xml:id="facs_18_note_3" place="foot" facs="#facs_18_r_3_1">Sit amet.</note> ipsum` |
| Footnote numbers | <sup>1</sup> Sit amet.                             | `⊤1⊥`          | `<hi rendition="#sup">1</hi>`        |                                                                                                      |

## Example `simplify.py`

### page2tei

We are happy to rely on [dariok/page2tei](https://github.com/dariok/page2tei) by Dario Kampkaspar as the first step
in our XML conversion.

```python
xml_data = transform.page2tei(args.infile_name)
print(xml_data, file=open("temp/tei.xml", 'w'))
```

### Post-processing specific to our workflow

This step adds the default renditions in the teiHeader and replaces some elements with a type by a native TEI element.

- `ab[@type='quote']` to `quote` for block quotes
- `ab[@type='commentary']` to `p[@type='commentary']` for commentary paragraphs (REMS)
- `ab[@type='first-paragraph']` to `p[@type='first-paragraph']` for non-indented paragraphs
- `ab[@type='paragraph-center']` to `p[@type='center'][@rendition='#c']` for centered paragraphs
- `ab[@type='bibliography']` to `p[@type='bibliography']` for bibliography lists (REMS)
- `ab[@type='archival-notice']` to `p[@type='archival-notice']` 
- `ab[@type='paragraph']` to `p[@rendition='#aq #ident']` 
- `ab[@type='other']` to `p[@rendition='#aq #ident']` 
- `fw[@type='page-number']` to `fw[@type='page'][@place='bottom']`

This step is unlikely to break the pipeline.

```python
xml_data = transform.postprocess_page2tei(xml_data)
print(xml_data, file=open("temp/tei-post.xml", "w"))
```

### Deletion of `tei:facsimile`

Here, the `tei:facsimile` element is removed because there is no intention of a parallel view.

```python    
xml_data = transform.remove_position_data(xml_data)
print(xml_data, file=open("temp/positions.xml", "w"))
```

### Encode italics

`⌠(.*?)⌡` -> `<hi rendition="#i">\1</hi>`

Before

```xml
⌠italics⌡
```

After 

```xml
<hi rendition="#i">italics</hi>
```

```python
# Text in italics is tagged in plain text with integrals. Replace with according tags.
xml_data = replacements.cursive_text(xml_data)
print(xml_data, file=open("temp/cur.xml", 'w'))
```

### Encode footnote numbers in the text

`⊂([0-9]+)⊃` -> `<note type="refnum">\1</note>`

Before

```xml
⊂1⊃
```

After 

```xml
<note type="refnum">1</note>
```

```python
xml_data = replacements.footnote_numbers(xml_data)
print(xml_data, file=open("temp/fn1.xml", 'w'))
```

### Encode footnote numbers in the footnotes

`⊤([0-9]+)⊥` -> `<hi rendition="#sup">\1</hi>`

Before

```xml
⊤1⊥
```

After 

```xml
<hi rendition="#sup">1</hi>
```

```python
xml_data = replacements.footnotes(xml_data)
print(xml_data, file=open("temp/fn2.xml", 'w'))
```

### Move footnotes to their inline position


Before

```xml
<div>
    <p>Lorem ipsum<note type="refnum">1</note> dolor</p>
    <note><hi rendition="#sup">1</hi> Sit amet.</note>
</div>

```

After 

```xml
<div>
    <p>Lorem ipsum<note place="foot" n="1">Sit amet.</note> dolor</p>
</div>
```

```python
xml_data = transform.move_footnotes(xml_data)
print(xml_data, file=open("temp/fn3.xml", 'w'))
```

### Encode small caps

`₍([A-Z\-]+)₎` -> `<hi rendition="#k">` + `m.group(1).lower()` + `</hi>`

```
₍ABC₎
```

After 

```xml
<hi rendition="#k">abc</hi>
```

```python
xml_data = replacements.small_caps(xml_data)
print(xml_data, file=open("temp/scap.xml", 'w'))
```

### First paragraph

`type="first-paragraph"` -> `type="first" rendition="#aq"`

```xml
<p type="first-paragraph">Lorem</p>
```

After 

```xml
<p type="first" rendition="#aq">Lorem</p>
```

```python
xml_data = replacements.first_paragraph(xml_data)
print(xml_data, file=open("temp/para.xml", 'w'))
```

### Encode document links

`\[↾([F0-9/\-]+)⇃\]` -> `lambda m: '[<ref target="#doc_' + re.sub(r'/', '_', m.group(1)) + '">' + m.group(1) + '</ref>]'"`

```
[↾1550/4⇃]
```

After 

```xml
[<ref target="#doc_1550_4">1550/4</ref>]
```

```python
xml_data = replacements.document_links(xml_data)
print(xml_data, file=open("temp/link.xml", 'w'))
```

### Encode bold text

`↾([^<>]?)⇃` -> `<hi rendition="#b">\1</hi>"`

Before

```
↾bold⇃
```

After 

```xml
<hi rendition="#b">bold</hi>
```

```python
xml_data = replacements.bold_text(xml_data)
print(xml_data, file=open("temp/bold.xml", 'w'))
```

### Move page begins into corresponding sections 1

`(<pb facs="#facs_([0-9]+)".*? />\n)(\s+)(</div>\n\s+<div>\n)` -> `\4\3\1`


Before

```xml
<div>
    <div>
        <p>Lorem ispum</p>
        <pb facs="#facs_([0-9]+)"/>
    </div>
    <div>
        <head>Dolor sit.</head>
    </div>
</div>
```
After

```xml
<div>
    <div>
        <p>Lorem ipsum</p>
    </div>
    <div>
        <pb facs="#facs_([0-9]+)"/>
        <head>Dolor sit.</head>
    </div>
</div>
```

```python
xml_data = replacements.move_pb_into_div1(xml_data)
print(xml_data, file=open("temp/pb1.xml", 'w'))
```

### Move page begins into corresponding sections 2

`(<pb facs="#facs_[0-9]+".*? />\s*<fw.*?>\s*.*?</fw>\s*)(</div>\s*<div>\s*)` -> `\2\1`

Before

```xml
<div>
    <div>
        <p>Lorem ispum</p>
        <pb facs="#facs_([0-9]+)"/>
        <fw/>
    </div>
    <div>
        <head>Dolor sit.</head>
    </div>
</div>
```
After

```xml
<div>
    <div>
        <p>Lorem ispum</p>
    </div>
    <div>
        <pb facs="#facs_([0-9]+)"/>
        <fw/>
        <head>Dolor sit.</head>
    </div>
</div>
```

```python
xml_data = replacements.move_pb_into_div2(xml_data)
print(xml_data, file=open("temp/pb2.xml", 'w'))
```

### Move page information into `tei:pb`

Before

```xml
<div>
    <pb/>
    <fw type="header">Running title</fw>
    <fw type="page">42</fw>
</div>
```
After

```xml
<div>
    <pb n="42"/>
</div>
```

```python
xml_data = transform.page_numbers(xml_data)
print(xml_data, file=open("temp/pb3.xml", 'w'))
```

### Join paragraphs

Before

```xml
<div>
    <p>Lorem ipsum</p>
    <pb/>
    <p type="continued">dolor sit amet.</p>
</div>
```
After

```xml
<div>
    <p>Lorem ipsum
    <pb/>
    dolor sit amet.</p>
</div>
```

```python
xml_data = transform.join_paragraphs(xml_data)
print(xml_data, file=open("temp/rep1.xml", "w"))
```

### Join italics and small caps 1

Before

```xml
<p>Dolor sit <hi rendition="#i">italic</hi>
    <lb/><hi rendition="#i">letters</hi> sit <hi rendition="#k">small</hi>
    <lb/><hi rendition="#k">caps</hi>.</p>
```
After

```xml
<p>Dolor sit <i>italic</i>
    <lb/><i>letters</i> sit <k>small</k>
    <lb/><k>caps</k>.</p>
```

```python
# join hi #i and #k
xml_data = transform.simplify_hi(xml_data)
print(xml_data, file=open("temp/hi1.xml", "w"))
```

### Join italics and small caps 2

`</k>(¬?\s*(?:<pb [^<>]+/>)?\s*<lb [^<>]+/>\s*)<k>` -> `\1`  
`</i>(¬?\s*(?:<pb [^<>]+/>)?\s*<lb [^<>]+/>\s*)<i>` -> `\1`

Before

```xml
<p>Dolor sit <i>italic</i>
    <lb/><i>letters</i> sit <k>small</k>
    <lb/><k>caps</k>.</p>
```
After

```xml
<p>Dolor sit <i>italic
    <lb/>letters</i> sit <k>small
    <lb/>caps</k>.</p>
```

```python
xml_data = replacements.join_small_caps(xml_data)
xml_data = replacements.join_cursive_text(xml_data)
print(xml_data, file=open("temp/hi2.xml", "w"))
```

### Join italics and small caps 3

Before

```xml
<p>Dolor sit <i>italic
    <lb/>letters</i> sit <k>small
    <lb/>caps</k>.</p>
```
After

```xml
<p>Dolor sit <hi rendition="#i">italic
    <lb/>letters</hi> sit <hi rendition="#k">small
    <lb/>caps</hi>.</p>
```

```python
xml_data = transform.expand_hi(xml_data)
print(xml_data, file=open("temp/hi.xml", "w"))
```

### Encode hyphenation in TEI way

Before

```xml
<p>Kunst¬
    <lb/>werk</p>
```
After

```xml
<p>Kunst<lb break="no" rend="hyphen"/>werk</p>
```

```python
xml_data = replacements.replace_hyphens(xml_data)
print(xml_data, file=open("temp/hyph.xml", 'w'))
```

### Remove `tei:lb`

Before

```xml
...
```
After

```xml
...
```

```python
xml_data = transform.remove_lb(xml_data)
print(xml_data, file=open("temp/removed-lb.xml", 'w'))
```

### Remove `tei:pb`

Before

```xml
...
```
After

```xml
...
```

```python
xml_data = transform.remove_pb(xml_data)
print(xml_data, file=open("temp/removed-pb.xml", 'w'))
```

### Pretty print

Pretty outfile with indentation and breaks specified per element.

```python
xml_data = transform.indent(xml_data)
print(xml_data, file=open("temp/ind.xml", 'w'))
```

### Write outfile

```python
with open(args.outfile_name, "w") as outfile:
    outfile.write(xml_data)
outfile.close()
```