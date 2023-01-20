---
layout: post
title: "A Map of the Summa Praedicantium"
sub_title: "Navigation using IIIF annotations"
date: 2023-01-14
post_number: 1
excerpt: A project to enable readers to navigate a digitized medieval text using automatically-generated IIIF annotations.
---
## The problem

I want to provide enhanced access to the content of an enormous medieval compendium of materials for preachers. The text was planned to allow easy reference: it consists of topical chapters (Death, Mercy, Penance, etc.), which are divided into numbered articles and numbered paragraphs. Working from a IIIF-enabled digitized incunabulum, I want to make it possible to start with a paragraph reference like "E.4.2" (= chapter E.4 "Equitas", paragraph 2), and leap immediately to that spot in the text.

{% include image.html 
	post_number=page.post_number 
	image_name="tabula-vocalis"
	caption="Entries for \"hawk\" (Accipiter) and \"bee\" (apis) in the index, with paragraph references to various chapters. The printed edition uses roman numerals, but the early manuscripts use arabic numerals." 
	alt="Four lines of text in a gothic font, with several references consisting of a capital letter and two roman numerals." 
%}

## The plan

The printed text has article and paragraph numbers in the margins. If I could enhance the IIIF manifest with an annotation for each marginal reference, I could use those to locate the page on which e.g. the "E.4.2" annotation is found, and highlight that annotation when you land on that page.

## The background

The text is the *Summa Praedicantium*, compiled by the Dominican friar John Bromyard in the early 14th century. In the 1990s, as a postdoctoral fellow at the COMERS Institute at the University of Groningen, I worked on an edition, and completed (or very nearly) the letters A-C, but then failed to get a position that would let me continue it. Recently I looked at my partial text and notes and thought it would be interesting and useful to share a navigable map of the *Summa* along with my list of cited sources with their paragraph references. 

## The manifest

The first requirement was a IIIF-enabled copy of the *Summa*. The best text to use would be one of the two complete early manuscripts, but the British Library and Peterhouse College Cambridge haven't had them scanned yet.[^mss] I therefore looked for copies of the first printed edition, made at Basel by Johannes Amerbach in the 1480s. I found two: one at the National Library of Portugal and one at the University of Basel. Both are good clear scans of complete copies of the text; the deciding difference is the speed of their image servers. Portugal's was overwhelmed by the requests for the 1384 page thumbnails (did I mention that the *Summa* is very large? About a million words, I estimated when I was working on it). So I'm using Basel's copy, which is very responsive. Since I wanted to keep working on upgrading my IIIF knowledge to version 3 of the APIs, I transformed the manifest to v.3 using [IIIF/prezi-2-3](https://github.com/IIIF/prezi-2-to-3).

[^mss]: British Library, MS Royal 7 E iv and Peterhouse College, MSS 24 and 25. For any Bromyard fans out there: do you know about the Köln ms? It wasn't in the literature when I was working on the *Summa*, and I don't think I've seen it in recent scholarship on Bromyard. Juliane Trede, *Die juristischen Handschriften des Stadtarchivs Köln*, Mitteilungen aus dem Stadtarchiv von Köln: Sonderreihe: Die Handschriften des Archivs, Heft 8 (Köln: Historisches Archiv der Stadt Köln, 2005), [pp.10-12](http://bilder.manuscripta-mediaevalia.de/hs//katalogseiten/HSK0556_b010_jpg.htm). Digitized from microfilm (but not IIIF-enabled; click "Open in the Mets Viewer"): [Vol. 1](https://historischesarchivkoeln.de/archive.xhtml?id=Vz++++++90003112PPLS#Vz______90003112PPLS); [Vol. 2](https://historischesarchivkoeln.de/archive.xhtml?id=Vz++++++90003113PPLS#Vz______90003113PPLS); [Vol. 3](https://historischesarchivkoeln.de/archive.xhtml?id=Vz++++++90003114PPLS#Vz______90003114PPLS). N.B. the microfilm frames often seem to be jumbled in the digital presentation.

## Annotating: the manual approach

I started by going through the text and capturing the canvas id of the first page of each of the 189 chapters. This enabled me to enhance the manifest with a chapter-level table of contents, which was a satisfying first step towards a map.

{% include image.html 
	post_number=page.post_number 
	image_name="toc"
	format="png"
	caption="Table of contents from the IIIF manifest, as displayed in Mirador 3" 
	alt="A screenshot showing a table of contents beside part of a printed page" 
%}

The next step was to create annotations for each marginal tag. I set up an instance of [glenrobson/simpleAnnotationServer](https://github.com/glenrobson/SimpleAnnotationServer) and starting creating them by hand: turn to the first page of chapter A.1, draw a box around the "Art. i" in the margin, and label it "A.1 Art.1". I did a couple of chapters this way, but it was clear that a complete map would take much longer than I could reasonably spend on this project, even if I tried to do a few chapters each week.

> **Digression**: if I had gone this way it would have fulfilled the lesson of an exemplum told by Bromyard which has been useful to me down the years: A farmer owned a neglected field that was overgrown with brambles, so one day he told his son to go and clear them. The son went to the field with his hoe, but the brambles were very high and thick, so he just lay down on the ground and slept; and he did likewise the next day, and the next. Eventually his father came to the field to see how the work was going. The chastened son explained what he had been doing. The father said: "If each day you had cleared just enough ground to sleep on, the job would be half finished by now." 

## Annotating: automation

To automate the annotations, the first requirement would be to generate boxes around the marginal notes. I've used [opencv-python](https://pypi.org/project/opencv-python/) a little in the past to optimize images for OCR, so I thought there might be a good example to follow for identifying boxes. This turned out to be right: based on [this tutorial](https://medium.com/pythoneers/text-detection-and-extraction-from-image-with-python-5c0c75a8ff14), I developed this script, which does a decent job of identifying text boxes on a page. TODO: how the script works. The marginal annotations are generally far enough from the text and the edge of the page to be distinct boxes. Here's a sample page, with green boxes around the marginal tags.

{% include image.html 
	post_number=page.post_number 
	image_name="13265309_rectanglebox" 
	caption="A page of the *Summa* with green boxes (generated by my script) around the marginal tags." 
	alt="A page of an early printed book with small marginal annotations boxed in green" 
%}

Using these boxes, along with the canvas ids and my map of the starting pages of the chapters, I could generate annotations assigned to the right canvas and chapter. It would then be a manual process to add the labels to the annotations (but a less time-consuming task than drawing all the boxes by hand).

## Training Tesseract

... or would that even be necessary? Given the very small character set in these tags (roman numbers up to "cli", various abbreviations of "Articulus" and "Exemplum", periods and colons), could I quickly train Tesseract to do a decent job of reading the tags? Find out in part 2.
