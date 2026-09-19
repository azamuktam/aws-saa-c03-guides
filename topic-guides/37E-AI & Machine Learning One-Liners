# Section 37E: AI & Machine Learning One-Liners

## The idea

These AWS AI/ML services usually appear in SAA questions as **very specific use cases**.

You generally don't need deep knowledge of how the machine-learning models work.

The best strategy is:

> **Identify what type of input you have → identify what the question wants to do with it → choose the unique service.**

For example:

```text
Images / video
→ Rekognition

Audio / speech → text
→ Transcribe

Text → speech
→ Polly

Text → another language
→ Translate

Text analysis / sentiment
→ Comprehend

Scanned documents / tables / forms
→ Textract

Search company documents
→ Kendra

Recommendations
→ Personalize

Time-series forecasting
→ Forecast

Chatbot
→ Lex

Build your own ML model
→ SageMaker
```

---

# AI / ML Service Family

| Service                | What it does                                                                                     | Signal                         |
| ---------------------- | ------------------------------------------------------------------------------------------------ | ------------------------------ |
| **Amazon Rekognition** | Looks at images and videos and detects things such as faces, objects, people, and unsafe content | Faces, objects, video          |
| **Amazon Transcribe**  | Takes audio/speech and turns it into written text                                                | Call recording → transcript    |
| **Amazon Polly**       | Takes written text and turns it into spoken audio                                                | App reads text aloud           |
| **Amazon Translate**   | Takes text in one language and translates it into another language                               | English → French               |
| **Amazon Comprehend**  | Takes text and analyzes its meaning, such as sentiment, entities, and key phrases                | Positive/negative review       |
| **Amazon Textract**    | Takes scanned documents/images and extracts text, tables, and form fields                        | Invoice/form → structured data |
| **Amazon Kendra**      | Searches company documents and finds relevant answers using natural-language queries             | Find vacation policy           |
| **Amazon Personalize** | Uses user/item behavior to generate personalized recommendations                                 | Customers also bought          |
| **Amazon Forecast**    | Uses historical time-series data to predict future values                                        | Predict future sales/demand    |
| **Amazon Lex**         | Lets you build conversational chatbots that understand user messages and respond                 | Customer-service chatbot       |
| **Amazon SageMaker**   | Lets data scientists build, train, tune, and deploy their own ML models                          | Train your own ML model        |

---

# Amazon Rekognition

**Amazon Rekognition = image and video analysis.**

It can analyze:

* images
* videos
* faces
* objects
* people
* unsafe or inappropriate content

### Signal

> **Faces / objects / people / image / video analysis → Rekognition**

---

## Example

> "A company needs to automatically detect faces and objects in uploaded images."

→ **Amazon Rekognition**

Another example:

> "A company needs to identify unsafe content in uploaded videos."

→ **Amazon Rekognition**

---

## Memory

```text
Image / video
→ Rekognition
```

---

# Amazon Transcribe

**Amazon Transcribe = speech-to-text.**

It takes audio or speech and converts it into written text.

```text
Audio
 ↓
Transcribe
 ↓
Text
```

### Use cases

* call-center recordings
* meeting recordings
* voice recordings
* audio transcription

### Signal

> **Speech → text → Transcribe**

### Example

> "A company records customer-service calls and wants to automatically create text transcripts."

→ **Amazon Transcribe**

---

# Amazon Polly

**Amazon Polly = text-to-speech.**

It takes written text and generates spoken audio.

```text
Text
 ↓
Polly
 ↓
Speech / audio
```

### Signal

> **Text → speech → Polly**

### Example

> "A mobile application needs to read articles aloud to users."

→ **Amazon Polly**

---

# Transcribe vs Polly

These are opposites.

```text
Speech
   ↓
Transcribe
   ↓
Text
```

```text
Text
   ↓
Polly
   ↓
Speech
```

### Memory

> **Transcribe = speech → text**

> **Polly = text → speech**

---

# Amazon Translate

**Amazon Translate = text translation between languages.**

It takes text in one language and translates it into another language.

```text
English
   ↓
Translate
   ↓
French
```

### Signal

> **Translate text between languages → Amazon Translate**

### Example

> "An application receives English customer messages and needs to provide them in French."

→ **Amazon Translate**

---

# Amazon Comprehend

**Amazon Comprehend = natural-language processing and text analysis.**

It can analyze text for things such as:

* sentiment
* entities
* key phrases
* language
* other text insights

### Signal

> **Understand the meaning or sentiment of text → Comprehend**

---

## Example

> "A company wants to determine whether customer reviews are positive, negative, or neutral."

→ **Amazon Comprehend**

### Another example

> "A company wants to automatically identify names, organizations, and locations in customer feedback."

→ **Amazon Comprehend**

---

## Memory

```text
Text
 ↓
Comprehend
 ↓
Meaning / sentiment / entities / key phrases
```

---

# Amazon Textract

**Amazon Textract = extract text, tables, and form fields from scanned documents and images.**

It is especially useful when the input is a **document**, such as:

* invoices
* forms
* scanned documents
* receipts
* identity documents
* tables

### Signal

> **Scanned document → text / tables / forms → Textract**

---

## Example

> "A company needs to automatically extract names, fields, tables, and values from scanned invoices."

→ **Amazon Textract**

---

## Important distinction

Do not confuse Textract with Rekognition.

```text
Rekognition
= image / video analysis

Textract
= document / text / table / form extraction
```

### Memory

```text
Scanned invoice
       ↓
   Textract
       ↓
Text + fields + tables
```

---

# Rekognition vs Textract

This is a common conceptual distinction.

### Rekognition

Use it when the question is about **understanding what is in an image or video**.

```text
Image
→ faces
→ objects
→ people
→ unsafe content
→ Rekognition
```

### Textract

Use it when the question is about **extracting structured information from a document**.

```text
Scanned invoice
→ text
→ tables
→ form fields
→ Textract
```

### Quick memory

> **Rekognition = SEE the image**

> **Textract = READ the document**

---

# Amazon Kendra

**Amazon Kendra = intelligent enterprise search.**

It can search company documents and return relevant information using natural-language queries.

### Signal

> **Search internal company documents using natural-language queries → Kendra**

---

## Example

Suppose employees have thousands of internal documents.

An employee asks:

> "What is our vacation policy?"

Kendra can search the organization's indexed content and return relevant information.

→ **Amazon Kendra**

---

## Memory

```text
Company documents
        ↓
      Kendra
        ↓
Natural-language search
        ↓
Relevant information
```

### Simple signal

> **Enterprise document search → Kendra**

---

# Amazon Personalize

**Amazon Personalize = personalized recommendations based on user and item behavior.**

It can use information such as:

* user behavior
* item interactions
* preferences
* historical activity

to generate personalized recommendations.

### Signal

> **Recommendations based on user behavior → Personalize**

---

## Example

> "An online store wants to recommend products based on each customer's previous activity."

→ **Amazon Personalize**

Another common example:

> "Customers who purchased this item may be interested in similar items."

→ **Amazon Personalize**

---

## Memory

```text
User behavior
      ↓
Personalize
      ↓
Personalized recommendations
```

### Simple signal

> **Recommendations → Personalize**

---

# Amazon Forecast

**Amazon Forecast = time-series forecasting.**

It uses historical time-series data to predict future values.

Common examples include:

* sales
* demand
* inventory requirements
* resource usage

### Signal

> **Predict future values from historical time-series data → Forecast**

---

## Example

> "A company wants to predict next month's product demand based on historical sales data."

→ **Amazon Forecast**

---

## Memory

```text
Historical time-series data
          ↓
       Forecast
          ↓
Future values
```

### Simple signal

> **Future demand / sales prediction → Forecast**

---

# Amazon Lex

**Amazon Lex = conversational chatbot service.**

It lets you build applications that understand user messages and interact conversationally.

It is commonly used for:

* customer-service chatbots
* conversational interfaces
* voice or text interactions

### Signal

> **Build a conversational chatbot → Lex**

---

## Example

> "A company wants to build a customer-service chatbot that can understand customer requests."

→ **Amazon Lex**

---

## Memory

```text
User
 ↓
Conversational interface
 ↓
Lex
 ↓
Application response
```

### Simple signal

> **Chatbot → Lex**

---

# Amazon SageMaker

**Amazon SageMaker = build, train, tune, and deploy your own machine-learning models.**

It is the service to think about when the question is about **custom machine learning development**, rather than simply calling a prebuilt AI capability.

Data scientists can use it to:

* build models
* train models
* tune models
* deploy models
* manage ML workflows

### Signal

> **Build/train/deploy your own ML model → SageMaker**

---

## Example

> "Data scientists need to build and train a custom machine-learning model using the company's own dataset."

→ **Amazon SageMaker**

---

## Important distinction

```text
Use a ready-made AI capability
→ Specialized AI service

Build your own ML model
→ SageMaker
```

For example:

```text
Image / video analysis
→ Rekognition

Speech → text
→ Transcribe

Custom ML model
→ SageMaker
```

---

# The Important Input → Output Patterns

One of the easiest ways to memorize these services is to look at the transformation.

## Audio / speech

```text
Audio
 ↓
Transcribe
 ↓
Text
```

## Text → audio

```text
Text
 ↓
Polly
 ↓
Speech
```

## Text → another language

```text
Text
 ↓
Translate
 ↓
Another language
```

## Text → meaning

```text
Text
 ↓
Comprehend
 ↓
Sentiment / entities / key phrases / insights
```

## Scanned document → structured information

```text
Scanned document
 ↓
Textract
 ↓
Text / tables / form fields
```

---

# The Important Use-Case Patterns

```text
Image / video
→ Rekognition

Enterprise document search
→ Kendra

Recommendations
→ Personalize

Time-series forecasting
→ Forecast

Chatbot
→ Lex

Custom ML model
→ SageMaker
```

---

# Common Question Patterns

> **"A company needs to analyze images and detect faces and objects."**

→ **Amazon Rekognition**

---

> **"Call recordings need to be converted into text."**

→ **Amazon Transcribe**

---

> **"An application needs to read text aloud."**

→ **Amazon Polly**

---

> **"Text needs to be translated from English to French."**

→ **Amazon Translate**

---

> **"Customer reviews need sentiment analysis."**

→ **Amazon Comprehend**

---

> **"A company needs to automatically extract fields and tables from scanned invoices."**

→ **Amazon Textract**

---

> **"Employees need to search internal company documents using natural-language queries."**

→ **Amazon Kendra**

---

> **"An e-commerce application needs to provide personalized product recommendations."**

→ **Amazon Personalize**

---

> **"A company wants to predict future demand using historical time-series data."**

→ **Amazon Forecast**

---

> **"A company wants to build a customer-service chatbot."**

→ **Amazon Lex**

---

> **"Data scientists need to build, train, tune, and deploy a custom ML model."**

→ **Amazon SageMaker**

---

# Important SAA Traps

## Rekognition vs Textract

```text
Image / video analysis
→ Rekognition
```

```text
Scanned documents / tables / forms
→ Textract
```

### Example

> "Detect whether a person appears in a video."

→ **Rekognition**

> "Extract invoice numbers, customer names, totals, and table data from scanned invoices."

→ **Textract**

---

## Transcribe vs Polly

Remember the direction:

```text
Speech → text
→ Transcribe
```

```text
Text → speech
→ Polly
```

---

## Translate vs Comprehend

These both work with text, but they do different things.

```text
English → French
→ Translate
```

```text
"Is this review positive or negative?"
→ Comprehend
```

### Memory

> **Translate = change the language**

> **Comprehend = understand the text**

---

## Kendra vs Comprehend

These can both work with text, but the question's objective matters.

```text
Analyze text
→ Comprehend
```

```text
Search company documents
→ Kendra
```

---

## Personalize vs Forecast

These are both predictive services, but they predict different things.

```text
What product should this user see?
→ Personalize
```

```text
How much demand will we have next month?
→ Forecast
```

---

## Lex vs SageMaker

```text
Build a conversational chatbot
→ Lex
```

```text
Build / train / deploy a custom ML model
→ SageMaker
```

---

# AI / ML Decision Tree

```text
What is the input or goal?
          │
          ├── Image / video?
          │       ↓
          │   Rekognition
          │
          ├── Speech → text?
          │       ↓
          │   Transcribe
          │
          ├── Text → speech?
          │       ↓
          │     Polly
          │
          ├── Translate language?
          │       ↓
          │   Translate
          │
          ├── Analyze text / sentiment?
          │       ↓
          │   Comprehend
          │
          ├── Scanned document / forms / tables?
          │       ↓
          │   Textract
          │
          ├── Search company documents?
          │       ↓
          │    Kendra
          │
          ├── Personalized recommendations?
          │       ↓
          │   Personalize
          │
          ├── Future time-series values?
          │       ↓
          │   Forecast
          │
          ├── Conversational chatbot?
          │       ↓
          │     Lex
          │
          └── Custom ML model?
                  ↓
               SageMaker
```

---

# Pocket Card

| Keyword                                      | Answer          |
| -------------------------------------------- | --------------- |
| Images / video                               | **Rekognition** |
| Faces / objects / people                     | **Rekognition** |
| Speech → text                                | **Transcribe**  |
| Call recording → transcript                  | **Transcribe**  |
| Text → speech                                | **Polly**       |
| Application reads text aloud                 | **Polly**       |
| Text → another language                      | **Translate**   |
| Sentiment / text analysis                    | **Comprehend**  |
| Scanned documents / invoices                 | **Textract**    |
| Tables / forms / fields from documents       | **Textract**    |
| Enterprise document search                   | **Kendra**      |
| Natural-language search of company documents | **Kendra**      |
| Recommendations                              | **Personalize** |
| User behavior → recommendations              | **Personalize** |
| Time-series forecasting                      | **Forecast**    |
| Future sales / demand                        | **Forecast**    |
| Chatbot                                      | **Lex**         |
| Conversational interface                     | **Lex**         |
| Build/train/deploy custom ML model           | **SageMaker**   |

---

# Final Memory

```text
Rekognition
= IMAGE / VIDEO ANALYSIS

Transcribe
= SPEECH → TEXT

Polly
= TEXT → SPEECH

Translate
= TEXT → ANOTHER LANGUAGE

Comprehend
= UNDERSTAND TEXT
= SENTIMENT / ENTITIES / KEY PHRASES

Textract
= SCANNED DOCUMENTS
= TEXT / TABLES / FORMS

Kendra
= ENTERPRISE DOCUMENT SEARCH

Personalize
= RECOMMENDATIONS

Forecast
= TIME-SERIES FORECASTING

Lex
= CHATBOT

SageMaker
= CUSTOM ML
= BUILD / TRAIN / TUNE / DEPLOY MODELS
```

# The Golden Rule

```text
Image / video
→ Rekognition

Speech → text
→ Transcribe

Text → speech
→ Polly

Text → another language
→ Translate

Text meaning / sentiment
→ Comprehend

Scanned document → structured information
→ Textract

Search company documents
→ Kendra

Recommendations
→ Personalize

Time-series forecasting
→ Forecast

Chatbot
→ Lex

Build your own ML model
→ SageMaker
```

> **Don't memorize the implementation.**
>
> **Memorize the unique signal.**

For example:

```text
Image / video       → Rekognition
Speech → text       → Transcribe
Text → speech       → Polly
Translation         → Translate
Sentiment           → Comprehend
Scanned invoice     → Textract
Company search      → Kendra
Recommendations     → Personalize
Future demand       → Forecast
Chatbot             → Lex
Custom ML           → SageMaker
```
