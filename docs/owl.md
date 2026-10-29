Strukt Owl
===

Self-contained, explainable AI/NLP utilities for PHP 8.2+, built on the Strukt
Math primitives.

<center><img src="owl.jpg" width="200" height="200"></center>

No API keys, network services, Python, model server, or database are required.

## Install

```sh
composer require strukt/owl
```

The package installs `vendor/bin/owl`.

## CLI

```text
sentiment       Analyze sentiment
emotion         Detect emotions
keywords        Extract keywords
summarize       Extractive TF-IDF-style summary
language        Detect language
similarity      Compare two texts
match           Find the closest candidate
classify        Classify text with Naive Bayes
train           Train a Naive Bayes model
spam            Detect spam with a built-in heuristic
readability     Analyze readability
anomaly         Detect numeric anomalies
```

Every applicable command supports `--json`.

```bash
./vendor/bin/owl sentiment "This product is absolutely fantastic!"
./vendor/bin/owl emotion "I am so happy that we finally won!"

cat article.txt | ./vendor/bin/owl keywords --limit=10
cat article.txt | ./vendor/bin/owl summarize --sentences=5

./vendor/bin/owl language "Habari ya asubuhi, uko aje?"
./vendor/bin/owl similarity "Kenya Airways" "Kenya Airways Limited"
./vendor/bin/owl match "Kenya Airways Ltd" "Kenya Airways" "Kenya Power" "Safaricom"

cat article.txt | ./vendor/bin/owl readability
./vendor/bin/owl anomaly "1,2,3,4,5,6,7,8,9,1000" --method=iqr
```

Text can be piped or read from a file:

```bash
cat article.txt | vendor/bin/owl keywords
vendor/bin/owl sentiment --file message.txt
```

## Classifier

```bash
vendor/bin/owl train sports/*.txt --class=sports --model=model.json
vendor/bin/owl train business/*.txt --class=business --model=model.json
vendor/bin/owl classify article.txt --model=model.json
```

Training accumulates into the existing model file, so each call can add one
class.

## PHP API

```php
use Strukt\Owl\Sentiment\SentimentAnalyzer;

$result = (new SentimentAnalyzer())->analyze('I absolutely love PHP!');
```

Main classes:

```text
Strukt\Owl\Emotion\EmotionAnalyzer
Strukt\Owl\Similarity\Similarity
Strukt\Owl\Nlp\Text
Strukt\Owl\Nlp\KeywordExtractor
Strukt\Owl\Nlp\Summarizer
Strukt\Owl\Nlp\Readability
Strukt\Owl\Classifier\NaiveBayesClassifier
Strukt\Owl\Language\LanguageDetector
Strukt\Owl\Spam\SpamDetector
Strukt\Owl\Anomaly\AnomalyDetector
```

## Design

Owl favors deterministic, inspectable algorithms over opaque remote services.
Every numeric step is expressed with `Strukt\Math` primitives, so scores are
built from typed value objects rather than loose floating-point arithmetic.

It is suitable for CLI tools, PHP applications, offline processing, prototypes,
data cleaning, and lightweight NLP.

## Credits

- TextRank: https://github.com/DavidBelicza/PHP-Science-TextRank
- PHP Sentiment Analyzer: https://github.com/davmixcool/php-sentiment-analyzer
