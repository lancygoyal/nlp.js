---
title: "@nlpjs/lang-en"
---

# @nlpjs/lang-en

## Installation

You can install @nlpjs/lang-en:

```bash
    npm install @nlpjs/lang-en
```

## Normalization

Normalization of a text converts it to lowercase and remove decorations of characters.

```javascript
const { NormalizerEn } = require("@nlpjs/lang-en");

const normalizer = new NormalizerEn();
const input = "This shóuld be normalized";
const result = normalizer.normalize(input);
console.log(result);
// output: this should be normalized
```

## Tokenization

Tokenization splits a sentence into words.

```javascript
const { TokenizerEn } = require("@nlpjs/lang-en");

const tokenizer = new TokenizerEn();
const input = "This isn't tokenized yet";
const result = tokenizer.tokenize(input);
console.log(result);
// output: [ 'This', 'is', 'not', 'tokenized', 'yet' ]
```

Tokenizer can also normalize the sentence before tokenizing, to do that provide a _true_ as second argument to the method _tokenize_

```javascript
const { TokenizerEn } = require('@nlpjs/lang-en');

const tokenizer = new TokenizerEn();
const input = 'This isn\'t tokenized yet';
const result = tokenizer.tokenize(input true);
console.log(result);
// output: [ 'this', 'is', 'not', 'tokenized', 'yet' ]
```

## Identify if a word is an english stopword

Using the class _StopwordsEn_ you can identify if a word is an stopword:

```javascript
const { StopwordsEn } = require("@nlpjs/lang-en");

const stopwords = new StopwordsEn();
console.log(stopwords.isStopword("is"));
// output: true
console.log(stopwords.isStopword("developer"));
// output: false
```

## Remove stopwords from an array of words

Using the class _StopwordsEn_ you can remove stopwords form an array of words:

```javascript
const { StopwordsEn } = require("@nlpjs/lang-en");

const stopwords = new StopwordsEn();
console.log(stopwords.removeStopwords(["who", "is", "your", "develop"]));
// output: ['develop']
```

## Change the stopwords dictionary

Using the class _StopwordsEn_ you can restart it dictionary and build it from another set of words:

```javascript
const { StopwordsEn } = require("@nlpjs/lang-en");

const stopwords = new StopwordsEn();
stopwords.dictionary = {};
stopwords.build(["is", "your"]);
console.log(stopwords.removeStopwords(["who", "is", "your", "develop"]));
// output: ['who', 'develop']
```

## Stemming word by word

An stemmer is an algorithm to calculate the _stem_ (root) of a word, removing affixes.

You can stem one word using method _stemWord_:

```javascript
const { StemmerEn } = require("@nlpjs/lang-en");

const stemmer = new StemmerEn();
const input = "developer";
console.log(stemmer.stemWord(input));
// output: develop
```

## Stemming an array of words

You can stem an array of words using method _stem_:

```javascript
const { StemmerEn } = require("@nlpjs/lang-en");

const stemmer = new StemmerEn();
const input = ["Who", "is", "your", "developer"];
console.log(stemmer.stem(input));
// outuput: [ 'Who', 'is', 'your', 'develop' ]
```

## Normalizing, Tokenizing and Stemming a sentence

As you can see, stemmer does not do internal normalization, so words with uppercases will remain uppercased.
Also, stemmer works with lowercased affixes, so _developer_ will be stemmed as _develop_ but _DEVELOPER_ will not be changed.

You can tokenize and stem a sentence, including normalization, with the method _tokenizeAndStem_:

```javascript
const { StemmerEn } = require("@nlpjs/lang-en");

const stemmer = new StemmerEn();
const input = "Who is your DEVELOPER";
console.log(stemmer.tokenizeAndStem(input));
// output: [ 'who', 'is', 'your', 'develop' ]
```

## Remove stopwords when stemming a sentence

When calling _tokenizeAndStem_ method from the class _StemmerEn_, the second parameter is a boolean to set if the stemmer must keep the stopwords (true) or remove them (false). Before using it, the stopwords instance must be set into the stemmer:

```javascript
const { StemmerEn, StopwordsEn } = require("@nlpjs/lang-en");

const stemmer = new StemmerEn();
stemmer.stopwords = new StopwordsEn();
const input = "who is your developer";
console.log(stemmer.tokenizeAndStem(input, false));
// output: ['develop']
```

## Sentiment Analysis

To use sentiment analysis you'll need to create a new _Container_ and use the plugin _LangEn_, because internally the _SentimentAnalyzer_ class try to retrieve the normalizer, tokenizer, stemmmer and sentiment dictionaries from the container.

```javascript
const { Container } = require("@nlpjs/core");
const { SentimentAnalyzer } = require("@nlpjs/sentiment");
const { LangEn } = require("@nlpjs/lang-en");

(async () => {
  const container = new Container();
  container.use(LangEn);
  const sentiment = new SentimentAnalyzer({ container });
  const result = await sentiment.process({ locale: "en", text: "I love cats" });
  console.log(result.sentiment);
})();
// output:
// {
//   score: 0.5,
//   numWords: 3,
//   numHits: 1,
//   average: 0.16666666666666666,
//   type: 'senticon',
//   locale: 'en',
//   vote: 'positive'
// }
```

The output of the sentiment analysis includes:

- _score_: final score of the sentence.
- _numWords_: total words of the sentence.
- _numHits_: total words of the sentence identified as having a sentiment score.
- _average_: score divided by numWords
- _type_: type of dictionary used, values can be afinn, senticon or pattern.
- _locale_: locale of the sentence
- _vote_: positive if score greater than 0, negative if score lower than 0, neutral if score equals 0.

## Example of usage on a classifier

```javascript
const { containerBootstrap } = require("@nlpjs/core");
const { Nlp } = require("@nlpjs/nlp");
const { LangEn } = require("@nlpjs/lang-en");

(async () => {
  const container = await containerBootstrap();
  container.use(Nlp);
  container.use(LangEn);
  const nlp = container.get("nlp");
  nlp.settings.autoSave = false;
  nlp.addLanguage("en");
  // Adds the utterances and intents for the NLP
  nlp.addDocument("en", "goodbye for now", "greetings.bye");
  nlp.addDocument("en", "bye bye take care", "greetings.bye");
  nlp.addDocument("en", "okay see you later", "greetings.bye");
  nlp.addDocument("en", "bye for now", "greetings.bye");
  nlp.addDocument("en", "i must go", "greetings.bye");
  nlp.addDocument("en", "hello", "greetings.hello");
  nlp.addDocument("en", "hi", "greetings.hello");
  nlp.addDocument("en", "howdy", "greetings.hello");

  // Train also the NLG
  nlp.addAnswer("en", "greetings.bye", "Till next time");
  nlp.addAnswer("en", "greetings.bye", "see you soon!");
  nlp.addAnswer("en", "greetings.hello", "Hey there!");
  nlp.addAnswer("en", "greetings.hello", "Greetings!");
  await nlp.train();
  const response = await nlp.process("en", "I should go now");
  console.log(response);
})();
```