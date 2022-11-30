# @nlpjs/lang-es

## Installation

You can install @nlpjs/lang-es:

```bash
    npm install @nlpjs/lang-es
```

## Normalization

Normalization of a text converts it to lowercase and remove decorations of characters.

```javascript
const { NormalizerEs } = require("@nlpjs/lang-es");

const normalizer = new NormalizerEs();
const input = "Esto debería ser normalizado";
const result = normalizer.normalize(input);
console.log(result);
// output: esto deberia ser normalizado
```

## Tokenization

Tokenization splits a sentence into words.

```javascript
const { TokenizerEs } = require("@nlpjs/lang-es");

const tokenizer = new TokenizerEs();
const input = "Esto debería ser tokenizado";
const result = tokenizer.tokenize(input);
console.log(result);
// output: [ 'Esto', 'debería', 'ser', 'tokenizado' ]
```

Tokenizer can also normalize the sentence before tokenizing, to do that provide a _true_ as second argument to the method _tokenize_

```javascript
const { TokenizerEs } = require("@nlpjs/lang-es");

const tokenizer = new TokenizerEs();
const input = "Esto debería ser tokenizado";
const result = tokenizer.tokenize(input, true);
console.log(result);
// output: [ 'esto', 'deberia', 'ser', 'tokenizado' ]
```

## Identify if a word is a spanish stopword

Using the class _StopwordsEs_ you can identify if a word is an stopword:

```javascript
const { StopwordsEs } = require("@nlpjs/lang-es");

const stopwords = new StopwordsEs();
console.log(stopwords.isStopword("un"));
// output: true
console.log(stopwords.isStopword("desarrollador"));
// output: false
```

## Remove stopwords from an array of words

Using the class _StopwordsEs_ you can remove stopwords form an array of words:

```javascript
const { StopwordsEs } = require("@nlpjs/lang-es");

const stopwords = new StopwordsEs();
console.log(
  stopwords.removeStopwords(["he", "visto", "a", "un", "programador"])
);
// output: ['he', 'visto', 'programador']
```

## Change the stopwords dictionary

Using the class _StopwordsEs_ you can restart it dictionary and build it from another set of words:

```javascript
const { StopwordsEs } = require("@nlpjs/lang-es");

const stopwords = new StopwordsEs();
stopwords.dictionary = {};
stopwords.build(["he", "visto"]);
console.log(
  stopwords.removeStopwords(["he", "visto", "a", "un", "programador"])
);
// output: ['a', 'un', 'programador']
```

## Stemming word by word

An stemmer is an algorithm to calculate the _stem_ (root) of a word, removing affixes.

You can stem one word using method _stemWord_:

```javascript
const { StemmerEs } = require("@nlpjs/lang-es");

const stemmer = new StemmerEs();
const input = "programador";
console.log(stemmer.stemWord(input));
// output: program
```

## Stemming an array of words

You can stem an array of words using method _stem_:

```javascript
const { StemmerEs } = require("@nlpjs/lang-es");

const stemmer = new StemmerEs();
const input = ["he", "visto", "a", "un", "programador"];
console.log(stemmer.stem(input));
// outuput: [ 'hab', 'vist', 'a', 'un', 'program' ]
```

## Normalizing, Tokenizing and Stemming a sentence

As you can see, stemmer does not do internal normalization, so words with uppercases will remain uppercased.
Also, stemmer works with lowercased affixes, so _programador_ will be stemmed as _program_ but _PROGRAMADOR_ will not be changed.

You can tokenize and stem a sentence, including normalization, with the method _tokenizeAndStem_:

```javascript
const { StemmerEs } = require("@nlpjs/lang-es");

const stemmer = new StemmerEs();
const input = "He visto a un PROGRAMADOR";
console.log(stemmer.tokenizeAndStem(input));
// output: [ 'hab', 'vist', 'a', 'un', 'program' ]
```

## Remove stopwords when stemming a sentence

When calling _tokenizeAndStem_ method from the class _StemmerES_, the second parameter is a boolean to set if the stemmer must keep the stopwords (true) or remove them (false). Before using it, the stopwords instance must be set into the stemmer:

```javascript
const { StemmerEs, StopwordsEs } = require("@nlpjs/lang-es");

const stemmer = new StemmerEs();
stemmer.stopwords = new StopwordsEs();
const input = "he visto a un programador";
console.log(stemmer.tokenizeAndStem(input, false));
// output: ['hab', 'vist', 'program']
```

## Sentiment Analysis

To use sentiment analysis you'll need to create a new _Container_ and use the plugin _LangES_, because internally the _SentimentAnalyzer_ class try to retrieve the normalizer, tokenizer, stemmmer and sentiment dictionaries from the container.

```javascript
const { Container } = require("@nlpjs/core");
const { SentimentAnalyzer } = require("@nlpjs/sentiment");
const { LangEs } = require("@nlpjs/lang-es");

(async () => {
  const container = new Container();
  container.use(LangEs);
  const sentiment = new SentimentAnalyzer({ container });
  const result = await sentiment.process({
    locale: "es",
    text: "me gustan los gatos",
  });
  console.log(result.sentiment);
})();
// output:
// {
//   score: 0.266,
//   numWords: 4,
//   numHits: 1,
//   average: 0.0665,
//   type: 'senticon',
//   locale: 'es',
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
const { LangEs } = require("@nlpjs/lang-es");

(async () => {
  const container = await containerBootstrap();
  container.use(Nlp);
  container.use(LangEs);
  const nlp = container.get("nlp");
  nlp.settings.autoSave = false;
  nlp.addLanguage("es");
  // Adds the utterances and intents for the NLP
  nlp.addDocument("es", "adios por ahora", "greetings.bye");
  nlp.addDocument("es", "adios y ten cuidado", "greetings.bye");
  nlp.addDocument("es", "muy bien nos vemos luego", "greetings.bye");
  nlp.addDocument("es", "debo irme", "greetings.bye");
  nlp.addDocument("es", "hola", "greetings.hello");

  // Train also the NLG
  nlp.addAnswer("es", "greetings.bye", "hasta la proxima");
  nlp.addAnswer("es", "greetings.bye", "¡te veo pronto!");
  nlp.addAnswer("es", "greetings.hello", "¡hola que tal!");
  nlp.addAnswer("es", "greetings.hello", "¡salludos!");
  await nlp.train();
  const response = await nlp.process("es", "debo irme");
  console.log(response);
})();
```