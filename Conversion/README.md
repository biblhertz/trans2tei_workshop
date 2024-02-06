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

