
<!-- README.md is generated from README.Rmd. Please edit that file -->

# ConversationAlign

October 2023: **ConversationAlign works but is still under
development!** <br/> Proceed with caution. Cross-reference each
processing stage against your original transcripts. <br/> Email
<jamie_reilly@temple.edu> before jumping in. He will help you! <br/>

<img src="man/figures/convo.jpeg" height="300"
alt="two people conversing" /> <br/>

# Overview

ConversationAlign analyzes alignment between interlocutors (conversation
partners) engaged in two-person conversations. ConversationAlign works
on language transcripts. It can handle text files (.txt) or comma
separated value (.csv) spreadsheet style files. ConversationAlign
transforms raw language data into simultaneous time series objects
spanning 30 possible dimensions via an embedded lookup database. Here’s
a schematic of the guts of the program… <br/>

<figure>
<img src="man/figures/overview.png" height="600"
alt="overview of ConversationAlign" />
<figcaption aria-hidden="true">overview of
ConversationAlign</figcaption>
</figure>

# Before starting: Prep your data

ConversationAlign can handle a home brew of your own preferred format.
However, your transcripts must have the following header columns at a
bare minimum: <br/> 1) Participant identifier (named Interlocutor’,
‘Speaker’, or ‘Participant’) <br/> 2) Text (named ‘Text’, ‘Utterance’,
or ‘Turn’) <br/>

The order of your columns does not matter. Any other data in your
transcripts (e.g., metadata, timestamps, grouping variables) will be
retained. Here’s an example of a transcript that will work. Don’t worry
about stripping punctuation. We will do that for you. <br/>

<img src="man/figures/example1_ts_edg.jpeg" height="200"
alt="Example transcript from Taylor Swift-Ellen DeGeneres Interview, 2013" />
<br/>

Considerations in prepping your language transcripts for
ConversationAlign: <br/>

1)  Save each conversation transcript as a separate file (e.g.,
    MaryJoe_FirstDateTalk.txt)
2)  Be careful/deliberate about your filenaming convention. The filename
    for each conversation will become its event ID in the dataframe
3)  Move all your language transcripts to be analyzed into one folder
    (e.g., “my_transcripts”) in the same directory you are running your
    R script.
4)  If you have metadata (e.g., age, timestamps, grouping variables),
    you can either append this to your original transcript or merge the
    metdata as a separate file. This is a useful option when you have
    many individual difference and demographic details.

# Installation

Install the development version of ConversationAlign from
[GitHub](https://github.com/) by entering the following in your console
or script:

``` r
install.packages("devtools")
devtools::install_github("Reilly-ConceptsCognitionLab/ConversationAlign")
```

# Read your transcripts into R

## read_dyads()

This function will read all your files and concatenate them into a
single dataframe, appending document IDs. You can call this dataframe
whatever you like. ‘read_dyads’ will default to reading all csv and txt
files in a folder called my_transcripts. Just remember that when you are
finished processing a set of transcripts, make sure to move them out of
that folder. You can think of ‘my_transcripts’ as a staging area for
loading data into ConversationAlign. <br/>

``` r
MyRawLangSamples <- read_dyads()
#if you want to specify a different folder, supply your own path
MyRawLangSamples <- read_dyads("/my_custompath")
```

<br/>

<img src="man/figures/example2_ts_edg.jpeg" height="200"
alt="Example of read transcripts from Taylor Swift-Ellen DeGeneres Interview, 2013" />
<br/>

# Clean your transcripts

## clean_dyads()

‘clean_dyads’ uses numerous regex to clean and format the data your just
read into R in the previous step. Although there are many cleaning
steps, here are the big ones: 1) to lowercase 2) omit stopwords 3)
replace contractions (e.g., ‘you’re’ to ‘you are’) 4) tick marks to
apostrophes 5) hypens to spaces 6) omits numerals 7) omits/squishes
extraneous white space 8) lemmatization <br/>

ConversationAlign calls the textstem package as a dependency to
lemmatize your language transcript. This converts morphologiocal
derivatives to their root forms. The default is lemmatize=T. Sometimes
you want to retain language output in its native form. If this is the
case, change the argument in clean_dyads to lemmatize=F. <br/>

‘clean_dyads’ outputs word count metrics pre/post cleaning by dyad and
interlocutor. This can be useful if you are interested in whether one
person just doesn’t produce many words or produces a great deal of empty
utterances.<br/>

``` r
MyCleanLangSamples <- clean_dyads(MyRawLangSamples) #default is lemmatize=TRUE
#If you do NOT want your language sample lemmatized, change the lemmatize argument to F or FALSE
MyCleanLangSamples <- clean_dyads(MyRawLangSamples, lemmatize=FALSE)
```

<br/>

<img src="man/figures/example3_ts_edg.jpeg" height="200"
alt="Example of cleaned transcripts from Taylor Swift-Ellen DeGeneres Interview, 2013" />
<br/>

# Align your transcripts

## align_dyads()

This is where a lot of the magic happens. align_dyads will take the
cleaned dataframe you created in the last step and yoke values to every
word by indexing a lookup database. The “align” step yokes data to each
word in the cleaned transcript text then structures a dataframe by
speaker (“Participant_ID”), exchange (“exchangecount”), and turn
(“turncount”) across each dyad (“event_id”). <br/>

You will be prompted to select one of more variables (and up to three)
to yoke data to that will be used in later steps to compute alignment
indices. You will be shown a menu wherein you can select up to three
variables to be yoked to your text. Following the menu steps, enter the
number of each variable you would like with a space separating values
(e.g., “10 14 19”). <br/>

Here are your choices: <br/>

anger, anxiety, boredom, closeness, confusion, dominance, doubt,
empathy, encouragement, excitement, guilt, happiness, hope, hostility,
politeness, sadness, stress, surprise, trust, valence, age of
acquisition, word length (by letters), morphemes per turn, prevalence
(how many people know this word), number of word senses (polysemy), word
frequency (lg10), arousal, concreteness, semantic diversity, and
semantic neighbors. <br/>

<https://reilly-lab.github.io/ConversationAlign_VariableLookupKey.pdf>
After you select the variables you would like yoked to your text, you
will be asked whether there is any metadata you would like yoked to your
data You can supply the filepath of a csv file containing metadata that
will be merged to the aligned data at this juncture, or click “Enter” to
skip this step. Run align_dyads on the cleaned dyads object you created
using the clean_dyads function.<br/>

``` r
MyAlignedDyads <- align_dyads(MyCleanLangSamples)
```

<img src="man/figures/example4_ts_edg.jpeg" height="200"
alt="Example of aligned transcripts with metics for anger, anxiety, and boredom (via AffectVec) from Taylor Swift-Ellen DeGeneres Interview, 2013" />
<br/>

# Summarize transcripts

## summarize_dyads()

Lastly, using the object produced by the prior align step, you can
obtain metrics of linguistic alignment between interlocutors on
variables yoked to your text during “align”. From this step, you will
gain metrics for (1) means for each variable per interlocutor per turn
(beginning with prefix “mean\_”), (2) the area under the curve
(beginning with prefix “auc\_”) per variable per dyad per exchange, and
(3) Spearman’s correlation coefficient (beginning with prefix
(“S_rho\_”) per variable per dyad per exchange. The means are of each
variable usage are calculated per interlocutor per turn for each
dimension within each dyad, and is calculated using the “mean” function
from the “base” package in R (R Core Team, 2023). AUC denotes the area
beneath the curve of the absolute difference between speakers’
expression on each variable over exchanges during their interaction
across all exchanges within each dyad, and is calculated using the “AUC”
function from the “DescTools” package in R (Signorell et al.,
<https://andrisignorell.github.io/DescTools/>). Spearman’s correlation
coefficient provides a metric of the overall association between two
interlocutors’ scores on each variable in their language over their
exchanges, and is calculated using the “cor.test(method =”spearman”)”
function from the “stats” package in R (R Core Team, 2023). This means
that within dyads, each interlocutor will have independent means for
each variable, but both interlocutors should have the same AUC and
Spearman’s correlation coefficient as one another. <br/>

<br/>

``` r
MySummarizedDyads <- summarize_dyads(MyAlignedDyads)
```

<br/>

# Caveat emptor

## Things you must be careful about

Any analysis of language comes with assumptions and potential bias. For
example, there are some instances where a researcher might care about
morphemes and grammatical elements such as ‘the’, ‘a’, ‘and’, etc.. The
default for ConversationAlign is to omit these as stopwords and to
average across all open class words (e.g., nouns, verbs) in each turn by
interlocutor. There are some specific cases where this can all go wrong.
Here’s what you need to consider: <br/>

1.  Stopwords: The package omits stopwords. [See the stopword list
    here](https://osf.io/atf5q/) if you would like to inspect this list.
    We included greetings, idioms, filler words, numerals, and pronouns
    in the omissions list. <br/>

2.  Lemmatization: The package will lemmatize your language transcripts
    by default. Lemmatization transforms inflected forms (e.g.,
    standing, stands) into their root or dictionary entry (e.g., stand).
    This helps for yoking offline values (e.g., happiness, concreteness)
    to each word and also entails what NLP folks refer to as ‘term
    aggregation’. However, sometimes you might NOT want to lemmatize.
    You can easily change this option by using the argument,
    “lemmatize=FALSE,” to the clean_dyads function below. <br/>

3.  Sample Size Issue 1– exchange count: The program derives
    correlations and AUC for each dyad as metrics of alignment. If there
    are 40 exchanges (80 turns) between conversation partners, the R
    value will be computed over 40 data points. For conversations less
    than about 30 turns, you should not trust the R values that
    ConversationAlign outputs. <br/>

4.  <span style="color:red;">Sample Size Issue 2 </span>: matching to
    lookup database: ConversationAlign works by yoking values from a
    lookup database to each word in your language transcript. Some
    variables have lots of values characterizing many English words.
    Other variables (e.g., age of acquisition) only cover about 30k
    words. When a word in your transcript does not have a ‘match’ in the
    lookup datase, ConversationAlign will return an NA which will not go
    into the average of the words for that interlocutor and turn. This
    can be dangerous when there are many missing values. Beware! <br/>

5.  <span style="color:red;">Compositionality </span>: ConversationAlign
    is a caveman in its complexity. It matches a value to each word as
    if that word is an island. Phenomena like polysemy (e.g., bank) and
    the modulation of one word by an intensifier (e.g., very terrible)
    are not handled. This is a problem for many of the affective
    measures but not for lexical variables like word length. <br/>

6.  <span style="color:red;"> Resampling for AUC </span>:
    summarize_dyads will output AUC values quantifying the distance
    between interlocutors on any dimension you specify. AUC will vary
    depending on the length of the dyad. Therefore, it is often
    necessary to resample (downsample) your dyads so that they are all
    of an equivalent length. This will get very wonky and
    uninterpretable for very short exchanges.

# Get in touch!

Contact <jamie_reilly@temple.edu> for feedback and assistance.
