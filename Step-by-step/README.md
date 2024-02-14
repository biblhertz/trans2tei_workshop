# Step by step conversion

## Installing the necessary programs

Clone the GitHub repo `biblhertz/trans2tei` recursively to get the dependencies:

```shell
git clone --recurse-submodules git@github.com:biblhertz/trans2tei.git
```

Paste a copy of [Saxon-HE](https://github.com/Saxonica/Saxon-HE/) into the root directory of `trans2tei`.
If the version is different from `saxon-he-10.5.jar`, adjust the variable `SAXON_JAR` in `transform.py`.

Make sure the following directories exist in `trans2tei`: 
`export` for Transkribus exports, `out` for outputs and `temp`for temporary files for debugging. (Get a [temporarily available example](https://transkribus.eu/export/2035342909945879992/export_job_7935643.zip))

In order to annotate hyphenations with proper TEI, it is advisable to add extensive wordlists in `language_data/de.txt`
and `language_data/en.txt`.
For languages other than English, you only need the corresponding list and an entry in `replacements.py` – `replace_hyphens`.
In `hyphen-words.txt`, you can add words with hyphens or proper names with a similar pattern from your corpus.

## Export your document from Transkribus

Start an export in Transkribus, using the PAGE format. For the conversion, you will not need the images.

Unzip the export and copy the directory with the document name to `exports`, so you have a `mets.xml` file at:  
`exports/<Name of your document>/mets.xml`

## Select the conversion that suits your document best

Generic candidates are:

- simplify.py: for an output without connection to the original pages and lines
- introduction.py (originally for REMS): Introductory text with no deep hierarchy
- gedanken.py: for Wölfflin's book *Gedanken zur Kunstgeschichte* (1941)

## Run your conversion

```shell
python3 introduction.py -i exports/your_export/mets.xml -o out/your_output.xml 
```

or 

```shell
python simplify.py -i exports/your_export/mets.xml -o out/your_output.xml
```

... depending on your environment and chosen conversion.

If nothing breaks the document hierarchy you should have your output file in the right format.
Possible causes for errors are:

- mismatched special character tags due to omission
- overlapping markup inbetween special character tags or with existing XML elements

The files in `temp` can be used to locate problems in the most recent conversion.

The exact steps are described in [Conversion](../Conversion/README.md).

## Manual cleanup

Some phenomenons have to be cleaned up after running the workflows in `gedanken.py` or `simplify.py`:

- Hierarchy of chapters: Usually this is a simple task that only involves some movement of closing brackets `</div>`.
  For our examples, fixing this was not a lot more work than doing it manually.
- Linking: This kind of data is virtually unrestricted and will need project-specific treatment.

