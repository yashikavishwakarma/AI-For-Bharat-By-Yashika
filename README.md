# SATHI - AI-Powered Civic Opportunity Platform

SATHI is an AI-driven platform designed to help underserved students and early-career individuals discover, understand, and qualify for scholarships, internships, government schemes, and skill development programs.

The platform focuses on improving access to reliable information by simplifying eligibility criteria, providing personalized recommendations, and offering multilingual support.

## Problem Statement

Many eligible students miss valuable opportunities because:

- Information is scattered across multiple websites  
- Eligibility criteria are complex and difficult to interpret  
- Guidance is not personalized  
- Language barriers limit accessibility  
- Fake or misleading opportunities create distrust  

Access to information exists, but clarity and structured guidance are often missing.

## Proposed Solution

SATHI acts as a decision-support system powered by AI. Instead of only listing opportunities, it helps users:

- Discover relevant opportunities based on their profile  
- Automatically check eligibility  
- Understand why they qualify or do not qualify  
- Receive clear next steps for application  
- Access simplified explanations in regional languages  
- Interact with a conversational AI assistant  
- View short informational videos linked to opportunities  
- Identify verified and trustworthy listings  

The goal is to move users from confusion to confident application.

## Key Features

### AI Eligibility Engine
Automatically evaluates eligibility based on user profile data and explains the result clearly with actionable next steps.

### Personalized Recommendations
Provides a goal-based opportunity feed that adapts to user preferences and interaction history.

### AI-Curated Micro-Learning Reels
Offers short, verified informational videos that explain opportunities and application processes.

### AI Chat Assistant
Allows users to ask questions and receive context-aware responses based on verified information sources.

### Multilingual and Voice Support
Supports regional languages and includes speech-to-text and text-to-speech capabilities.

### Trust and Verification Layer
Detects suspicious listings, assigns confidence scores, and enables user reporting.

## System Overview

SATHI follows a modular layered architecture consisting of:

- Presentation Layer (Web and Mobile Applications)
- API Gateway
- Application Services Layer
- AI Services Layer
- Data Layer

AI components are central to the system and power eligibility evaluation, recommendations, conversational assistance, and verification.

Detailed technical design is available in `design.md`.

## Technologies Used

Frontend:
- React
- React Native
- Tailwind CSS

Backend:
- Node.js
- Express
- Python (FastAPI)

AI and Machine Learning:
- Transformer-based NLP models
- Retrieval-Augmented Generation (RAG)
- Recommendation algorithms

Database and Storage:
- PostgreSQL
- MongoDB
- Redis
- Cloud Storage

## Target Users

- Students from underserved communities  
- First-generation college students  
- Early-career job seekers  

## Vision

To ensure that no opportunity is missed due to lack of clarity, accessibility, or structured guidance.
