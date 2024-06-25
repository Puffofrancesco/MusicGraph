# MusicGraph
Knowledge Graph about music. Developed for a Knowledge Representation and Reasoning course

# 1 Datasets and Task
This project consists in the transformation of a set of two tabular datasets into an informative
knowledge graph enriched with information retrieved from multiple resources.
The two starting dataset were:
<ul>
  <li> Rolling Stone's 500 Greatest Albums of All Time
(https://www.kaggle.com/datasets/notgibs/500-greatest-albums-of-all-time-rolling-stone)
A publication of Rolling Stone in 2012 containing the top 500 albums of all time
based on votes from selected rock musicians, critics, and industry figures. In
particular this dataset contains information about: position on the list, year of release,
album name, artist name, genre name and subgenre name. The albums are taken
from the MusicBrainz database</li>
<li> Top 5000 Albums of All Time Dataset
(https://www.kaggle.com/datasets/michaelbryantds/top-5000-albums-of-all-time-ratey
ourmusiccom)
Containing the top 5000 albums of all time as ranked by the users of the website
rateyourmusic.com . The dataset included information about: ranking, album name,
artist name, release date, genres, descriptors, average rating, number of ratings, and
number of reviews.</li>
</ul>

Both the datasets were focused on a music chart but provided different information and were
of significantly different dimensions both in row size and column size.
However the goal of the project was to join the information coming from the two given tabular
datasets and to use it in order to create an RDF knowledge graph. This was not meant to be
done in a strictly closed local environment, instead the second main point of the project was
to connect the initial graph with external knowledge repositories (in particular with DBPedia’s
knowledge graph and with Wikipedia).
Once the knowledge graph had been extended, it was required to include an ontology, using
either the RDFS schema or the OWL one, and to materialize the inference that could be
made adding it to the enriched graph.
During the whole process I think that the main focus was to emphasize the meaning of the
data, indeed the final step was to run queries on the graph that showed how the starting
datasets were enriched and how it can be exploited to gather useful information.
The libraries I used to develop this project are (more information in the README.txt file):
<ul>
<li> RDFLib </li>
<li> SPARQLWrapper </li>
<li> Pandas </li>
<li> wikipediaapi </li>
<li> owlrl </li>
</ul>

# 2 Workflow
Starting to analyze the given datasets it was clear that, while sharing the same goal and
even having some content in common, there are some noticeable differences in their
schemas. For example the datasets had 336 albums in common.
However most of the shared albums, due to the different ranking criteria, occupied a different
position in the two sets. Unfortunately only the second dataset (the rateyourmusic.com one)
had the criteria specified in the tabular data, so it was impossible to make a comparison
between the two rankings. This made me choose not to focus on the position of each album
in the ranking and instead to approach the datasets just as unordered sets of data about
them.
My first step into the problem was merging the two dataset, I decided to do so by keeping six
columns: Release Date (which actually is the year), Album, Artist, Genre, Subgenres,
Descriptors, creating a final pandas dataframe of this kind:
Obviously though there are two columns which are not shared by the two dataframes, so,
the column “Subgenre” and the column “Descriptors” have a lot of “NaN” valued rows, that
are rows with null values.
Then, in order to make the dataframe easier to work with, I modified the data in the
columnes, removing the spaces and the special characters from the strings and making the
“Genres” and “Subgenre” columns into columns of lists.
This process was carried out iterating through the rows of the dataframe and, initially, storing
the data of each column in a variable. Then, the creation of the local graph and the
integration with the DBPedia’s one are done simultaneously; for each entity, indeed, while
adding the triples related to it to the graph, the function “check_entity_presence” queries
DBPedia to verify the existence of the entity existence in its graph. According to the output of
the “check_entity_presence” function the entity was inserted into the correct namespace,
either the example namespace which I used for the local entities or the DBPedia’s one.
Then, I started working with all the artists that I found in DBPedia in order to find which of
them are bands and to query DBPedia so as to retrieve the city and the country where the
band was founded, all the members of each band and their city and country of origin.
Afterwards I enriched the graph with data gathered from wikipedia, in particular, focusing on
the artists who make up the bands in the graph, I looked for their role in their band inside the
summary of their wikipedia page. The main problem developing this task was extracting
information from unstructured data, I approached this problem in two ways. At first I wanted
to exploit OpenAI’s API in order to make a call to GPT3.5 that returned the keywords found.
However, not having a proper OpenAI’s billing plan I decided to go for a simpler yet less
efficient solution, matching a list of keywords with the content of the summary and adding
those that could be found as the role of the artist.
Subsequently I created an ontology for the graph, creating the proper classes and adding
domains and ranges for each property. This was done using the OWL Schema even if the
RDFS one would have been enough but I thought, for a purpose of scalability, that
implementing already the classes as OWL classes would make much easier to, later on, add
complex concepts and to expand the ontology without the need to change anything
retroactively.
Lastly I materialized the inference derived from the ontology using the owlrl python library.
At this point the graph was completed and ready to be queried and explored.

# 3 RDF Knowledge Graph .
The knowledge graph contains 11 classes and 11 properties.
All the classes are implemented locally in the example domain and they are:
<ul>
<li> MusicAlbum</li>
<li> Person</li>
<li> Artist, subclass of the Person class</li>
<li> Band, subclass of the Artist class</li>
<li> BandMember, subclass of the Artist class</li>
<li> Genre</li>
<li> SubGenre, subclass of the Genre class</li>
<li> Descriptor</li>
<li> City</li>
<li> Country</li>
<li> BandRole</li>
</ul>
For the properties, instead, when it was possible, I used those already present in the
DBPedia’s knowledge graph, else I implemented them locally in the example domain. The
properties are:
<ul>
<li> dbp:artist, connects each album to its artist </li>
<li> dbp:released, connects each album to its release date, the date is implemented as a
Literal of type date</li>
<li> dbp:genre, connects each album to one of its genres</li>
<li> dbo:MusicSubgenre, connects a subgenre to its genre</li>
<li> ex:descriptor, connects an album to one of its descriptors</li>
<li> ex:foundingCity, connects a band to the city in which it was founded</li>
<li> ex:foundingCountry, connects a band to the country in which it was founded</li>
<li> dbo:bandMember, connects a Band to its members</li>
<li> ex:hasRole, connects the member of a band to its role in the band</li>
<li> dbo:birthPlace, connects an artist to his or her hometown</li>
<li> dbo:birthCountry, connects an artist to his or her country of origin</li>
</ul>
In order to show how the knowledge graph can be exploited and how the initial data has
been enriched I ran some queries on it.
For example, I found that the year with the greatest number of albums in the dataset is 1971,
with 160 albums; this means that 1971 may be the most influential year from the musical
point of view. To find this out I queried for the number of albums released each year
Another interesting query shows the most common genre in each country. To discover this I
counted the number of albums of each genre made by bands founded in a single country.
This process was then repeated for each country, selecting for each of them the genre with
greatest number of occurrences.
More queries are available in the python notebook.

# 4 Conclusion
It is impossible though to overlook the limitations of my project that may not lead to an
optimal experience.
For instance I think that creating at first a complete local knowledge graph in the example
namespace or in another private domain and then, once completed, connect it with the
entities in DBPedia using the owl:sameAs predicate would have led to a better result.
Indeed, I think queries would be easier both to write and to compute and the graph would be
more complete and organized. My solution, though, is faster in generating the graph and I
preferred this advantage due to time constraints and replicability of the notebook while
accepting to have a less well organized graph.
Another addition I would like to do to my project is the linkage of the entities of the
knowledge graph with the music MusicBrainz database, exploiting the source of the first
dataset proposed for the project.
Also, I think adding the positioning of each album (where possible) in both the proposed
dataset could lead to interesting insight, such as how the different ranking approaches result
in divergent conclusions. I decided not to implement this in my project to focus on the most
interesting information and not to over complicate it (it already needs around 7 hours to run).
Moreover, I would like to improve the process of knowledge extraction from wikipedia,
creating a more efficient function for keyword recognition and others to retrieve different
information from the complete wikipedia page without limiting the program to the summary.
Having more information in the knowledge graph it would be helpful to expand the ontology
with complex concepts
