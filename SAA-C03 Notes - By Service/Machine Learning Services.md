---
course: Ultimate AWS Certified Solutions Architect Associate 2026
service: Machine Learning Services
version: C (by service)
source_chapters: [23]
related: [S3, Kinesis & Firehose, Lambda, EventBridge]
tags: [aws, saa-c03, machine-learning, rekognition, transcribe, polly, translate, lex, connect, comprehend, sagemaker, kendra, personalize, textract]
---

# Machine Learning Services

Concept-only note merged from chapter 23. Lab/demo narration is left out (see [[23 - Machine Learning]] in Version B). For the exam, the skill is mapping a use case to the right service. Related: [[S3]], [[Kinesis & Firehose]], [[Lambda]].

## 1. Rekognition (images and video)
(src: 23/01-Rekognition Overview)
- Finds **objects, people, text, scenes** in images and videos.
- Features: labeling, **content moderation**, text detection, face detection and analysis (gender, age range, emotions), face search and verification, **celebrity recognition**, pathing (e.g. tracking players in a sports video).
- Use your own database of familiar faces or compare against celebrities.
- **Content moderation** detects inappropriate, unwanted or offensive content (social networks, broadcast media, advertising, e-commerce):
  1. Rekognition analyzes the image.
  2. You set a **Minimum Confidence Threshold** for flagging; lower threshold = more matches.
  3. Optional **manual review with Amazon Augmented AI (A2I)** for flagged items.
  - Helps comply with regulations requiring detection before content is posted.

> [!tip] Exam
> Rekognition = images/video; content moderation with confidence threshold, then human review via A2I.

## 2. Transcribe, Polly, Translate
(src: 23/02-Transcribe Overview, 23/03-Polly Overview, 23/04-Translate Overview)

| Service | Function | Key points |
|---|---|---|
| **Transcribe** | speech to text | deep learning **ASR** (Automatic Speech Recognition); **auto-redact PII** (name, age, SSN); **automatic language identification** for multilingual audio; use cases: customer service calls, closed captioning/subtitles, searchable media metadata |
| **Polly** | text to speech (opposite of Transcribe) | **Pronunciation lexicons** (stylized words, acronyms such as AWS -> "Amazon Web Services"; uploaded, used with `SynthesizeSpeech`); **SSML** (Speech Synthesis Markup Language: emphasis, phonetic pronunciation, breathing, whispering, newscaster style, breaks) |
| **Translate** | natural, accurate language translation | localize websites/apps for international users; large volumes of text |

> [!tip] Exam
> Polly: stylized words/acronyms -> **lexicons**; whisper/phonetic/more control -> **SSML**.

## 3. Lex and Connect
(src: 23/05-Lex + Connect Overview)
- **Lex**: same technology as **Alexa**; **ASR** (speech to text) plus **natural language understanding** (recognizes intent); builds **chatbots and call center bots**.
- **Connect**: cloud-based **virtual contact center**; receive calls, create contact flows; integrates with CRMs and AWS services; **no upfront payment**, about **80% cheaper** than traditional contact center solutions [verify]
> [!warning] Correction [note]
> The 80% figure is the instructor's claim; I did not find an AWS source for it, so it is unconfirmed.
- Flow example: customer calls an Amazon Connect number -> Lex streams the call and understands the intent (e.g. schedule a meeting) -> invokes the right **Lambda function** -> function updates the CRM.

> [!tip] Exam
> Lex = ASR/chatbots. Connect = contact centers.

## 4. Comprehend and Comprehend Medical
(src: 23/06-Comprehend Overview, 23/07-Comprehend Medical Overview)
- **Comprehend** = **NLP** (anything NLP on the exam). Fully managed and **serverless**; finds insights and relationships in text.
- Capabilities: language detection, key phrases/places/people/brands/events, **sentiment analysis**, tokenization and parts of speech, organizing a collection of documents **by topic**.
- Use cases: analyze customer emails for positive/negative experience drivers; group articles by discovered topics.
- **Comprehend Medical**: detects useful information in **unstructured clinical text** (doctor notes, discharge summaries, test results, case notes) using NLP; detects **PHI** (protected health information) via the **DetectPHI API**.
- Input patterns: documents in **S3** then call the API; **Kinesis Data Firehose** for real-time analysis; **Transcribe** first (voice to text) then Comprehend Medical.

[verify] Instructor refers to "Kinesis Data Firehose".
> [!warning] Correction [note]
> Unconfirmed in this session: the service has since been renamed Amazon Data Firehose (name change not checked against a source).

## 5. SageMaker AI
(src: 23/08-SageMaker AI Overview)
- **Fully managed service for developers and data scientists to build ML models**; a higher-level, more involved service than the single-purpose services above.
- Covers the whole ML process in one place without provisioning servers yourself:
  1. Gather and **label** data (e.g. survey data with the known exam score as the label).
  2. **Build** the model.
  3. **Train and tune** it.
  4. **Deploy** it and apply it to new data to make predictions (example: predict a student's exam score from experience, study time, practice exams).

## 6. Kendra
(src: 23/09-Kendra Overview)
- **Fully managed document search service** powered by ML; extracts **answers** from within documents (text, PDF, HTML, PowerPoint, Word, FAQs...).
- Indexes many data sources into an ML-powered **knowledge index**; supports **natural language search** (e.g. "where is the IT support desk?" -> "1st floor").
- **Incremental learning** from user interactions/feedback to promote preferred results; fine-tune by data importance, freshness, custom filters.

> [!tip] Exam
> Document search service -> Kendra.

## 7. Personalize
(src: 23/10-Personalize Overview)
- Fully managed ML service for **real-time personalized recommendations** (product recommendations, re-ranking, customized direct marketing); same technology as Amazon.com.
- Data in from **S3** and/or the Personalize API (real-time integration); exposes a **customized personalization API** to websites/apps; can drive SMS/email personalization.
- Takes **days, not months**; no need to build, train and deploy your own ML. Use cases: retail, media and entertainment.

> [!tip] Exam
> Personalized recommendations -> Personalize.

## 8. Textract
(src: 23/11-Textract Overview)
- Extracts **text, handwriting and data** from scanned documents (PDFs, images, forms, tables) using ML.
- Examples: driver license fields (date of birth, document ID). Use cases: financial (invoices, reports), healthcare (medical records, insurance claims), public sector (tax forms, IDs, passports).

## 9. Summary table
(src: 23/12-Machine Learning Summary)

| Service | Remember it as |
|---|---|
| Rekognition | face detection, labeling, celebrity recognition (images/video) |
| Transcribe | audio -> text (subtitles) |
| Polly | text -> audio |
| Translate | translations |
| Lex | conversational bots / chatbots |
| Connect | cloud contact center (with Lex) |
| Comprehend | natural language processing |
| SageMaker | full ML service for developers/data scientists |
| Kendra | ML-powered document search engine |
| Personalize | real-time personalized recommendations |
| Textract | detect and extract text/data from documents |

## Not included here
- Console demos in each lecture (Rekognition site, Transcribe streaming, Polly voices, Comprehend Medical analysis) were dropped as screen-dependent.
- All 12 lectures have transcripts. No diagrams: no AWS reference was fetched.
