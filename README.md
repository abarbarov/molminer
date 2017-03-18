# MolMiner
MolMiner is a library and command-line interface for extracting compounds (called "_chemical entities_") from scientific literature. It's written in Python (currently supporting only Python 3). It should work on all platforms, but problem is that some dependencies are very hard to compile on Windows. Actually it's a wrapper around several open-source tools for chemical information retrieval, namely [ChemSpot][1], [OSRA][2] and [OPSIN][3], using their command-line interface and adding some extended functionality.
# Overview
MolMiner is able to extract chemical entities from scientific literature in various formats including PDF and scanned images. It extracts entities both from text and 2D structures. Text is normalized using part of code from [ChemDataExtractor](https://github.com/mcs07/ChemDataExtractor/blob/master/chemdataextractor/text/normalize.py). Text entities are assigned by [ChemSpot][1] to one of classes: "SYSTEMATIC", "IDENTIFIER", "FORMULA", "TRIVIAL", "ABBREVIATION", "FAMILY", "MULTIPLE". IUPAC names are converted to computer-readable format like SMILES or InChI with [OPSIN][3]. 2D stuctures are recognised in document and converted to computer-readable format with [OSRA][2]. Entities successfully converted to computer-readable format are standardized using [MolVS](https://github.com/mcs07/MolVS) library. Entities are also annotated in PubChem and ChemSpider databases using [PubChemPy](https://github.com/mcs07/PubChemPy) and [ChemSpiPy](https://github.com/mcs07/ChemSpiPy). For processing of PDF files is used [GraphicsMagick][4] and for OCR [Tesseract][4].
# Installation
MolMiner self is written in Python, but it uses several binaries and some of them have complicated compilation dependencies. So the easiest way is to install MolMiner including dependencies as a [conda][6] package hosted on [Anaconda Cloud](https://anaconda.org/).

To install MolMiner without dependencies just download this repository and run `$ python setup.py install`. MolMiner will be then available from shell as `molminer` and also as a Python library.

## Conda package (currently only for linux64)
[Conda][6] is a package, dependency and environment management for any language including Python. MolMiner package includes precompiled dependencies and data files. It also manages all the needed envinronment variables and enables bash auto-completion.

**TO BE DONE**

<!---
1. Download _conda_ from https://conda.io/miniconda.html
2. Create new environment: `$ conda create -n my_new_env python=3`
3. Activate environment: `$ source activate my_new_env`
4. Install MolMiner: `$ conda install -c lich molminer`
5. Use MolMiner: `$ molminer --help`
--->
## From source (linux)
### Binaries
You need all these binaries for MolMiner. They should be installed so path to them is in `$PATH` environment variable (like `/usr/local/bin`). I haven't tried to compile these dependencies on Windows, but that doesn't mean it's impossible.
- [OSRA][2]. This is probably the most complicated binary. Official information is [here](https://sourceforge.net/p/osra/wiki/Dependencies/) and [here](https://github.com/gorgitko/molminer/blob/master/docs/osra-readme.txt). My installation notes are [here](https://github.com/gorgitko/molminer/blob/master/docs/osra-installation.txt).
  - Compile GraphicsMagick with as many supported image formats as possible ([dependencies](http://wiki.octave.org/GraphicsMagick#Main_dependencies)). It's also used for converting PDF to images and for image editing/transformation.
  - Use Tesseract version 4 and up.
  - [Patched version](https://sourceforge.net/projects/osra/files/openbabel-patched/) of OpenBabel is needed.
  - Put OSRA data files (`spelling.txt`, `superatom.txt`) to some directory and add this directory to `$OSRA_DATA_PATH` environment variable.
- [ChemSpot][1]. Just download it and:
  - Put ChemSpot JAR file to directory accesible from `$PATH` and rename it to `chemspot.jar`.
  - Also put there [this bash script](https://github.com/gorgitko/molminer/blob/master/scripts/chemspot). It's used for running ChemSpot. Its first argument is maximum amount of memory for ChemSpot process. Subsequent arguments are forwarded to ChemSpot CLI.
  - Put ChemSpot data files (`dict.zip`, `ids.zip`, `multiclass.bin`) to some directory and add this directory to `$CHEMSPOT_DATA_PATH` environment variable.
- [OPSIN][3]. Just download it and:
  - Put OPSIN JAR file to directory accesible from `$PATH` and rename it to `opsin.jar`.
  - Also put there [this bash script](https://github.com/gorgitko/molminer/blob/master/scripts/opsin). It's used for running ChemSpot. All arguments are forwarded to OPSIN CLI.
- [GraphicsMagick][4]. OSRA needs it for compilation, but it's binary is also directly used by MolMiner. Compile it with as many supported image formats as possible ([dependencies](http://wiki.octave.org/GraphicsMagick#Main_dependencies).
- [Tesseract][5]. OSRA needs it for compilation, but it's binary is also directly used by MolMiner. Use version 4 and up.
  - Tesseract needs language data files. Download them [here](https://github.com/tesseract-ocr/tessdata), put them to some directory and this directory to `$TESSDATA_PREFIX` environment variable.
- [poppler-utils](https://en.wikipedia.org/wiki/Poppler_(software)#poppler-utils). Utils for PDF files built on top of [Poppler](https://poppler.freedesktop.org/) library.
  - Ubuntu (or any OS with `apt` packaging): `$ sudo apt-get install poppler-utils`
- [libmagic](https://github.com/threatstack/libmagic). Reads the magic bytes of file and determine its MIME type.
  - Ubuntu (or any OS with `apt` packaging): `$ sudo apt-get install libmagic1 libmagic-dev`
  
Paths to data files can be also specified in both MolMiner CLI and library, but defining them in environment variables is the easiest way.
### Python dependencies
Dependencies listed in `setup.py` will be installed automatically when you run `$ python setup.py install`. Unfortunately, there is a complicated dependency [RDKit](http://www.rdkit.org/). It's best to install it as a [conda package](https://anaconda.org/rdkit/rdkit).

# Usage
Basic syntax is: `molminer [OPTIONS] COMMAND [ARGS]`

MolMiner has four commands (you can view them with `$ molminer --help`):
- `ocsr`: Extract 2D structures with OSRA. OCSR stands for _Optical Chemical Structure Recognition_.
- `ner`: Extract textual chemical entities with ChemSpot. NER stands for _Named Entity Recognition_.
- `convert`: Convert IUPAC names to computer-readable format with OPSIN.
- `extract`: Combine all the previous commands.

To each command you can view its options with `$ molminer COMMAND --help`

## Input
- Input can be single PDF, image or text file. Type of input file will be automatically determined, but you can specify it with `-i [pdf|pdf_scan|image|text]` option (`text` value is of course not supported by OSRA, resp. `ocsr` command). Only PDF containing scanned papers cannot be identified so you must pass `-i pdf_scan` option.
- Input from `stdin` is also supported. You can use it together with `ner` and `convert` command. For `convert` a list of IUPAC names is expected, each name on single line.
- If you know that your text is paged, i.e. contains page separators -- ASCII control character 12 (Form Feed, '\f'), you can pass `--paged-text` flag and to each entity will be assigned page. This is automatically done when input is PDF file.

## Output
- Result is a CSV file. Defaultly, MolMiner will write result to `stdout`. If you want to write result directly to file, use `-o <file>` option. To change CSV file delimiter use `-d <delimiter>` option.
- Chemical entities, which were successfully converted to computer-readable format, can be also written to SDF file by specifying `--sdf-output <file>` option. If you don't want to create new SDF file and just append to it, use `--sdf-append` flag.
- When using `extract` command, you can also output CSV files separately from OSRA, ChemSpot and OPSIN by using the `--separated-output` flag.

## Defaultly enabled features
By default, these features are enabled:
- Conversion of PDF files to temporary PNG images using GraphicsMagick (GM). OSRA itself can handle PDF files, but using this is more reliable, because OSRA (v2.1.0) is showing wrong information when converting directly from PDF (namely: coordinates, bond length and possibly more ones) and also there are sometimes incorrectly recognised structures. Also it seems that this is sometimes a little bit faster (internally each temporary image is processed in parallel and results are then joined). Use `--no-use-gm` flag to disable it.
- Standardization of chemical entities converted to computer-readable format. See [MolVS documentation](http://molvs.readthedocs.io/en/latest/guide/standardize.html) for explanation. Use `--no-standardization` flag to disable it.
- Annotation of chemical entities in PubChem and ChemSpider. This will try to assign compound IDs by searching separately with different identifiers (entity name, SMILES etc.). If single result is found by searching with entity name, missing indentifiers are added. InChI-key is preffered in searching. To annotate using ChemSpider you need ChemSpider API token. You can get it by signing up on their [website](http://www.chemspider.com/). Then provide this token with `--chemspider-token <token>` option.
- Normalization of text. This is strongly recommended to keep as is, because sometimes is ChemSpot producing weird and unparsable results. Use `--no-normalize-text` flag to disable it.
- Parallel processing will use all available cores. Use `-j <#cores>` option to change it. '-1' to use all CPU cores. '-2' to use all CPU cores minus one.

# MolMiner library
[Autogenerated API documentation][7]

## Wrapper classes
For each of OSRA, ChemSpot and OPSIN, there is a wrapper class. Some general options can be set in constructor. Each class has a `process()` method which takes path to input file or string and do the desired job with it. It returns an OrderedDict instance with results and can also write a CSV file. Classes, methods and their parameters are well documented in [autogenerated API documentation][7].

Example:

```python
from molminer import OSRA
from pprint import pprint

osra = OSRA(unpaper=2)
extracted = osra.process("path/to/document.pdf", output_file="path/to/output.csv", output_formats=["smiles", "inchi"],
                         chemspider_token="your-chemspider-token")
pprint(extracted)
```

## [Extractor class](https://gorgitko.github.io/molminer/molminer.html#module-molminer.Extractor)
This class combines OSRA, ChemSpot and OPSIN to extract chemical entities both from text and 2D structures. It has the same interface as wrapper classes. To constructor you can pass dicts with key-values mapping to wrapper classes constructor's named arguments.

Example:

```python
from molminer import Extractor
from pprint import pprint

extractor = Extractor(chemspot_options={"max_memory": 16})
extracted = extractor.process("path/to/document.pdf")
pprint(extracted)
```

# Notes
- ChemSpot itself is very memory-consuming so dictionary and ID lookup is disabled by default. Only CRF, OpenNLP sentence and multiclass models will be used by default. Maximum memory used by Java process is set to 8 GB by default. It is strongly recommended to use swap file on SSD disk when available memory is under 8 GB (see https://www.digitalocean.com/community/tutorials/how-to-add-swap-space-on-ubuntu-16-04 for more details). If you want to use dictionary and ID lookup in ChemSpot, pass `--chs-dict dict.zip` and `--chs-ids ids.zip` options. If you are using MolMiner library, pass `path_to_dict="dict.zip"` and `path_to_ids="ids.zip"` to ChemSpot class constructor.
- Unfortunately, there wasn't enough time to write unit tests. I hope I will find time in future to do it.
- We also wanted to test MolMiner's quality. That means mainly the completeness of extraction and ratio of false positives. Unfortunately, there aren't complex test data which will cover both textual and 2D structure chemical entities. We don't have enough time to prepare such a complex dataset manually, so for now you can separately look at [ChemSpot][1] and [OSRA](https://sourceforge.net/p/osra/wiki/Validation/) test results.

# Support
Feel free to open issue ticket. This is the prefered way since we don't have mailing list.

# Acknowledgement
MolMiner was the job description of my diploma thesis at [University of chemical technology in Prague](http://www.vscht.cz/?jazyk=en), [Laboratory of chemistry and informatics](http://ich.vscht.cz/). I would like to thank my supervisor Daniel Svozil for leading the work and Martin Sicho for helping me with conda distribution of MolMiner.

Citations of used software:
- Rocktäschel, T., Weidlich, M., and Leser, U. (2012).  ChemSpot: A Hybrid System for Chemical Named Entity Recognition. Bioinformatics 28 (12): 1633-1640.
- "Optical Structure Recognition Software To Recover Chemical Information: OSRA, An Open Source Solution" J. Chem. Inf. Model., 2009, 49 (3), pp 740–743.
- Chemical Name to Structure: OPSIN, an Open Source Solution Daniel. M. Lowe, Peter T. Corbett, Peter Murray-Rust, Robert C. Glen, Journal of Chemical Information and Modeling 2011 51 (3), 739-753

[1]: https://www.informatik.hu-berlin.de/de/forschung/gebiete/wbi/resources/chemspot/chemspot
[2]: https://sourceforge.net/p/osra/wiki/Home/
[3]: https://bitbucket.org/dan2097/opsin
[4]: http://www.graphicsmagick.org/
[5]: https://github.com/tesseract-ocr/tesseract
[6]: https://conda.io/docs/index.html
[7]: https://gorgitko.github.io/molminer/
