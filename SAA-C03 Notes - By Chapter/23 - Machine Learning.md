---
course: Ultimate AWS Certified Solutions Architect Associate 2026
chapter: 23
chapter_title: Machine Learning
version: B (by chapter)
services: [Rekognition, Transcribe, Polly, Translate, Lex, Connect, Comprehend, Comprehend Medical, SageMaker AI, Kendra, Personalize, Textract, Augmented AI]
tags: [aws, saa-c03, machine-learning, ai-services]
---

# 23 - Machine Learning

Related: [[Machine Learning Services]] (Version C service note) · [[S3]] · [[SNS]] · [[Lambda]] · [[Kinesis & Firehose]]

## Chapter summary
- Exam goal: map a **use case keyword to the ML service**. Learn the whole list.
- **Rekognition** = images/videos (faces, labels, celebrities, content moderation, text, pathing); **Transcribe** = speech to text; **Polly** = text to speech; **Translate** = translation.
- **Lex** = chatbots (same tech as Alexa, ASR + NLU); **Connect** = cloud contact center (no upfront payment, ~80% cheaper than traditional).
- **Comprehend** = NLP (sentiment, key phrases, language, topics); **Comprehend Medical** = NLP on clinical text (DetectPHI API).
- **SageMaker AI** = build, train, tune and deploy your own ML models (for developers and data scientists, not a single-purpose API).
- **Kendra** = ML-powered document search; **Personalize** = real-time personalized recommendations; **Textract** = extract text/data from scanned documents, forms, tables.
- Rekognition content moderation: **minimum confidence threshold** + optional human review with **Amazon Augmented AI (A2I)**.
- Polly: **Pronunciation lexicons** for stylized words/acronyms, **SSML** for speech control (whisper, breaks, phonetics, newscaster style).

---

## 01 - Rekognition Overview
(src: 23/01-Rekognition Overview)

- Finds objects, people, text and scenes in **images and videos** using ML.
- Features: labeling, **content moderation**, text detection, face detection and analysis (gender, age range, emotions), face search and verification, celebrity recognition, **pathing** (e.g. tracking player movement in a sports video). You can also build your own database of familiar faces.
- **Content moderation**: detects inappropriate, unwanted or offensive content (social networks, broadcast media, advertising, e-commerce) to give a safe user experience.
  1. Image analyzed by Rekognition.
  2. You set a **Minimum Confidence Threshold** for flagged items; lower threshold = more matches. The value is how confident Rekognition is that the item is really inappropriate.
  3. Optional **manual review** of flagged images with **Amazon Augmented AI (A2I)**.
  4. Helps comply with regulations requiring detection of such content before posting.

> [!tip] Exam
> Rekognition = images and videos. Content moderation = confidence threshold, then A2I for human review.

---

## 02 - Transcribe Overview
(src: 23/02-Transcribe Overview)

- Converts **speech to text** with deep learning (**ASR**, Automatic Speech Recognition), quickly and accurately.
- Features: automatic **PII redaction** (e.g. names, age, Social Security Number, phone numbers) and **automatic language identification** for multilingual audio (e.g. English + French).
- Use cases: transcribe customer service calls, automated closed captioning/subtitling, metadata for media assets (searchable archive).

### Hands-on steps
1. Transcribe console -> create a transcript / start streaming (language English US); speak and watch the live text.
2. Enable PII identification and redaction; repeat with a spoken name and phone number (they appear hidden).
3. Choose automatic language identification (English + French) and stream both languages.

> [!tip] Exam
> Transcribe = speech to text, PII redaction, automatic language identification.

---

## 03 - Polly Overview
(src: 23/03-Polly Overview)

- Opposite of Transcribe: **text to speech** with deep learning, to build applications that talk.
- **Pronunciation lexicons**: customize pronunciation of stylized words (e.g. "St3ph4ne" read as Stephane) and acronyms (replace "AWS" with "Amazon Web Services"). Upload lexicons and use them in the `SynthesizeSpeech` operation.
- **SSML (Speech Synthesis Markup Language)**: finer control instead of plain text - emphasize words, phonetic pronunciation, breathing, whispering, Newscaster style, breaks (e.g. a 3-second break).

| Need | Use |
|---|---|
| Stylized words / acronyms | Pronunciation lexicon |
| Whisper, breaks, phonetic pronunciation, speaking style | SSML |

### Hands-on steps
1. Polly console -> choose engine (neural = most natural) and voice; type text and listen.
2. Add an SSML break tag to the text and listen again.
3. Additional settings -> Customize pronunciation -> apply an uploaded lexicon (file mapping AWS to Amazon Web Services).

> [!tip] Exam
> Stylized words/acronyms = lexicon. Whisper/phonetics/pauses = SSML.

---

## 04 - Translate Overview
(src: 23/04-Translate Overview)

- Natural and accurate **language translation**; localize websites and applications for international users; translates large volumes of text efficiently.

---

## 05 - Lex + Connect Overview
(src: 23/05-Lex + Connect Overview)

- **Amazon Lex**: same technology as Alexa. **ASR** (speech to text) + **natural language understanding** (understands intent of text/callers). Used to build **chatbots** and **call center bots**.
- **Amazon Connect**: cloud-based **virtual contact center**; receive calls, create contact flows; integrates with CRMs and AWS services. **No upfront payment, about 80% cheaper** than traditional contact center solutions.
- Flow: caller phones a number defined in Connect -> Lex streams the call and detects intent -> invokes the right **Lambda function** -> Lambda acts (e.g. schedules a meeting in the CRM).

[verify] "Amazon Connect is about 80% cheaper than traditional contact center solutions."
> [!warning] Correction [note]
> Not checked against an AWS source; unconfirmed. Treat as marketing figure from the lecture.

> [!tip] Exam
> Lex = ASR/chatbots. Connect = contact center. Together = smart call center.

---

## 06 - Comprehend Overview
(src: 23/06-Comprehend Overview)

- **NLP (Natural Language Processing)**: whenever the exam says NLP, think Comprehend.
- Fully managed and **serverless**; ML to find insights and relationships in text.
- Capabilities: detect language, extract key phrases / places / people / brands / events, **sentiment analysis**, tokenization and parts of speech, organize a collection of text files by **topic**.
- Use cases: analyze customer emails for positive/negative experience drivers; group articles by auto-discovered topics.

---

## 07 - Comprehend Medical Overview
(src: 23/07-Comprehend Medical Overview)

- Detects and returns useful information from **unstructured clinical text** (doctor notes, discharge summaries, test results, case notes) using NLP; detects **protected health information (PHI)** via the **DetectPHI API**.
- Architecture inputs: documents in **S3** -> Comprehend Medical API; or **Kinesis Data Firehose** for real-time analysis; or voice -> **Transcribe** -> text -> Comprehend Medical.
- Console demo: paste a doctor's note, run real-time analysis; entities such as age, procedure name, medication (generic name, strength, dosage, route, frequency) are extracted and organized.

---

## 08 - SageMaker AI Overview
(src: 23/08-SageMaker AI Overview)

- Fully managed service for **developers and data scientists to build ML models**; higher level and harder to use than the single-purpose services above.
- Without it you must do many steps and provision servers. SageMaker helps with: **labeling** data, **building** the model, **training and tuning**, and **deploying** it for predictions.
- Instructor's example: predict a student's exam score from collected data (years of IT/AWS experience, course time, practice exams) labeled with actual scores; train, then apply to new students.

> [!tip] Exam
> Custom ML models for developers/data scientists = SageMaker.

---

## 09 - Kendra Overview
(src: 23/09-Kendra Overview)

- Fully managed **document search** service powered by ML; extracts answers from documents (text, PDF, HTML, PowerPoint, Word, FAQs, from many data sources).
- Builds an internal **knowledge index**; supports **natural language search** (e.g. "where is the IT support desk?" -> "1st floor").
- **Incremental learning** from user interactions/feedback to promote preferred results; fine-tune results by data importance, freshness, custom filters.

> [!tip] Exam
> Document search service = Kendra.

---

## 10 - Personalize Overview
(src: 23/10-Personalize Overview)

- Fully managed ML service for **real-time personalized recommendations** (product recommendations, re-ranking, customized direct marketing); same technology as Amazon.com.
- Input: data from **S3** (e.g. user interactions) and/or the Personalize **API** for real-time data; exposes a customized personalization API for websites/mobile apps; can also drive SMS/email personalization.
- Takes **days, not months**; no need to build, train and deploy ML yourself. Use cases: retail, media and entertainment.

> [!tip] Exam
> Personalized recommendations = Personalize.

---

## 11 - Textract Overview
(src: 23/11-Textract Overview)

- Extracts **text, handwriting and data** from any scanned document using AI/ML (PDFs, images, forms, tables); e.g. driver license -> date of birth, document ID.
- Use cases: financial services (invoices, financial reports), healthcare (medical records, insurance claims), public sector (tax forms, ID documents, passports).

---

## 12 - Machine Learning Summary
(src: 23/12-Machine Learning Summary)

| Service | Remember it as |
|---|---|
| Rekognition | face detection, labeling, celebrity recognition |
| Transcribe | audio to text (subtitles) |
| Polly | text to audio |
| Translate | translations |
| Lex (+ Connect) | chatbots (+ cloud contact center) |
| Comprehend | natural language processing |
| SageMaker | full ML service for developers/data scientists |
| Kendra | ML-powered document search |
| Personalize | real-time personalized recommendations |
| Textract | detect and extract text/data from documents |

---

## Not covered in this chapter's lectures
- All 12 lectures have transcripts. Hands-on content is limited to console demos in lectures 02, 03 and 07.
