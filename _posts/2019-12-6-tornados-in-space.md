---
layout: post
title: Tornados in Space
date: 2019-12-6
---

Say we want a quick way to analyze snippets of text from diverse environments. Let's hide [SpaCy](https://spacy.io/) behind a little [Tornado](https://github.com/tornadoweb/tornado/blob/stable/docs/index.rst) API and call it, like, `spacetornado`. We'll need a Unix terminal and some version of Python 3.

---

To keep our workspace neat and tidy, we'll use `venv`. First, we create a new directory and initialize a Python 3 virtual environment:

```
> $ mkdir spacetornado
> $ cd spacetornado
> $ python3 -m venv .
> $ source bin/activate
```

Next we'll install Tornado and SpaCy, then download the small version of the (English) model:

```
> (spacetornado) $ pip install tornado
> (spacetornado) $ pip install spacy
> (spacetornado) $ python -m spacy download en_core_web_sm
```

Great. Now to assemble the thing. We'll have 3 little modules:

- `operations.py`: SpaCy operations
- `handlers.py`: Tornado request handlers
- `host.py`: Tornado server

---

##### `operations.py`

Here we'll wrap SpaCy with a bit of code to mold responses. We'll have 4 methods:

- tokens (everything)
- noun_chunks (all your nouns are belong to me)
- entities (the nouns, but special)
- similarity (correlation on a [0, 1] scale)

```
import spacy

# Load the SpaCy model

nlp = spacy.load('en_core_web_sm')

# Define the response shape for each API operation.
# We'll follow SpaCy's structure and naming conventions pretty closely, but decoupling
# the objects returned from the SpaCy domain model might make things easier later on.

def __mapToken(token):
    return {
        'text'		: str(token.text),
        'lemma'         : str(token.lemma_),
        'pos'           : str(token.pos_),
        'tag'           : str(token.tag_),
        'prettytag'     : str(spacy.explain(token.tag_)),
        'dep'           : str(token.dep_),
        'shape'         : str(token.shape_),
        'alpha'         : str(token.is_alpha),
        'stop'          : str(token.is_stop)
    }

def __mapNounChunk(chunk):
    return {
        'text'		: str(chunk.text),
        'roottext'	: str(chunk.root.text),
        'rootdep'	: str(chunk.root.dep_),
        'rootheadtext'	: str(chunk.root.head.text)
    }

def __mapEntity(entity):
    return {
        'text'		: str(entity.text),
        'start'		: str(entity.start_char),
        'end'		: str(entity.end_char),
        'label'		: str(entity.label_)
    }

def __mapSimilarity(left, right):
    return {
        'similarity'	: left.similarity(right)
    }

# Define the operations. In the first 3 operations we're returning
# arrays; in `similarity` we're returning a single object.

def tokens(text):
    return [__mapToken(token) for token in nlp(text)]

def noun_chunks(text):
    return [__mapNounChunk(chunk) for chunk in nlp(text).noun_chunks]

def entities(text):
    return [__mapEntity(entity) for entity in nlp(text).ents]

def similarity(left, right):
    return __mapSimilarity(nlp(left), nlp(right))
```

##### `handlers.py`

Here we'll hook each operation into a Tornado request handler and convert it to pretty-indented JSON.

```
import json
import tornado.web
import operations

# JSON for everyone!

content_type = "Content-Type", "application/json"

# Handlers for all the operations

class Tokens(tornado.web.RequestHandler):
    def post(self):
        self.set_header(content_type[0], content_type[1])
        self.write(json.dumps(operations.tokens(self.get_body_argument("text")), indent=4))

class NounChunks(tornado.web.RequestHandler):
    def post(self):
        self.set_header(content_type[0], content_type[1])
        self.write(json.dumps(operations.noun_chunks(self.get_body_argument("text")), indent=4))

class Entities(tornado.web.RequestHandler):
    def post(self):
        self.set_header(content_type[0], content_type[1])
        self.write(json.dumps(operations.entities(self.get_body_argument("text")), indent=4))

class Similarity(tornado.web.RequestHandler):
    def get(self):
        self.set_header(content_type[0], content_type[1])
        self.write(json.dumps(operations.similarity(self.get_query_argument("left"), self.get_query_argument("right")), indent=4))
    def post(self):
        self.set_header(content_type[0], content_type[1])
        self.write(json.dumps(operations.similarity(self.get_body_argument("left"), self.get_body_argument("right")), indent=4))
```

##### `host.py`

Tie it all together:

```
import logging
import tornado.ioloop
import tornado.web
import spacy
import handlers

# Production-ready configuration solution :)

host = "localhost"
port = "8888"

# Ready the loggers

logger = logging.getLogger('spacetornado')

def loggers():
    handler = logging.StreamHandler()
    handler.setFormatter(logging.Formatter('%(asctime)s %(levelname)s [%(name)s] %(message)s'))
    logger.addHandler(handler)
    tornadoLogger = logging.getLogger("tornado.access")
    tornadoLogger.propagate = False
    tornadoLogger.setLevel(logging.DEBUG)
    tornadoLogger.handlers = [handler]

# Bind endpoints to handlers

def app():
    return tornado.web.Application([
        (r"/tokens", handlers.Tokens),
        (r"/noun_chunks", handlers.NounChunks),
        (r"/entities", handlers.Entities),
        (r"/similarity", handlers.Similarity)])

# Spin it up!

if __name__ == "__main__":
    loggers()
    app().listen(port)
    logger.info("Listening at %s:%s." % (host, port))
    tornado.ioloop.IOLoop.current().start()
```

---

That's it: time to tokenize.

```
> (spacetornado) python src/host.py
> 2019-12-06 21:17:28,540 INFO [spacetornado] Listening at localhost:8888.
```

Pop open a new terminal:

```
> curl -X POST --data "text=Who said there were no tornados in space?" http://localhost:8888/tokens
> [
    {
        "text": "Who",
        "lemma": "who",
        "pos": "PRON",
        "tag": "WP",
        "prettytag": "wh-pronoun, personal",
        "dep": "nsubj",
        "shape": "Xxx",
        "alpha": "True",
        "stop": "True"
    },
    {
        "text": "said",
        "lemma": "say",
        "pos": "VERB",
        "tag": "VBD",
        "prettytag": "verb, past tense",
        "dep": "ROOT",
        "shape": "xxxx",
        "alpha": "True",
        "stop": "False"
    },
    {
        "text": "there",
        "lemma": "there",
        "pos": "PRON",
        "tag": "EX",
        "prettytag": "existential there",
        "dep": "expl",
        "shape": "xxxx",
        "alpha": "True",
        "stop": "True"
    },
    {
        "text": "were",
        "lemma": "be",
        "pos": "AUX",
        "tag": "VBD",
        "prettytag": "verb, past tense",
        "dep": "ccomp",
        "shape": "xxxx",
        "alpha": "True",
        "stop": "True"
    },
    {
        "text": "no",
        "lemma": "no",
        "pos": "DET",
        "tag": "DT",
        "prettytag": "determiner",
        "dep": "det",
        "shape": "xx",
        "alpha": "True",
        "stop": "True"
    },
    {
        "text": "tornados",
        "lemma": "tornado",
        "pos": "NOUN",
        "tag": "NNS",
        "prettytag": "noun, plural",
        "dep": "attr",
        "shape": "xxxx",
        "alpha": "True",
        "stop": "False"
    },
    {
        "text": "in",
        "lemma": "in",
        "pos": "ADP",
        "tag": "IN",
        "prettytag": "conjunction, subordinating or preposition",
        "dep": "prep",
        "shape": "xx",
        "alpha": "True",
        "stop": "True"
    },
    {
        "text": "space",
        "lemma": "space",
        "pos": "NOUN",
        "tag": "NN",
        "prettytag": "noun, singular or mass",
        "dep": "pobj",
        "shape": "xxxx",
        "alpha": "True",
        "stop": "False"
    },
    {
        "text": "?",
        "lemma": "?",
        "pos": "PUNCT",
        "tag": ".",
        "prettytag": "punctuation mark, sentence closer",
        "dep": "punct",
        "shape": "?",
        "alpha": "False",
        "stop": "False"
    }
]
```

Not bad!
