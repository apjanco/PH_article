# Finding Places in Text with the World Historical Gazetteer 
Susan Grunewald and Andrew Janco

sgrunewa@alumni.cmu.edu

ajanco@haverford.edu

## 1. Lesson Overview: 
Researchers often need to be able to search a corpus of texts for a defined list of terms. In many cases, historians are interested in certain places named in a text or texts. This lesson details how to programmatically search documents for a list of terms, including place names. First, we produce a CSV file with a row for each occurrence of the term and an HTML file of the text with the terms highlighted. This visualization of the results can be used to interpret the results and to assess their usefulness for a given project.  

In this lesson, readers will use the Python pathlib library to load a directory of files. Using textract, users are able to recognize text in a large range of file types, including pdf, docx, and jpeg files. For those with an existing gazetteer or list of terms, readers will write a function that returns a list of term matches and their locations in a text. For those without a gazetteer, users can use a statistical language model that has been trained to perform named entity recognition (NER). Finally, users will create a TSV file in the Linked Places Format, which can then be uploaded to the [World-Historical Gazetteer](http://whgazetteer.org/) for reconciliation, geocoding, and basic mapping.

This lesson will be useful for anyone wishing to perform NER on a text corpus. Other users may wish to skip the text extraction portion of this lesson and focus solely on the spatial elements of the lesson, that is gazetteer building and using the World Historical Gazetteer. These spatial steps are especially useful for someone looking to create historical maps in a largely point and click interface. We have designed this lesson to show how to combine text analysis with spatial analysis, but understand that some readers may only be interested in one of these two methodologies. We urge you to try both parts of the lesson together if you have time to see how the two can be linked in one project and see how the results of these two parts can be ported into another form of digital analysis. 

Please note that the sample data and context of this lesson constitute an example of multilingual DH. The memoir texts are in German and the list of place names represented German transliterations of Russian names for places across the former Soviet Union circa 1941-1956.

## 2. Historical example:
This lesson is an application of co-author Susan Grunewald's research and serves as a practical use-case of how and why these methods are beneficial to historians. Grunewald has worked to map forced labor camps of German POWs in the Soviet Union during and after the Second World War. The results of her mapping have shown that contrary to popular memory, German POWs in the Soviet Union were sent more commonly to industrial and reconstruction projects in Western Russia rather than Siberia. She then wanted to understand if POW memoirs gave a false misrepresentation of Siberian captivity, which could have helped to create or promulgate this popular memory.

In this lesson, we will use a list of camp names to identify mentions of camps and locations in POW memoirs. We then use our HTML text output file to determine if a mention is direct, "I was taken to Voikovo," or indirect "I heard that Voikovo was in Siberia." This data can then be mapped to demonstrate that not all direct place mentions in POW memoirs were in Siberia. Rather, the term "Siberia" served as a decorative term that framed POWs as victims who had endured harsh conditions and cruelty in an exoticized Soviet East.

## 3. Building a corpus:
For the sake of this lesson, we have compiled a sample dataset of selections from digitized POW memoirs to search for place names. Due to copyright restrictions, these are not the full text of the memoirs but rather snippets from roughly 35 sources. [You can download the text corpus here**insert hyperlink to some repository**]().

To build your own corpus for this exercise, all you need are files saved in .txt format. If you need help building a corpus of .txt files, see [this existing *Programming Historian* tutorial by Andrew Akhlaghi](https://programminghistorian.org/en/lessons/OCR-and-Machine-Translation) for instruction on how to turn PDF files into machine readable text files. 

## 4. Building a gazetteer:
In short, a gazetteer is merely a list of place names. For our example, we are using information from [an encyclopedia about German prisoners of war camps](https://www.worldcat.org/title/orte-des-gewahrsams-von-deutschen-kriegsgefangenen-in-der-sowjetunion-1941-1956-findbuch/oclc/682052025&referer=brief_results) as attested in central Soviet government documents. This particular encyclopedia lists the nearest location (e.g. settlement or city) for each of the roughly 4,000 forced labor POW camps. 

As a quick side note, this encyclopedia is an interesting example of the many layers of languages involved in studying Soviet history. The original source book is in German, meaning it uses German transliteration standards for the Russian Cyrillic alphabet. Generally, the places named in the book represent the contemporary Soviet names. This means that the places might not be called that today or they might be Russified versions of places that are more commonly named in a different way due to post-Soviet identity politics. Finally, some of the place names may still have the same name, but as an added difficulty, the German transliteration is of a Russian transliteration of a local language, such as Armenian, providing extra layers of distortion to the later mapping process.

The gazetteer from this lesson also is an example of a historical gazetteer. That is, these names are those utilized by a particular regime during a specific historical context. Trying to map the places named in the memoirs from this case is not a simple task as the names have changed both in the Soviet and post-Soviet era. By using a resource such as the World Historical Gazetteer, it is possible to have the larger historical gazetteer system serve as a crosswalk between the historical and contemporary names, giving us the ability to easily map these historical locations on a modern map. This process is not foolproof with larger, common geographic information systems such as Google Maps or ArcGIS. 
Users can build their own gazetteer simply by listing places of importance for their study in an Excel spreadsheet and saving the document as a comma-separated value (CSV) file. Note, for this lesson and the World Historical Gazetteer, it is best to only focus on the names of settlements (i.e. towns and cities). The World Historical Gazetteer does not currently support mapping and geolocating for states or countries at this point in time. 

## 5. Finding Places in Text with Python
From your computer’s perspective, text is nothing more than a sequence of characters. If you ask Python to iterate over a snippet of text, you’ll see that it returns just one letter at a time. Note that the index starts at 0, not 1 and that spaces are part of the sequence. 

```python
text = "Siberia has many rivers."
for index, char in enumerate(text):
    print(index, char)
```
```
0  S
1  i
2  b
3  e
4  r
5  i
6  a
7  
8  h
9  a
10 s
11  
12 m
13 a
14 n
15 y
16  
17 r
18 i
19 v
20 e
21 r
22 s
23 .
```

When we ask Python to find a word, say “Siberia,” in a larger text, it is actually searching for a capital “S” followed by “i” “b” and so on. It returns a match only if it finds exactly the right letters in the right order.  When it makes a match, Python’s find() function will return the location of the first character in the sequence. For example:

```python
text = "Berlin is a city in Germany."
text.find("Germany")
```
```
20
```

Keep in mind that computers are very precise and picky.  Any messiness in the text will cause the word to be missed, so `text.find("berlin")` returns -1, which means that the sequence could not be found. You can also accidentally match characters that are part of the sequence, but not part of a word.  Try `text.find("in Ger")`.  You get 17 as the answer because that is the beginning of the “in Ger” sequence, which is present in the text, but isn’t the thing you’d normally want to find. 

While pure Python is sufficient for many tasks, natural language processing (NLP) libraries allow us to work computationally with the text as language. NLP reveals a whole host of linguistic attributes of the text that can be used for analysis.  For example, the machine will know if a word is a noun or a verb with part of speech tagging.  We can find the direct object of a verb to determine who is speaking and the subject of that speech.  NLP gives your programs an instant boost of information that opens new forms of analysis and a greater engagement with the linguistic attributes of text than is often common for most historians.    

Our first NLP task is tokenization. This is where our text is split into meaningful parts (be they lexemes or word tokens). The sentence, “Siberia has many rivers.” can be split into the tokens: <Siberia><has><many><rivers><.>  Note that the ending punctuation is now distinct from the word rivers. The rules for tokenization depend on the language and there are many methods and opinions on how to do it.    

For this lesson, we’ll be using an NLP library called [spaCy](https://spacy.io/). This library focuses on “practical NLP” and is designed to be efficient, simple and works well on a basic laptop.  For these reasons, spaCy can be a good choice for practice-minded historians without a fancy gaming computer or research cluster. However, these tasks can also be completed equally well with similar libraries, such as [NLTK](https://www.nltk.org/) or [Stanza](https://stanfordnlp.github.io/stanza/ner.html).  

Once you’ve run `pip install spacy` [(see this article if you’re new to pip)](https://programminghistorian.org/en/lessons/installing-python-modules-pip), you can now import the object for your language that will have the tokenization rules specific to your language. The spaCy documentation [here](https://spacy.io/usage/models/#languages) lists the currently supported languages and their language codes.

To load the language, you will import it just like any other Python module. For example, `from spacy.lang.de import German` or `from spacy.lang.en import English` 
In Python, this line says to look in the spacy directory, then go into the subfolders lang and de to import the Language object called German from that folder.

We are now able to tokenize our text with the following:
```python
from spacy.lang.de import German
nlp = German()
doc = nlp("Berlin ist eine Stadt in Deutschland.")
for token in doc:
    print(token.i, token.text) 
```
```    
0 Berlin
1 ist
2 eine
3 Stadt
4 in
5 Deutschland
6 .
```
Note that each token now has its own index.

With the language object we can tokenize the text, remove stop words and punctuation, or many other common text processing tasks.  For further information, Ines Montani has created an excellent free [online course](https://course.spacy.io/en/).

Now let’s focus back on the task at hand. We need to load our list of placenames and find where they occur in a text. To do this, let’s start by reading the file with a list of names.  We’ll use Python’s pathlib library, which offers a simple way to read the text or data in a file. In the following example, we import pathlib and use it to open a file called ‘gazetteer.txt’ and load its text.  We then create a Python list of the place names by splitting on the new line character “\n”. This assumes that your file has a line for each place name.  If you’ve used a different format in your file, you may need to split on the comma “,”, tab ”\t” or pipe “|”. To do this, just change the value inside .split() below. 

```python
from pathlib import Path

gazetteer = Path("gazetteer.txt").read_text()
gazetteer = gazetteer.split("\n")
```

At this point, you should be able to `print(gazetteer)` and get a nice list of places:

```python
print(gazetteer) 
```
```
['Armenien', 'Aserbaidshan', 'Aserbaidshen', 'Estland', … ] 
```

>>> Extra Trick: Check the first and last entry in the list to make sure it’s not an empty string "" This will happen when you’ve got an empty row in the file.  Python will treat ‘’ as a place, which is nonsense and not what you’re looking for. If the first entry is "", just remove it by slicing the list `gazetteer = gazetteer[1:]` This snips off the first entry.  To cut the last, use `gazetteer = gazetteer[:-1]` For more on slicing see [this *Programming Historian* tutorial](https://programminghistorian.org/en/lessons/manipulating-strings-in-python#slice). 

### Matching Place Names 
Now that we have a list of place names, let’s find where those terms appear in our texts.  As an example, let’s use this sentence:

```python
text = "Karl-Heinz Quade ist von März 1944 bis August 1948 im Lager 150 in Grjasowez interniert."
```

Karl-Heinz Quade was interned in Camp 150 in Gryazovets from March 1944 to August 1948. Looking at the text, there’s one clear place name Gryazovets, which is a town 450 km from Moscow. We just need to show our computer how to find it (and all the other places we care about). 

```python
from spacy.lang.de import German
from spacy.matcher import Matcher

nlp = German()

doc = nlp(text) #remember text is "Karl-Heinz Quade ist von März...

matcher = Matcher(nlp.vocab)
for place in gazetteer:
    pattern = [{'LOWER': place.lower()}]
    matcher.add(place, None, pattern)

matches = matcher(doc)
for match_id, start, end in matches:
    print(start, end, doc[start:end].text)
```
```
13 14 Grjasowez
```
The matcher will find tokens that match the patterns that we’ve given it.  Note that we’ve changed the place names to all lower case letters so that the search will be case-insensitive. Use “ORTH” instead of ”LOWER” if you want case-sensitive search. Note that we get a list of matches that includes what was matched as well as the start and end indexes of the matched spans or tokens. With Matchers, we are able to search for combinations of more than one word such as “New York City” or “Steamboat Springs.” This is really important because you might have “York”, “New York” and “New York City” in your places list. 

If you’ve ever worked with [regular expressions](https://programminghistorian.org/en/lessons/understanding-regular-expressions), some of this should feel familiar. However, rather than matching on sequences of characters, we’re matching token patterns that can also include part of speech and other linguistic attributes of the text.  As an example, let's also match on “Camp 150” (which is “Lager 150” in German). We’ll add a new pattern that will make a match whenever we have “Lager” followed by a number. 

```python
pattern = [{'LOWER': 'lager'},  #the first token should be ‘lager’
           {'LIKE_NUM': True}] # the second token should be a number 
# Add the pattern to the matcher

matcher.add("LAGER_PATTERN", None, pattern)
``` 
We now see:
```
10 12 Lager 150
13 14 Grjasowez
```

The pattern can be any sequence of tokens and their attributes. For more on how to wield this new superpower, see the [spaCy documentation](https://spacy.io/api/matcher/), the spaCy course and the [Rule-based Matcher Explorer](https://explosion.ai/demos/matcher). 

At this point you may want to know which items appear most frequently.  To get frequencies, you can use Python’s Counter object. In the following cell, we create an empty list and then add the text for each match. The counter will then return the frequency for each term in the list.   
```python
from collections import Counter
count_list = []
for match_id, start, end in matches:
    count_list.append(doc[start:end].text)

counter = Counter(count_list)
counter.most_common(10)
```

```
[('Lager 150', 1), ('Grjasowez', 1)] 
```
Note that couter result is a tuple with the term and count. One way to get these values is:
```
for term, count in counter.most_common(10):
    print(term, count)
```

To save our results, we can create a CSV file that contains all of our matches.  A very common and convenient way to do this is the Pandas library (`pip install pandas`).   

```python
import pandas as pd

data = []
for match_id, start, end in matches:
    data.append({“start”:start, “end”:end, “id”:match_id, “text”:doc[start:end].text})
df = pd.DataFrame(data)
df.to_csv(“my_matches.csv”, index=False)  
```

The final step in this section is to export our matches in the [tab separated value (TSV) format required by the World Historical Gazetteer](https://github.com/LinkedPasts/linked-places). A TSV file is just formatted text, so we’ll create the file manually using \t to add tab separators and \n to end each line. 

```python
output_text = ""
column_header = "id \t title \t title_source \t start \t end  \n"  
output_text += column_header  
# get the unique place names by creating a list of names and then converting the list to a set
places_list = [ doc[start:end].text for match_id, start, end in matches ]
unique_places = set(places_list)
start_date = 1800
end_date = 2000
source_title = “Karl-Heinz Quade Diary”
for id, place in enumerate(unique_places): 
    output_text += f"{id} \t {place} \t {source_title} \t {start_date} \t {end_date} \n"
Path("quade_diary_places.tsv").write_text(output_text)
```

### Reformatting for Linked Open Data
Once you have an output file that lists which places are named in the corpus, it is possible to reformat the list into a particular data standard for use in an online system. Will will be using a specific linked open data (or LODLAM) format known as Linked Places Tab Separated Value (TSV) for this process. In brief, linked open data is a set of best practices for web publishing data, which allows interoperability between projects and systems. 

Linked Places TSV is an attempt by spatial digital historians to create a data standard for the type of research we are performing in this lesson. This standard is an attempt to make a linked open data uniform formatting system so that researchers can easily share their data with other digital analysis platforms, codes, APIs, etc. You can see the [Linked Places Github](https://github.com/LinkedPasts/linked-places), part of the larger Linked Pasts project, for more details.

Download the blank but formatted sample Linked Places TSV here [**INSERT LINK TO REPOSITORY**]. You will notice that there are a handful of columns in this sample sheet. These are the mandatory columns required for the World Historical Gazetteer.

To double check, here is a properly formatted Linked Places TSV for the lesson dataset. Download it here to check your progress or for use in the next part of the lesson [**INSERT LINK TO REPOSITORY/FILE**]. 

## 6. Uploading to the World Historical Gazetteer
Now that we have a Linked Places TSV, we will upload it to the [World Historical Gazetteer (WHG)](http://whgazetteer.org/). The WHG is a fully web based application. It indexes place references drawn from historical sources, adding temporal depth to a core of approximately 1.8 million modern records. This is especially useful for places whose names have changed over time. By using the WHG, users can upload their data and rapidly find the coordinates of historical places. As mentioned in the gazetteer section above, this service provides automatic geocoding that is suitable for use with historical data. Many common geocoding services such as Google Maps or those behind a paywall barrier such as through ArcGIS are unsuitable for historical research as they are based primarily upon 21st century information. They rarely support historical name information beyond the most common instances. Additionally, the WHG also supports a multitude of languages. Finally, geocoding and basic mapping are achievable through a graphical user interface. This circumnavigates the need to use a script, to trace layers from maps in a different program, or create relational databases and table joins in GIS software.

The WHG is free to use. To get started, register for an account and then sign in. You will be taken to the “Datasets” tab. Press “add new” to upload your Linked Pasts TSV to the WHG. Insert a title, such as “POW Memoir Places,” and a brief description. Do not check the public box for now, as we do not want this dataset to be visible to anyone by your own user account. You can in the future upload your historic place name information to the WHG if you desire to contribute to the project. Next, browse for you Linked Places TSV and upload it. Do not change the formatting selection. It must remain as Delimited/Spreadsheet.

Once the dataset is successfully uploaded, you can begin what is known as the reconciliation process. Back on the “Data” tab, click on the TSV you uploaded. This will take you to a new screen to initially view the dataset’s metadata. Click on the “Reconciliation” tab at the top of this page. Reconciliation is the process of linking your TSV entries to the database of named place entities and their additional relations in the WHG. On the “Reconciliation” tab, press the “add new task” button. You currently have the option to reconcile your data with Getty TGN or Wikidata. Select one of the two options and press start. If you want to limit the geographical area of your results, apply a geographic constraint filter before pressing start. If none of the pre-defined regions are acceptable, you can create your own user area to fine tune the results given in the reconciliation process. 

After pressing Start, you will be returned to the main “Reconciliation” tab. The process will run for a while and you will receive an email when the process is complete. Your next step is to review the results. You will be told how many locations need to be reviewed for each pass of the reconciliation process. Press the “Review” button next to each of the passes that require review. You will be taken to a new screen that asks you to match your record with records in the WHG. You will be given a choice of potential matches on the right hand side of the screen. Hover over them and they will appear as green flashes on the map illustration screen. When you find the one (note - there should only be one and the type should match, that is look for something labeled as a settlement and not a state or some other type of geographic entity), select that it is a close match and hit save. You will then move onto the next record. Once all of the potential matches from the review levels have been checked, you should go back to the “Data” page.

On the “Data” page, click on your dataset. In the new page that opens, click on “Browse” to get a rough rendering of your dataset on a map. You can download a PNG of the map by pressing the download icon that appears in the top left corner of the map. 

## 7. Future Mapping Work and Suggested Further Lessons
If you return to the “Metadata” tab, you can download an augmented dataset in Linked Places format. This augmented dataset will have the geographic coordinates (latitudes and longitudes) for your places. You can then use the augmented dataset in desktop or web-based mapping applications such as [QGIS](https://www.qgis.org/en/site/) or [ArcGIS Online](https://www.arcgis.com/index.html) to undertake more advanced geographic information system (GIS) spatial analysis. In these programs, you can change the map visualizations, perform analysis, or make a multimedia web mapping project. We highly suggest you look at the additional Programming Historian mapping lessons on [Installing QGIS and Adding Layers](https://programminghistorian.org/en/lessons/qgis-layers) as well as [Creating Vector Layers in QGIS](https://programminghistorian.org/en/lessons/vector-layers-qgis) to see how you can use the results of this lesson to carry out further analysis. 


