## Document segmentation

In order to convert your transcription into a structured document, your page should be properly segmented with a text-area for each paragraph and structural element.
This will allow you apply a structural tag to each element at text-area level.
The basic list of structural tags for a TEI conversion is the standard list available in Transkribus app and includes:
- paragraph // marks each paragraph of text. 
- heading // the chapter/paragraphs headings.
- header // the running header on the top of the page
- footnote // footnotes of the page
- footnote-continued // the part of a footnote that continues from the previous page
- page-number // the number of the page, wherever it could be positioned
- catch-word // the word/phrase at the bottom right of a page with the beginning of the subsequent page
- signature-mark // the mark used to identify a quire or section in bookbinding
- caption // image caption
- TOC-entry // Elements from the Table of Content
- marginalia // notes or titles on the marigins of a page

_Please note_
To have a proper running text split in paragraphs all elements should have a separate textarea and not a line with structural tag "paragraph".  This is also true for footnotes, each footnote must be on a different text-area.
In order to be able to combine paragraphs spanning more than one page, we introduced the _praragraph_continued_ (similar to footnotes) and in general, the "_continued" suffix to each tag that needed it.

Other structure tags, especially for REMS, include
- commentary // the editor comment
- abstract // the abstract of a document
- bibliography // the bibliography of a document
- archival-notice // information about the source of a document
- document-number // id of document
- document-title // title of document

### Tagging existing documents (expert only)
If you have documents with block text recognition but no structure tagging it could be useful to try training a P2PaLA model with about 20 pages of manually tagged areas and apply the model as "Label existing transcriptions->Regions".
Please be aware that each text area/region needs to be visually distict from the others (i.e. paragraph and commentary should use different fonts and font size). Since P2PaLA cannot rely on content, you will probably need to check all "-continued" regions.

### Field models (beta only)
It could be possible to train a field model for region either to recognize the regions or even the regions with structural tags altogether, but at the moment this is experimental and there are no public models. 

###  Process goals
With a series of subsequent XSLT transformations and global REGEX substitutions, the Smart Model and the segmentation make it possible to:
- Use the content of text areas marked as "page_number" as the label for new pages
- Combine the content of footnotes with the reference in the text, thanks to the identity between the superscript numbers marked in the text and in the notes
- Join hyphenated words in the text into whole words. Hyphens at the end of lines in the transcription are marked with the symbol ¬ (U+00AC), but not wanting to risk joining words normally written with a hyphen with an automatic substitution, attempts were made to use dictionaries (not useful for ancient Italian) to reduce errors.
- Link references between documents (indicated as bold numbers beetween square brackets in REMS) with the corresponding documents. Also for this, the presence of groups of documents in the references made it impossible to completely automate the process.



