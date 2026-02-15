# Requirements Document

## Introduction

CareerPath India is an AI-powered career guidance platform designed specifically for Indian youth. The system provides personalized career recommendations, learning roadmaps with free resources, AI-powered career counseling through a chatbot, and resume/LinkedIn profile building capabilities. The platform supports 13 Indian languages to ensure accessibility across diverse linguistic backgrounds and helps students make informed career decisions based on their education level, interests, location, and aspirations.

The platform consists of a FastAPI backend integrated with Google Gemini AI and a React + TypeScript frontend. It features an intelligent fallback system that uses a comprehensive career pool database (80+ careers) and predefined roadmaps when AI services are unavailable, ensuring continuous service availability.

## Glossary

- **System**: The CareerPath India platform (backend API + frontend web application)
- **Student**: A user of the platform seeking career guidance
- **Profile**: A collection of student information including education, interests, location, and career preferences
- **Career_Suggestion**: A recommended career path with match score, salary expectations, and required skills
- **Roadmap**: A structured learning path with sequential steps and free resources for a specific career
- **AI_Engine**: The Google Gemini-powered component that generates career recommendations and responses
- **Career_Pool**: A database of 80+ careers with metadata for intelligent fallback scoring
- **Match_Score**: A numerical value (60-96) indicating how well a career aligns with a student's profile
- **Free_Resource**: A learning material (course, video, article, or government scheme) available at no cost
- **Chat_Session**: A conversational interaction between a student and the AI career counselor
- **Resume_Builder**: The component that generates professional resumes and LinkedIn profiles
- **Excel_Store**: The data persistence layer that saves user information to Excel files
- **Language**: One of 13 supported Indian languages (English, Hindi, Tamil, Telugu, Bengali, Marathi, Kannada, Malayalam, Gujarati, Punjabi, Odia, Assamese, Urdu)
- **Stream**: Educational stream (Science PCM, Science PCB, Commerce, Arts/Humanities, Vocational)
- **Field**: Career category (Technology, Engineering, Business, Healthcare, Creative & Design, etc.)
- **Pay_Tier**: Salary expectation category (Low: Below ₹6 LPA, Medium: ₹6-15 LPA, High: Above ₹15 LPA)
- **Education_Rank**: Numeric ranking of education levels (1=10th, 2=12th, 3=Diploma, 4=UG, 5=Graduate, 6=Postgraduate, 7=PhD)

## Requirements

### Requirement 1: Multi-Language Support

**User Story:** As a student from any Indian state, I want to use the platform in my preferred language, so that I can understand career guidance in my native language.

#### Acceptance Criteria

1. THE System SHALL support 13 Indian languages: English, Hindi, Tamil, Telugu, Bengali, Marathi, Kannada, Malayalam, Gujarati, Punjabi, Odia, Assamese, and Urdu
2. WHEN a student selects a language, THE System SHALL display all UI text in the selected language
3. WHEN the AI_Engine generates content, THE System SHALL generate career descriptions, roadmap steps, and chat responses in the student's selected language
4. THE System SHALL maintain consistent terminology across all language translations
5. WHEN a student changes language, THE System SHALL persist the language preference throughout the session

### Requirement 2: Student Profile Collection

**User Story:** As a student, I want to provide my educational background and career preferences, so that the system can give me personalized recommendations.

#### Acceptance Criteria

1. THE System SHALL collect the student's name, citizenship status, education level, stream/branch, degree, subjects, interests, preferred field, annual pay expectation, and job location
2. WHEN a student enters profile information, THE System SHALL validate that required fields (name, citizenship, education level, interests, preferred field, pay expectation, location) are provided
3. THE System SHALL support education levels: 10th Pass, 12th Pass, Diploma, Undergraduate (Pursuing), Graduate, Postgraduate, PhD
4. THE System SHALL support streams: Science (PCM), Science (PCB), Commerce, Arts/Humanities, Vocational, Not Applicable
5. THE System SHALL allow students to select multiple interests from a predefined list
6. THE System SHALL support pay expectations: Below ₹3 LPA, ₹3-₹6 LPA, ₹6-₹10 LPA, ₹10-₹15 LPA, ₹15-₹25 LPA, Above ₹25 LPA, Not Sure
7. THE System SHALL support major Indian cities and regions as job location preferences

### Requirement 3: AI-Powered Career Analysis

**User Story:** As a student, I want to receive personalized career suggestions based on my profile, so that I can explore careers that match my background and interests.

#### Acceptance Criteria

1. WHEN a student submits their profile, THE System SHALL generate exactly 5 career suggestions
2. THE AI_Engine SHALL rank careers based on education level, stream alignment, interest overlap, pay expectations, and field preferences
3. THE System SHALL NOT suggest careers requiring higher education than the student possesses
4. WHEN a student has Science (PCB) stream, THE System SHALL prioritize healthcare and biology-related careers over pure engineering careers
5. WHEN a student has Science (PCM) stream, THE System SHALL prioritize technology and engineering careers over medical careers
6. THE System SHALL ensure career suggestions span at least 2-3 different fields for diversity
7. FOR ALL career suggestions, THE System SHALL provide a match score between 60 and 96
8. FOR ALL career suggestions, THE System SHALL include title, description, expected salary in Indian Rupees (LPA), required skills list, and field category
9. THE System SHALL apply stream mismatch penalties to careers that don't align with the student's educational stream
10. WHEN the AI_Engine is unavailable, THE System SHALL use a fallback scoring algorithm based on the career pool database
11. THE fallback scoring algorithm SHALL use weighted multi-signal scoring with field match (15 pts), stream alignment (30 pts with -15 penalty for mismatch), interest overlap via Jaccard similarity (30 pts), education fit (10 pts), pay tier match (10 pts), and degree keyword bonus (5 pts)
12. THE System SHALL expand user interests into semantic tags using the INTEREST_TAGS mapping
13. THE System SHALL inject stream-derived tags using STREAM_INTEREST_BOOST for implicit matching
14. THE System SHALL apply diversity bonus (5 pts) to careers from underrepresented fields in the result set
15. THE System SHALL limit careers from the same field to maximum 3 out of 5 suggestions
16. THE System SHALL normalize raw scores into the 60-96 range for consistent match score presentation

### Requirement 4: Learning Roadmap Generation

**User Story:** As a student, I want to see a detailed learning roadmap for my chosen career, so that I know what steps to take and what free resources are available.

#### Acceptance Criteria

1. WHEN a student selects a career, THE System SHALL generate a learning roadmap with 5-7 sequential steps
2. FOR ALL roadmap steps, THE System SHALL include step number, title, description, duration estimate, and 2-4 free resources
3. THE System SHALL prioritize Indian government platforms (NPTEL, SWAYAM, PMKVY, Skill India) in resource recommendations
4. THE System SHALL include resources from YouTube, freeCodeCamp, Khan Academy, and free audit courses from Coursera/edX
5. FOR ALL resources, THE System SHALL provide name, actual URL, platform name, and resource type (course/video/article/scheme)
6. THE System SHALL calculate and display total duration for completing the entire roadmap
7. WHEN a specific career has a predefined roadmap in the career_resources database, THE System SHALL use that roadmap
8. WHEN no predefined roadmap exists, THE AI_Engine SHALL generate a custom roadmap
9. THE System SHALL ensure all recommended resources are genuinely free and accessible in India
10. THE System SHALL support fuzzy matching for career titles when looking up predefined roadmaps (case-insensitive substring matching)
11. WHEN using predefined roadmaps, THE System SHALL calculate total duration by summing step durations and converting to appropriate units (weeks or months)
12. WHEN AI is unavailable and no predefined roadmap exists, THE System SHALL generate a generic 3-step fallback roadmap with government resources

### Requirement 5: AI Career Counseling Chatbot

**User Story:** As a student, I want to chat with an AI counselor about my career questions, so that I can get personalized advice and clarifications.

#### Acceptance Criteria

1. THE System SHALL provide a chat interface accessible from any page in the application
2. WHEN a student sends a message, THE AI_Engine SHALL respond in the student's selected language
3. THE System SHALL maintain conversation history within a chat session
4. THE AI_Engine SHALL provide advice specific to the Indian education system, entrance exams (JEE, NEET, CAT, UPSC), and job market
5. THE AI_Engine SHALL recommend free learning resources available in India
6. THE AI_Engine SHALL maintain a friendly, encouraging, and supportive tone like a helpful older sibling
7. THE System SHALL pass student profile context to the AI_Engine for personalized responses
8. WHEN the AI_Engine is unavailable, THE System SHALL provide predefined responses for common topics (identity, greetings, salary, resources, exams)
9. THE System SHALL limit responses to 2-4 short paragraphs for readability
10. THE AI_Engine SHALL use bullet points or numbered lists when presenting multiple options
11. THE System SHALL convert chat history to Gemini-compatible format (using "user" and "model" roles)
12. THE AI_Engine SHALL use system instructions to define personality, expertise, and response rules
13. THE System SHALL support multi-turn conversations by maintaining and passing conversation history
14. THE fallback responses SHALL handle identity questions, greetings, salary inquiries, resource requests, and exam guidance

### Requirement 6: Resume and LinkedIn Profile Builder

**User Story:** As a student, I want to generate a professional resume and LinkedIn profile, so that I can apply for jobs effectively.

#### Acceptance Criteria

1. THE System SHALL collect work experience, school education, college education, projects, and skills from the student
2. FOR ALL experience entries, THE System SHALL capture title, company, duration, and description
3. FOR ALL school education entries, THE System SHALL capture class level (10th/12th), board, school name, percentage, and year
4. FOR ALL college education entries, THE System SHALL capture degree, stream, college name, CGPA/percentage, and year
5. FOR ALL project entries, THE System SHALL capture title, role, description, and technologies used
6. WHEN a student submits resume information, THE AI_Engine SHALL generate a professional ATS-friendly resume in Markdown format
7. WHEN a student submits resume information, THE AI_Engine SHALL generate an optimized LinkedIn profile with headline and About section in Markdown format
8. THE System SHALL include a professional summary in the generated resume
9. THE System SHALL suggest 5 top skills to pin on LinkedIn
10. THE System SHALL format the resume with proper section headers (Professional Summary, Education, Experience, Projects, Skills)
11. WHEN the AI_Engine is unavailable, THE System SHALL generate a basic template resume with the provided information
12. THE LinkedIn profile SHALL include a catchy headline and first-person "About" section
13. THE System SHALL use ## for section headers in the resume markdown
14. THE System SHALL parse AI responses that may contain JSON in markdown code blocks
15. THE resume builder SHALL be accessible from the language selection page and from the results page

### Requirement 7: User Data Persistence

**User Story:** As a platform administrator, I want to save user data for analytics and improvement, so that we can understand user needs and improve the platform.

#### Acceptance Criteria

1. WHEN a student completes career analysis, THE System SHALL save their data to an Excel file
2. THE Excel_Store SHALL capture timestamp, name, language, career paths, interests, degree, and location
3. THE System SHALL create the Excel file with headers if it doesn't exist
4. THE System SHALL append new user data as rows in the Excel file
5. THE System SHALL format the Excel header row with bold text
6. THE System SHALL handle missing optional fields by storing "Not specified"
7. THE System SHALL store career paths and interests as comma-separated values

### Requirement 8: API Health Monitoring

**User Story:** As a developer, I want to check the API health and AI availability, so that I can monitor system status.

#### Acceptance Criteria

1. THE System SHALL provide a health check endpoint at /api/health
2. WHEN the health endpoint is called, THE System SHALL return status "ok" and a message
3. THE System SHALL indicate whether the Gemini AI engine is available or unavailable
4. THE System SHALL respond to health checks within 1 second

### Requirement 9: Career Pool Database

**User Story:** As the system, I want to maintain a comprehensive career database, so that I can provide accurate fallback recommendations when AI is unavailable.

#### Acceptance Criteria

1. THE System SHALL maintain a career pool with at least 80 diverse career options
2. FOR ALL careers in the pool, THE System SHALL store title, description, salary range, required skills, field, tags, compatible streams, minimum education level, and pay tier
3. THE System SHALL categorize careers into fields: Technology, Engineering, Business, Healthcare, Creative & Design, Education, Government Services, Law, Agriculture, Media & Communication, Science & Research, Hospitality & Tourism, Sports & Fitness, Skilled Trades
4. THE System SHALL map education levels to numeric ranks (1=10th, 2=12th, 3=Diploma, 4=UG, 5=Graduate, 6=Postgraduate, 7=PhD)
5. THE System SHALL map pay expectations to tiers (1=Low: Below ₹6 LPA, 2=Medium: ₹6-15 LPA, 3=High: Above ₹15 LPA)
6. THE System SHALL maintain interest-to-tag mappings for semantic matching (INTEREST_TAGS)
7. THE System SHALL maintain stream-to-interest boost mappings for implicit matching (STREAM_INTEREST_BOOST)
8. THE Career_Pool SHALL include careers spanning all education levels from 10th pass to PhD
9. THE Career_Pool SHALL include careers from all major streams (PCM, PCB, Commerce, Arts, Vocational)
10. THE Career_Pool SHALL include salary ranges in Indian Rupees (LPA format)
11. THE Career_Pool SHALL include 3-5 required skills for each career
12. THE Career_Pool SHALL include semantic tags for intelligent interest matching
13. THE INTEREST_TAGS mapping SHALL cover at least 15 common interest categories
14. THE STREAM_INTEREST_BOOST mapping SHALL inject 5-7 relevant tags per stream

### Requirement 9A: Career Resources Database

**User Story:** As the system, I want to maintain predefined learning roadmaps for popular careers, so that I can provide high-quality curated resources even when AI is unavailable.

#### Acceptance Criteria

1. THE System SHALL maintain a career resources database with predefined roadmaps for popular careers
2. FOR ALL predefined roadmaps, THE System SHALL include 5-7 sequential learning steps
3. FOR ALL roadmap steps, THE System SHALL include step number, title, description, duration, and 2-4 free resources
4. FOR ALL resources, THE System SHALL include actual working URLs to free learning platforms
5. THE System SHALL prioritize Indian government platforms (NPTEL, SWAYAM, PMKVY, Skill India) in predefined roadmaps
6. THE System SHALL include YouTube playlists from reputable educators (CodeWithHarry, Apna College, Krish Naik, freeCodeCamp, etc.)
7. THE System SHALL include resources from freeCodeCamp, Khan Academy, Coursera (audit mode), and other free platforms
8. THE predefined roadmaps SHALL cover high-demand careers (Software Developer, Data Scientist, Cybersecurity Analyst, AI/ML Engineer, etc.)
9. THE System SHALL provide helper functions (_r, _step) for consistent roadmap data structure
10. THE System SHALL organize resources by type (course, video, article, scheme)

### Requirement 10: Error Handling and Fallback

**User Story:** As a student, I want the system to work even when AI services are unavailable, so that I can still get career guidance.

#### Acceptance Criteria

1. WHEN the Gemini API is unavailable, THE System SHALL use the career pool database for recommendations
2. WHEN the Gemini API is unavailable, THE System SHALL use predefined roadmaps from the career resources database
3. WHEN the Gemini API is unavailable, THE System SHALL provide predefined chat responses for common questions
4. WHEN the Gemini API is unavailable, THE System SHALL generate basic template resumes
5. THE System SHALL log AI availability status at startup
6. WHEN API calls fail, THE System SHALL return HTTP 500 with error details
7. THE System SHALL handle JSON parsing errors when extracting AI responses from markdown code blocks
8. THE System SHALL attempt direct JSON parsing first, then extract from code blocks, then search for JSON patterns
9. THE System SHALL try multiple regex patterns to extract JSON from AI responses (code blocks, arrays, objects)
10. THE System SHALL gracefully degrade to fallback functionality without exposing errors to users
11. THE System SHALL check for GEMINI_API_KEY environment variable before initializing AI
12. THE System SHALL set gemini_available flag based on successful SDK initialization
13. THE System SHALL print clear build-time messages indicating AI availability status

### Requirement 11: CORS and Security

**User Story:** As a developer, I want the API to be accessible from the frontend, so that the application works correctly.

#### Acceptance Criteria

1. THE System SHALL enable CORS to allow requests from any origin during development
2. THE System SHALL accept credentials in cross-origin requests
3. THE System SHALL allow all HTTP methods and headers in CORS configuration
4. THE System SHALL load the Gemini API key from environment variables
5. THE System SHALL NOT expose the API key in responses or logs

### Requirement 12: Frontend User Experience

**User Story:** As a student, I want an intuitive and visually appealing interface, so that I can easily navigate the platform.

#### Acceptance Criteria

1. THE System SHALL provide a single-page application with step-based navigation
2. THE System SHALL support navigation steps: language selection, profile form, results, roadmap, resume form, resume view
3. THE System SHALL display a "Start Over" button to reset the application state
4. THE System SHALL show loading indicators during API calls
5. THE System SHALL display error messages in a dismissible banner
6. THE System SHALL show career cards with match scores, salary, skills, and action buttons
7. THE System SHALL format roadmap steps with duration, description, and clickable resource links
8. THE System SHALL provide a floating chat panel accessible from any page
9. THE System SHALL display the chat panel as expandable/collapsible
10. THE System SHALL render resume and LinkedIn profile content in Markdown format
11. THE System SHALL display the brand name "CareerPath India 🇮🇳" in the header
12. THE System SHALL make the brand name clickable to trigger "Start Over" functionality
13. THE System SHALL show a "Build Your Resume" button on the language selection page
14. THE System SHALL show a "Build Your Resume" button on the results page after career suggestions
15. THE System SHALL maintain language preference across all steps
16. THE System SHALL use translations for all UI text based on selected language
17. THE System SHALL save user data to Excel automatically after career analysis
18. THE System SHALL handle API errors gracefully with user-friendly error messages
19. THE System SHALL show loading overlay with spinner and message during AI analysis
20. THE System SHALL organize career results in a grid layout
