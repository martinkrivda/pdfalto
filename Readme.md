# pdfalto

[![Build Status](https://travis-ci.org/kermitt2/pdfalto.svg?branch=master)](https://travis-ci.org/kermitt2/pdfalto)
[![SWH](https://archive.softwareheritage.org/badge/origin/https://github.com/kermitt2/pdfalto/)](https://archive.softwareheritage.org/browse/origin/https://github.com/kermitt2/pdfalto/)
[![License: GPL v2](https://img.shields.io/badge/License-GPL%20v2-blue.svg)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)

**pdfalto** is a command line executable for parsing PDF files and producing structured XML representations of the PDF content in [ALTO](https://github.com/kermitt2/pdfalto/blob/master/schema/alto.xsd) format, capturing in particular all the layout and style information of the PDF. 

**pdfalto** is initially a fork of [pdf2xml](http://sourceforge.net/projects/pdf2xml), developed at XRCE, with modifications for robustness, addition of features, improved layout element detections, and output enhanced format in [ALTO](https://github.com/altoxml/documentation/wiki) (including in particular space information, useful for instance for further machine learning processing). It is based on the [Xpdf](http://www.xpdfreader.com/) library.  

The latest stable version is *0.4*. Working version (master) is *0.5*.

An Archlinux package for pdfalto is available [here](https://aur.archlinux.org/packages/pdfalto/), thanks to @andreasbaumann. The build process described [below](https://github.com/kermitt2/pdfalto#build) will create a portable standalone pdfalto executable that can be packaged with other tools without further installation requirements for the end-user. 

# Requirements

* compilers : clang 3.6 or gcc 4.9
* makefile generator : cmake 3.12.0
* fetching dependencies : wget

# Usage

General usage is as follow: 

```
Usage: pdfalto [options] <PDF-file> [<xml-file>]
  -f <int>                      : first page to convert
  -l <int>                      : last page to convert
  -verbose                      : display pdf attributes
  -noImage                      : do not extract Images (Bitmap and Vectorial)
  -outline                      : create an outline file xml
  -annotation                   : create an annotations file xml
  -noLineNumbers                : do not output line numbers added in manuscript-style textual documents
  -readingOrder                 : blocks follow the reading order
  -noText                       : do not extract textual objects (might be useful, but non-valid ALTO)
  -charReadingOrderAttr         : include TYPE attribute to String elements to indicate right-to-left reading order (might be useful, but non-valid ALTO)
  -fullFontName                 : fonts names are not normalized
  -nsURI <string>               : add the specified namespace URI
  -opw <string>                 : owner password (for encrypted files)
  -upw <string>                 : user password (for encrypted files)
  -filesLimit <int>             : limit of asset files be extracted
  -q                            : don't print any messages or errors
  -v                            : print version info
  -h                            : print usage information
  -help                         : print usage information
  --help                        : print usage information
  -?                            : print usage information
```

In addition to the [ALTO](https://github.com/altoxml/documentation/wiki) file describing the PDF content, the following files are generated:

* `_metadata.xml` file containing a pdf file metadata (generate metadata information in a separate XML file as ALTO schema does not support that).

* `_annot.xml` file containing a description of the annotations in the PDF (e.g. GOTO, external http links, ...) obtained with `-annotation` option

* `_outline.xml` file containing a possible PDF-embedded table of content (aka outline) obtained with `-outline` option

* `.xml_data/` subdirectory containing the vectorial (.vec) and bitmap images (.png) embedded in the PDF, this is generated by default - when the option `-noImage` is not present. This extraction slows down the process very significantly, so if no image is required, use the option `-noImage`. When the images are not extracted, image elements with layout properties still appear in the ALTO file, but they reference no extracted image files. 

## Extra script to get only text content

The goal of pdfalto is to extract all the content of a PDF, not just text, but also layout, style, font, vector graphics, embedded bitmap, annotation, metadata, and outline information. For convenience and debugging, we provide a simple XSLT to extract only the text content from the produced ALTO XML file. For instance, using `xsltproc` command line, the following outputs the text content only:

> xsltproc schema/alto2txt.xsl alto_file.xml

# Dependencies

Dependencies can be recompiled by running this [script](https://github.com/kermitt2/pdfalto/blob/master/install_deps.sh)

> ./install_deps.sh

The script will download and build the dependencies unders `libs/` and the additional language support packages for xpdf under `languages/`. 

If necessary, see [compiling dependencies procedures](Dependencies_INSTALL.md) for further details.

##### Known issues 
([issue 41](https://github.com/kermitt2/pdfalto/issues/41)) might occur while building, in this case you'll need to compile the dependencies before building pdflato.

# Build

* NOTE for windows : it's recommended to use Cygwin and install standard libraries (either for cland or gcc)
> git clone https://github.com/kermitt2/pdfalto.git && cd pdfalto

* Xpdf-4.03 is shipped as git submodule, to download it: 

> git submodule update --init --recursive

* Build pdfalto:

> cmake .

> make

The executable `pdfalto` is generated in the root directory. Additionally, this will create a static library for xpdf-4.03 at the following path `xpdf-4.03/build/xpdf/lib/libxpdf.a` and all the libraries and their respective subdirectory. 

To use the additional xpdf language support packages, the executable `pdfalto` comes with a config file `xpdfrc` and language resources installed under `languages/`. Both `xpdfrc` and `languages/` must be alongside the executable `pdfalto` to be used. To add `pdfalto` with these additional resources to a third party application (e.g. GROBID), move the executation together with these files: 

```
lopez@work:~$ ls my_pdfalto/
languages  pdfalto  xpdfrc
```

##### Known issues 
([issue #135](https://github.com/kermitt2/pdfalto/issues/135)) on macOS "fontconfig.h file not found" might occur while building, see described workaround.

# Future work

- Text like containing block element characters (https://unicode.org/charts/PDF/U2B00.pdf) are used as placeholders for unknown character unicodes, instead of what would be expected when visually inspecting the text. The reason for these unsolved character unicode values is that the actual characters are glyphs that are embedded in the PDF document which use free unicode range for embedded fonts, not the right unicode. The only way to extract the valid text for those special characters is to use OCR at glyph level . This is our targeted main future enhancement, relying on a custom Deep Learning approach.

- map special characters in secondary fonts to their expected unicode 

- try to optimize speed and memory

- see the issue tracker for further tasks

# Changes

New in version 0.4 (apart various bug fixes):

- support for xpdf language support package for language-specific fonts like Arabic, Chinese-simplified, Japanese, etc. they are pre-installed locally and portable 

- refined line number detection and fixing a bug which could result in random missing numbers in the ALTO output

- update to xpdf-4.03

- fix issue with character spacing due to invalid rotation condition

- update dependencies and dependency install script

New in version 0.3 (apart various bug fixes):

- line number detection: line numbers (typically added for review in manuscripts/preprints) are specifically identified and not anymore mixed with the rest of text content, they will be grouped in a separate block or, optionally, not outputted in the ALTO file (`noLineNumbers` option)

- removal of `-blocks` option, the block information are always returned for ensuring ALTO validation (`<TextBlock>` element)

- bug fixing on reading order

- fix possible incorrect XMax and YMax values at 0 on block coordinates having only one line

New in version 0.2 (apart various bug fixes):

- support Unicode composition of characters

- generalize reading order to all blocks (it was limited to the blocks of the first page)

- detect subscript/superscript text font style attribute

- use SVG as a format for vectorial images

- propagate unsolved character Unicode value (free Unicode range for embedded fonts) as encoded special character in ALTO (so-called "placeholder" approach)

- generate metadata information in a separate XML file (as ALTO schema does not support that)

- use the latest version of xpdf, version 4.00

- add cmake

- [ALTO](https://github.com/altoxml/documentation/wiki) output is replacing custom Xerox XML format

- Note: this released version was used for Grobid release 0.5.6

New in version 0.1 (apart various bug fixes): 

- encode URI (using `xmlURIEscape` from libxml2) for the @href attribute content to avoid blocking XML wellformedness issues. From our experiments, this problem happens in average for 2-3 scholar PDF out of one thousand.

- output coordinates attributes for the BLOCK elements when the `-block` option is selected,

- add a parameter `-readingOrder` which re-order the blocks following the reading order when the -block option is selected. By default in pdf2xml, the elements followed the PDF content stream (the so-called _raw order_). In xpdf, several text flow orders are available including the raw order and the reading order. Note that, with this modification and this new option, only the blocks are re-ordered.

  From our experiments, the raw order can diverge quite significantly from the order of elements according to the visual/reading layout in 2-4% of scholar PDF (e.g. title element is introduced at the end of the page element, while visually present at the top of the page), and minor changes can be present in up to 100% of PDF for some scientific publishers (e.g. headnote introduced at the end of the page content). This additional mode can be thus quite useful for information/structure extraction applications exploiting pdfalto output. 

- use the latest version of xpdf, version 3.04.


# Contributors

Contact: Patrice Lopez (patrice.lopez@science-miner.com)

pdfalto is developed by Patrice Lopez (patrice.lopez@science-miner.com) and Achraf Azhar (achraf.azhar@inria.fr).

pdf2xml is orignally written by Hervé Déjean, Sophie Andrieu, Jean-Yves Vion-Dury and  Emmanuel Giguet (XRCE) under GPL2 license. 

[Xpdf](http://www.xpdfreader.com/) is developed by Glyph & Cog, LLC (1996-2017) and distributed under GPL2 or GPL3 license. 

The windows version has been built originally by [@pboumenot](https://github.com/boumenot) and ported on windows 7 for 64 bit, then for windows (native and cygwin) by [@lfoppiano](https://github.com/lfoppiano) and [@flydutch](https://github.com/flydutch).  


# License

As the original pdf2xml and main dependency Xpdf, pdfalto is distributed under GPL2 license. 

# Useful links

Some tools for converting ALTO into other formats:

- https://github.com/filak/hOCR-to-ALTO
- https://github.com/UB-Mannheim/ocr-fileformat
