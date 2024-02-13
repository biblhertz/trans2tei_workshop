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

Dependent code: `transform.page2tei("path-to/mets.xml")`  
Default output: `temp/tei.xml`

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

Dependent code: `transform.postprocess_page2tei(xml_string)`  
Default output: `temp/tei-post.xml`

### Deletion of `tei:facsimile`

Here, the `tei:facsimile` element is removed because there is no intention of a parallel view.

Dependent code: `transform.remove_position_data(xml_string)`  
Default output: `temp/positions.xml`

### Encode italics

Italics encoded by special characters are not visible as XML. They are replaced with regular expressions.
In this step, the new elements only exist as string and will turn into proper elements once the document is parsed.

`⌠(.*?)⌡` -> `<hi rendition="#i">\1</hi>`

Before:

```xml
<p facs="#facs_33_TextRegion_1631109932128_1025">
    <lb facs="#facs_33_r1l27" n="N001"/>Man stößt also hier auf eine unterste Schicht von Begriffen (daher der Name
    <lb facs="#facs_33_r1l28" n="N002"/>⌠Grund⌡begriffe), auf denen das bildliche Vorstellen in seiner allgemeinsten Form
    <lb facs="#facs_33_r1l29" n="N003"/>beruht.</p>
```

After:

```xml
<p facs="#facs_33_TextRegion_1631109932128_1025">
    <lb facs="#facs_33_r1l27" n="N001"/>Man stößt also hier auf eine unterste Schicht von Begriffen (daher der Name
    <lb facs="#facs_33_r1l28" n="N002"/><hi rendition="#i">Grund</hi>begriffe), auf denen das bildliche Vorstellen in seiner allgemeinsten Form
    <lb facs="#facs_33_r1l29" n="N003"/>beruht.</p>
```

Dependent code: `replacements.cursive_text(xml_string)`  
Default output: `temp/cur.xml`

### Encode footnote numbers in the text

Similarly, footnote numbers in the text are recognized with regular expressions and elements are inserted.

`⊂([0-9]+)⊃` -> `<note type="refnum">\1</note>`

Before:

```xml
<p facs="#facs_114_TextRegion_1632680143045_494">
    <lb facs="#facs_114_r1l5" n="N001"/>Um aus diesem Wirrsal herauszukommen, empfiehlt es sich zunächst, der
    <lb facs="#facs_114_r1l6" n="N002"/>Geschichte der Interpretation in ihren wichtigsten Stationen nachzugehen⊂1⊃.
    <lb facs="#facs_114_r1l7" n="N003"/>Der erste, der eine ausführliche Erklärung des Blattes gibt, ist Arend in seiner
    <lb facs="#facs_114_r1l8" n="N004"/>1728 erschienenen Gedächtnisschrift, aus Anlaß des 200jährigen Todestages
    <lb facs="#facs_114_r1l9" n="N005"/>Dürers⊂2⊃...</p>
```

After:

```xml
<p facs="#facs_114_TextRegion_1632680143045_494">
    <lb facs="#facs_114_r1l5" n="N001"/>Um aus diesem Wirrsal herauszukommen, empfiehlt es sich zunächst, der
    <lb facs="#facs_114_r1l6" n="N002"/>Geschichte der Interpretation in ihren wichtigsten Stationen nachzugehen<note type="refnum">1</note>.
    <lb facs="#facs_114_r1l7" n="N003"/>Der erste, der eine ausführliche Erklärung des Blattes gibt, ist Arend in seiner
    <lb facs="#facs_114_r1l8" n="N004"/>1728 erschienenen Gedächtnisschrift, aus Anlaß des 200jährigen Todestages
    <lb facs="#facs_114_r1l9" n="N005"/>Dürers<note type="refnum">2</note>...</p>
```

Dependent code: `replacements.footnote_numbers(xml_string)`  
Default output: `temp/fn1.xml`

### Encode footnote numbers in the footnotes

Footnote numbers in the footer are replaced in a similar manner.

`⊤([0-9]+)⊥` -> `<hi rendition="#sup">\1</hi>`

Before:

```xml
<note place="foot" n="[footnote reference]" facs="#facs_114_TextRegion_1632680135558_482">
    <lb facs="#facs_114_r1l35" n="N001"/>⊤1⊥ Ich sehe dabei ab von Vasari, der kurzweg sagt, die Beschäftigung mit den dargestellten
    <lb facs="#facs_114_r1l36" n="N002"/>Dingen habe die Frau melancholisch gemacht und würde jeden melancholisch machen...</note>
```

After:

```xml
<note place="foot" n="[footnote reference]" facs="#facs_114_TextRegion_1632680135558_482">
    <lb facs="#facs_114_r1l35" n="N001"/><hi rendition="#sup">1</hi> Ich sehe dabei ab von Vasari, der kurzweg sagt, die Beschäftigung mit den dargestellten
    <lb facs="#facs_114_r1l36" n="N002"/>Dingen habe die Frau melancholisch gemacht und würde jeden melancholisch machen...</note>
```

Dependent code: `replacements.footnotes(xml_string)`  
Default output: `temp/fn2.xml`

### Move footnotes to their inline position

With an XML transformation that inserts the footnote at the footnote number and deletes it in the original position,
the footnotes are made inline.

For the transformation, the XML content is parsed before and transformed to a string after again.

Before:

```xml
<div>
    <p facs="#facs_114_TextRegion_1632680143045_494">
        <lb facs="#facs_114_r1l5" n="N001"/>Um aus diesem Wirrsal herauszukommen, empfiehlt es sich zunächst, der
        <lb facs="#facs_114_r1l6" n="N002"/>Geschichte der Interpretation in ihren wichtigsten Stationen nachzugehen<note type="refnum">1</note>.
        <lb facs="#facs_114_r1l7" n="N003"/>Der erste, der eine ausführliche Erklärung des Blattes gibt, ist Arend in seiner
        <lb facs="#facs_114_r1l8" n="N004"/>1728 erschienenen Gedächtnisschrift, aus Anlaß des 200jährigen Todestages
        <lb facs="#facs_114_r1l9" n="N005"/>Dürers<note type="refnum">2</note>...</p>
    <note place="foot" n="[footnote reference]" facs="#facs_114_TextRegion_1632680135558_482">
        <lb facs="#facs_114_r1l35" n="N001"/><hi rendition="#sup">1</hi> Ich sehe dabei ab von Vasari, der kurzweg sagt, die Beschäftigung mit den dargestellten
        <lb facs="#facs_114_r1l36" n="N002"/>Dingen habe die Frau melancholisch gemacht und würde jeden melancholisch machen...</note>
</div>
```

After:

```xml
<div>
    <p facs="#facs_114_TextRegion_1632680143045_494">
        <lb facs="#facs_114_r1l5" n="N001"/>Um aus diesem Wirrsal herauszukommen, empfiehlt es sich zunächst, der
        <lb facs="#facs_114_r1l6" n="N002"/>Geschichte der Interpretation in ihren wichtigsten Stationen nachzugehen<note n="1" xml:id="facs_114_note_1" place="foot" facs="#facs_114_TextRegion_1632680135558_482">
        <lb facs="#facs_114_r1l35" n="N001"/> Ich sehe dabei ab von Vasari, der kurzweg sagt, die Beschäftigung mit den dargestellten
        <lb facs="#facs_114_r1l36" n="N002"/>Dingen habe die Frau melancholisch gemacht und würde jeden melancholisch machen...</note>.
        <lb facs="#facs_114_r1l7" n="N003"/>Der erste, der eine ausführliche Erklärung des Blattes gibt, ist Arend in seiner
        <lb facs="#facs_114_r1l8" n="N004"/>1728 erschienenen Gedächtnisschrift, aus Anlaß des 200jährigen Todestages
        <lb facs="#facs_114_r1l9" n="N005"/>Dürers<note n="2" xml:id="facs_114_note_2" place="foot" facs="#facs_114_TextRegion_1632680134071_478">
        <lb facs="#facs_114_r1l39" n="N001"/> H. C. Arend: Das Gedächtnis der Ehren... A. Dürers 1728 (Goslar).</note>...</p>
</div>
```

Dependent code: `transform.move_footnotes(xml_string)`  
Default output: `temp/fn3.xml`

### Encode small caps

In addition to replacements, small caps are encoded as lower case letters.

`₍([A-Z\-]+)₎` -> `<hi rendition="#k">` + `m.group(1).lower()` + `</hi>`

Before:

```
₍ABC₎
```

After:

```xml
<hi rendition="#k">abc</hi>
```

Dependent code: `replacements.small_caps(xml_string)`  
Default output: `temp/scap.xml`

### Encode document links

A project specific example from 'Raphael in Early Modern Sources':  
Find links to other documents and construct the link target from the link content.

`\[↾([F0-9/\-]+)⇃\]` -> `lambda m: '[<ref target="#doc_' + re.sub(r'/', '_', m.group(1)) + '">' + m.group(1) + '</ref>]'"`

Before:

```xml
<p>[↾1550/4⇃]</p>
```

After:

```xml
<p>[<ref target="#doc_1550_4">1550/4</ref>]</p>
```

Dependent code: `replacements.document_links(xml_string)`  
Default output: `temp/link.xml`

### Encode bold text

The replacements of bold passages does not differ much from that of italics.

`↾([^<>]?)⇃` -> `<hi rendition="#b">\1</hi>"`

Before:

```
↾bold⇃
```

After:

```xml
<hi rendition="#b">bold</hi>
```

Dependent code: `replacements.bold_text(xml_string)`  
Default output: `temp/bold.xml`

### Move page begins into corresponding sections 1

In order to interact properly with pagination, the page begins are moved into the following chapters.

`(<pb facs="#facs_([0-9]+)".*? />\n)(\s+)(</div>\n\s+<div>\n)` -> `\4\3\1`

Before:

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
After:

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

Dependent code: `replacements.move_pb_into_div1(xml_string)`  
Default output: `temp/pb1.xml`

### Move page begins into corresponding sections 2

If printed page numbers are present, they are moved, too.

`(<pb facs="#facs_[0-9]+".*? />\s*<fw.*?>\s*.*?</fw>\s*)(</div>\s*<div>\s*)` -> `\2\1`

Before:

```xml
<div>
    <div>
        <p>Lorem ispum</p>
        <pb facs="#facs_42"/>
        <fw/>
    </div>
    <div>
        <head>Dolor sit.</head>
    </div>
</div>
```
After:

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

Dependent code: `replacements.move_pb_into_div2(xml_string)`  
Default output: `temp/pb2.xml`

### Move page information into `tei:pb`

Most of our projects do not focus on *form work* (fw) and delete it.

If not already present, the page number is copied form the `fw` to the `pb` during this step.

Before:

```xml
<div>
    <fw type="page" place="bottom" facs="#facs_41_TextRegion_1622902436286_1439">
        <lb facs="#facs_41_r1l32" n="N001"/>27</fw>
    <pb facs="#facs_42" n="42" xml:id="img_0042"/>
    <fw type="header" place="top" facs="#facs_42_TextRegion_1624278083468_97">
        <lb facs="#facs_42_r1l1" n="N001"/>GEDANKEN ZUR KUNSTGESCHICHTE</fw>
</div>
```
After:

```xml
<div>
    <pb n="28" facs="#facs_42" xml:id="img_0042"/>
</div>
```

Dependent code: `transform.page_numbers(xml_string)`  
Default output: `temp/pb3.xml`

### Join paragraphs

Paragraphs that still carry the type *continued* are merged over page boundaries.

Before:

```xml
<div>
    <p>Lorem ipsum</p>
    <pb/>
    <p type="continued">dolor sit amet.</p>
</div>
```
After:

```xml
<div>
    <p>Lorem ipsum
        <pb/>
        dolor sit amet.</p>
</div>
```

Dependent code: `transform.join_paragraphs(xml_string)`  
Default output: `temp/rep1.xml`

### Join italics and small caps 1

First step: Replace elements with attributes by plain elements so the start and the end tag encode all information.

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

Dependent code: `transform.simplify_hi(xml_string)`  
Default output: `temp/hi1.xml`

### Join italics and small caps 2

Second step: Delete ending tags in old line and starting tags in new line.

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

Dependent code: `replacements.join_small_caps(xml_string)`  
Default output: `temp/hi2.xml`

### Join italics and small caps 3

Third step: Restore original tag schema.

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

Dependent code: `transform.expant_hi(xml_string)`  
Default output: `temp/hi1.xml`

### Encode hyphenation in TEI way

For full-text indexing and reader-friendly views, hyphenation is detected and encoded in a TEI-native way.

Before:

```xml
<p type="paragraph" facs="#facs_9_TextRegion_1622895576163_258">
    <lb facs="#facs_9_r1l2" n="N001"/>Seit Jahren haben gutmeinende Freunde und vertrauensvolle Verleger mir 
    zu¬
    <lb facs="#facs_9_r1l3" n="N002"/>geredet, 
    einen Band gesammelter Vorträge und Aufsätze herauszugeben.
</p>
```
After:

```xml
<p type="paragraph" facs="#facs_9_TextRegion_1622895576163_258">
    <lb facs="#facs_9_r1l2" n="N001"/>Seit Jahren haben gutmeinende Freunde und vertrauensvolle Verleger mir 
    zu<lb break="no" rend="hyphen" facs="#facs_9_r1l3" n="N002"/>geredet, 
    einen Band gesammelter Vorträge und Aufsätze herauszugeben.</p>
```

Dependent code: `replacements.replace_hyphens(xml_string)`  
Default output: `temp/hyph.xml`

### Hyphenation (details)

The recognition of hyphenations is implemented in a defensive way that attends to resolve only clear-cut cases.
It is based on language specific word lists and a cascade of replacements:

- Find all '¬' and replace them with '-' before numbers or capitals (as in "Merriam-Webster")
- Load a list of known words with hyphens used in the project (or any such list)
- Replace '¬' with '-' in such words and mark the break as rendered without additional hyphen. (Words like "un-ionized")
- Load extensive language specific lists
- Replace '¬' with '-' in the remaining cases if the combined form is a valid word

### Remove `tei:lb`

Remove `lb` elements from the document except in special cases like in headings.

Dependent code: `transform.remove_lb(xml_string)`  
Default output: `temp/removed-lb.xml`

### Remove `tei:pb`

Remove `pb` elements form the document to get only the content.

Dependent code: `transform.remove_pb(xml_string)`  
Default output: `temp/removed-pb.xml`

### Pretty print

Pretty outfile with indentation and breaks specified per element.
To simplify the replacements above, pretty printing is best done at the end.

Dependent code: `transform.indent(xml_string)`  
Default output: `temp/ind.xml`

### Write outfile

The document defined before is written to the outfile specified by the argument `-o`.
