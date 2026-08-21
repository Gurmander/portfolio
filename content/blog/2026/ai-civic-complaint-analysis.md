---
title: "Integrating Computer Vision into a Civic Complaint Platform"
date: 2026-08-21
description: "How I integrated object detection and vision models into a FastAPI-based civic complaint workflow."
categories:
  - AI / ML
tags:
  - FastAPI
  - YOLO
  - Computer Vision
  - Gemini Vision
  - Python
draft: false
---

For **Urban Fix**, our Software Engineering project, I worked on integrating AI-assisted image analysis into a civic complaint management system.

The basic idea was simple: when a citizen reports a civic issue and uploads an image, the system should use the image as additional information rather than relying entirely on manually entered text.

## The complaint workflow

A citizen can report issues such as:

- potholes
- garbage
- drainage problems
- waterlogging
- other civic infrastructure issues

The complaint then moves through municipal workflows involving officials and ward supervisors.

The AI component runs during the complaint-reporting process.

```text
Citizen uploads image
        │
        ▼
Complaint category
        │
        ▼
AI image analysis
        │
        ├── Object detection
        │
        └── Vision model analysis
        │
        ▼
Severity / description
        │
        ▼
Citizen reviews complaint
        │
        ▼
      Submit
```

## Using YOLO for known categories

For categories where specialized object-detection models were available, I used **YOLO-based detection**.

For example, pothole and garbage images can be processed by models trained to recognize those objects.

A simplified result might look like:

```text
Image
  ↓
YOLO Model
  ↓
Detected objects
  ↓
Bounding boxes + confidence
  ↓
Severity assessment
```

Using a specialized model makes sense when the target object is well defined.

## Supporting broader civic issues

Not every civic problem can easily be represented using a single object-detection class.

Issues such as drainage problems or waterlogging may require a broader understanding of what is happening in an image.

For those situations, the application can use **vision-language models**, including Gemini Vision and NVIDIA-based vision models, to provide contextual analysis.

This creates a hybrid approach:

```text
                   Complaint Image
                         │
                         ▼
                   Category Check
                    /           \
                   /             \
          Known YOLO class     Other issue
                 │                 │
                 ▼                 ▼
              YOLO           Vision Model
                 │                 │
                 └──────┬──────────┘
                        ▼
               Complaint Assessment
```

## Connecting AI with FastAPI

The model itself is only one part of the system.

The backend needs to:

1. receive the uploaded image,
2. determine the appropriate analysis path,
3. run the model,
4. interpret the result,
5. return structured information to the frontend.

FastAPI worked well for this because the AI functionality could be exposed through normal API endpoints.

The rest of the application did not need to know how each model worked internally.

It only needed a predictable response.

## Handling slow model calls

One practical problem I encountered was that external vision-model requests could take significantly longer than ordinary API operations.

A slow synchronous operation can block backend request handling if it is executed directly in the application event loop.

For blocking operations, I used an approach that allowed the work to execute outside the main asynchronous request path.

This was an important reminder that integrating an AI model into a web application is not just an ML problem.

It is also a backend and systems-design problem.

## AI should assist, not replace the workflow

The application does not rely on AI to make every final decision.

Instead, the model provides additional information such as:

* detected issue
* severity
* suggested description
* contextual assessment

The user can still review the complaint before submitting it.

Municipal officials and supervisors continue to manage the actual complaint lifecycle.

That separation is important because model predictions are not always correct.

## What I learned

Integrating computer vision into a real application required thinking beyond model accuracy.

I had to consider:

* which model should handle which category
* API design
* image uploads
* model latency
* fallback behaviour
* structured model outputs
* how predictions fit into the existing application workflow

The main lesson was that an AI feature becomes useful only when it fits naturally into the surrounding software system.

The model is one component — the backend architecture and user workflow determine whether the feature is actually practical.
