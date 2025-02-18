Wordist
=============

Wordist is an open source dictionary with incremental search **hosted entirely on a static web server**.

Try out wordist here http://akumpf.github.io/wordist/

There's no backend logic doing anything fancy; it's just a big folder full of little javascipt snippets with corresponding definitions, parts of speech, etc. so you can experiment with language without 3rd-party APIs.

The entire dataset is a bit hefty to access directly (> 20MB), but breaking it up into 3000+ little pages of words and definitions keeps it light-weight and speedy for the web. Big thanks to the GNU Collaborative International Dictionary of English for the data that makes Wordist possible.

## Usage

### including wordist.js

First things first! You'll need to add a script tag to your HTML to get the party started.

```
<script src="http://akumpf.github.io/wordist/dist/wordist.js"></script>
```

You'll probably want to download your own copy and put it on your server to ensure things are fast, but for testing you can just pull in the javascript remotely as shown above.

### wordist.init(options)

Before you make calls to wordist, you should tell it where things are. Do this by calling `init`.

```
options = {
  path: "PATH_TO_DICTIONARY_DIST" // default is "./dist"
}
```

### wordist.getDef(word, callback)

Get a definition for a word.

```
// get the definition of "apple"
wordist.getDef("apple", function exampleDefHandler(err, root, data){
  if(err)   return console.warn(err); 
  if(!root) return console.warn("Unexpected result?", root, data);
  if(!data) return console.log("No def for: "+root);
  // --
  var entries        = data.e||[];
  var altWords       = data.a||[];
  var seeAlso        = data.seeAlso||{};
  var keyWords       = data.keyWords||[];
  var pronunciations = data.p||[];
  var rhymeKeys      = data.r||[];
  console.log(root, altWords, entries, seeAlso, keyWords, pronunciations, rhymeKeys, data);
});

// note that pronunciations and rhymeKeys are an array of arrays of strings and map to:
// [[root pronunciations/rhymes],[alt1 pronunciations/rhymes],[alt2 pronunciations/rhymes],...]
```

### wordist.getDefClosest(word, callback)

If you want to try harder to find a definition (in case the word is not a direct hit in the dictionary) use `getDefClosest()`. 

```
//This will fallback to the definition for "apple".
wordist.getDefClosest("applely", function exampleDefHandler(err, root, data){
  if(err)   return console.warn(err); 
  if(!root) return console.warn("Unexpected result?", root, data);
  if(!data) return console.log("No def for: "+root);
  // --
  var entries        = data.e||[];
  var altWords       = data.a||[];
  var seeAlso        = data.seeAlso||{};
  var keyWords       = data.keyWords||[];
  var pronunciations = data.p||[];
  var rhymeKeys      = data.r||[];
  console.log(root, altWords, entries, seeAlso, keyWords, pronunciations, rhymeKeys, data);
});
```

### wordist.getDefRandom(callback)

Sometimes you just want a random word, and `getDefRandom` will do that for you.

```
// Get a random word.
wordist.getDefRandom(function exampleDefHandler(err, root, data){
  if(err)   return console.warn(err); 
  if(!root) return console.warn("Unexpected result?", root, data);
  if(!data) return console.log("No def for: "+root);
  // --
  var entries        = data.e||[];
  var altWords       = data.a||[];
  var seeAlso        = data.seeAlso||{};
  var keyWords       = data.keyWords||[];
  var pronunciations = data.p||[];
  var rhymeKeys      = data.r||[];
  console.log(root, altWords, entries, seeAlso, keyWords, pronunciations, rhymeKeys, data);
});
```

### wordist.getPoSAll(partOfSpeech, callback)

Wordist contains lists of all defined words grouped by part of speech (like noun, verb, adjective, etc.).

None of the lists are enormous in size, but nouns and adjectives are a few hundred kB, so be aware of that if you are looking to keep your site fast.

Note that `partOfSpeech` is just the first 4 lettters of the part of speech you are interested in.

```
// get all the adjectives!
wordist.getPoSAll("adje", function(err, pos, allWords){
  if(err) return console.warn(err);
  console.log(pos,allWords);
});
```

### wordist.getPoSRandom(partOfSpeech, callback)

You can also just get a random word within a part of speech. 

This is handy for generating creative output (e.x. Mad Libs).

```
wordist.getPoSRandom("adje", function(err, randAdjective){
  if(err) return console.warn(err);
  console.log(randAdjective);
});
```

### wordist.getFieldAll(field, callback)

Wordist contains lists of all defined words grouped by field where defined (like acoustics, forestry, navigation, etc.).

Note that `field` is often shorthand for a longer, more readable name. See `getFields()` for the full list.

```
// get all the zoology words!
wordist.getFieldAll("zool", function(err, field, wordsInField){
  if(err) return console.warn(err);
  console.log(field, wordsInField);
});
```

### wordist.getFields(callback)

Fetches and returns the full list of fields specified in the dictionary (like acoustics, forestry, navigation, etc.).

```
// get all the fields defined in the dictionary
wordist.getFields(function(err, allFields){
  if(err) return console.warn(err);
  console.log(allFields);
});
```

### wordist.getFindable(percent, callback)

Wordist computes the findability of each word in the dictionary. 

Findability is a measure of how many times a word is included other definitions (without double counting). It can be used to get a general sense of how common a word is in the language.

The findable words are pre-sorted into 100 sections. 1 will return the most common words. 100 will return the least common words that are still findable (occurance >= 2).  

```
// Get the most findable words in the dictioary!
wordist.getFindable(1, function(err, pgNum, findableWords){
   if(err) return console.warn(err);
   console.log(pgNum,findableWords);
});
```

### wordist.distillText(txt)

Sometimes you want to pull out words from a larger block of text that are likely to be important or meaningful. Wordist has a simple function that takes in a text string and returns a list of words that are likely to be useful for further analysis.

Essentially, this function filters out pronouns, prepositions, contractions, punctuation, and common dictionary words that are likely to be extraneous. 

Note that the array may have duplicate words, so be sure to unique-ify it if routing into further processing so you don't do more data crunching than necessary.

```
wordist.distillText("It's not everyday that you see a flying goat eating breakfast on a surfboard, but today was no ordinary day")
//  ["everyday", "flying", "goat", "eating", "breakfast", "surfboard", "today", "ordinary", "day"]
```

### wordist.getRhymes(rhymeKey, callback)

Each definition also returns a set of rhymeKeys (`r`, which is a text representation of the phonemes from the last strong vowel sound in the word) for each pronunciation of the word. You can then use a rhymeKey to get all rhymes for words ending with those same phonemes. 

The returned array includes both the rhyming words (`w`) and their relative findability in the dictionary (`f`) since findability can be a useful metric when bubbling up relevant suggestions to a user.

```
// get words that rhyme with the common pronunciation of the word "bubble" (which is specified with a rhyme key of "ah1-b-ah0-l", as found when fetching the definition).
wordist.getRhymes("ah1-b-ah0-l", function exampleRhymeHandler(err, rhymeKey, rhymingWords){
  if(err)   return console.warn(err); 
  if(!rhymeKey) return console.warn("Unexpected result?", rhymeKey, rhymingWords);
  if(!rhymingWords) return console.log("No rhymes for: "+rhymeKey);
  // --
  console.log(rhymeKey, rhymingWords);
});
// ah1-b-ah0-l 
(8) [
  0: {w: "double",   f: 306}
  1: {w: "trouble",  f: 104}
  2: {w: "bubble",   f: 47}
  3: {w: "stubble",  f: 12}
  4: {w: "rubble",   f: 6}
  5: {w: "redouble", f: 2}
  6: {w: "hubbell",  f: 0}
  7: {w: "hubble",   f: 0}
]
```



### example code

You can also see wordist in action by checking out `index.html` or play with the demo here http://akumpf.github.io/wordist/

## Building wordist from source

To use wordist, you shouldn't need to build anything. 

But in case you want to monkey with the internals and add functionality to wordist...

* Download the source
* Run `npm install` to get all the required node modules.
* run `node build.js` for each stage (currently 3, set this in build.js). This takes about an hour.
* load `index.html` and make sure it worked!

## License

Wordist is licensed under Creative Commons 0 (Public Domain) except where otherwise noted (the libraries, for example, have their own terms). 

Wordist is backed by dictionary data from the GNU Collaborative International Dictionary of English (or GCIDE). See http://gcide.gnu.org.ua/.

GCIDE is free software; you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation; either version 3, or (at your option) any later version.

On top of GCIDE, Wordist also makes use of a cleanly parsed version of the dictionary by javierjulio https://github.com/javierjulio/dictionary.

Additionally, rhymes are generated by using phonemes specified in the CMU Pronouncing Dictionary http://svn.code.sf.net/p/cmusphinx/code/trunk/cmudict/.

Wordist is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.


