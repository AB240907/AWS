# Design Document: SwasthyaMitra Healthcare Assistant

## Overview

SwasthyaMitra is a serverless, cloud-native AI healthcare assistant built on AWS infrastructure. The system employs a multi-layered architecture combining mobile/web frontends, serverless backend services, AI/ML capabilities through Amazon Bedrock, and comprehensive health data management. The design prioritizes data security, low-bandwidth optimization, multilingual support, and scalability to serve lakhs of Indian patients across diverse geographic and linguistic regions.

The core innovation lies in the AI Agent orchestration layer, which uses Amazon Bedrock agents to reason through patient queries, coordinate multiple AWS services (Textract for OCR, Polly for speech synthesis, Translate for language support, Location Service for provider discovery), and provide personalized health guidance based on individual medical history.

## Problem Statement

Indian healthcare faces critical challenges:

1. **Fragmented Medical Records**: Patients lack centralized, lifelong health records, leading to repeated tests and incomplete medical histories
2. **Language Barriers**: 500M+ Indians don't speak English, limiting access to health information
3. **Rural Healthcare Access**: Limited availability of doctors and specialists in Tier-2/3 cities and rural areas
4. **Medication Non-Adherence**: 50%+ patients don't follow prescription schedules correctly
5. **Health Literacy**: Low awareness about symptoms, preventive care, and when to seek medical attention
6. **Connectivity Challenges**: Rural areas have limited internet bandwidth
7. **Cost**: Healthcare consultations are expensive for routine questions and guidance

SwasthyaMitra addresses these challenges through AI-powered, accessible, multilingual health assistance.

## Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           USER LAYER                                     │
│  ┌──────────────────┐              ┌──────────────────┐                │
│  │  Mobile App      │              │  Web Interface   │                │
│  │  (React Native)  │              │  (React PWA)     │                │
│  └────────┬─────────┘              └────────┬─────────┘                │
└───────────┼──────────────────────────────────┼──────────────────────────┘
            │                                  │
            └──────────────┬───────────────────┘
                           │
┌──────────────────────────┼───────────────────────────────────────────────┐
│                    API GATEWAY LAYER                                     │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │  Amazon API Gateway (REST API)                                     │ │
│  │  - /auth/*  /query/*  /history/*  /ocr/*  /providers/*            │ │
│  └────────────────────────┬───────────────────────────────────────────┘ │
│  ┌────────────────────────┴───────────────────────────────────────────┐ │
│  │  Amazon Cognito (Phone Number Auth + OTP)                          │ │
│  └────────────────────────────────────────────────────────────────────┘ │
└──────────────────────────┼───────────────────────────────────────────────┘
                           │
┌──────────────────────────┼───────────────────────────────────────────────┐
│                  APPLICATION LAYER (Lambda Functions)                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌─────────────┐ │
│  │ Orchestrator │  │ OCR Processor│  │ Reminder     │  │ Provider    │ │
│  │ Lambda       │  │ Lambda       │  │ Lambda       │  │ Search      │ │
│  │ (Main Agent) │  │ (Textract)   │  │ (EventBridge)│  │ Lambda      │ │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬──────┘ │
└─────────┼──────────────────┼──────────────────┼──────────────────┼────────┘
          │                  │                  │                  │
┌─────────┼──────────────────┼──────────────────┼──────────────────┼────────┐
│         │            AI/ML LAYER              │                  │        │
│  ┌──────▼──────────────────────────────────┐  │                  │        │
│  │  Amazon Bedrock Agent (Claude 3 Sonnet) │  │                  │        │
│  │  Tools: History, Symptom, Drug Check    │  │                  │        │
│  └──────┬──────────────────────────────────┘  │                  │        │
│  ┌──────▼──────┐  ┌──────────┐  ┌───────────┐ │                  │        │
│  │  Textract   │  │  Polly   │  │ Translate │ │                  │        │
│  │  (OCR)      │  │  (TTS)   │  │ (Lang)    │ │                  │        │
│  └─────────────┘  └──────────┘  └───────────┘ │                  │        │
│  ┌──────────┐  ┌────────────────────────────┐ │                  │        │
│  │Transcribe│  │  Location Service          │◄┼──────────────────┘        │
│  │(STT)     │  │  (Provider Discovery)      │ │                           │
│  └──────────┘  └────────────────────────────┘ │                           │
└─────────┬──────────────────────────────────────┼───────────────────────────┘
          │                                      │
┌─────────┼──────────────────────────────────────┼───────────────────────────┐
│         │              DATA LAYER              │                           │
│  ┌──────▼──────────────────────────────────────▼───────────────────────┐  │
│  │  Amazon DynamoDB                                                     │  │
│  │  Tables: Users, MedicalHistory, Prescriptions, Reminders,           │  │
│  │          Appointments, Providers                                     │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │  Amazon S3                                                           │  │
│  │  Buckets: prescription-images, lab-reports, user-documents          │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │  AWS IoT Core (Optional - Wearable Devices)                         │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────────────────┘
┌───────────────────────────────────────────────────────────────────────────┐
│                    MONITORING & SCHEDULING                                │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐       │
│  │  CloudWatch      │  │  EventBridge     │  │  SNS (Push       │       │
│  │  (Logs/Metrics)  │  │  (Reminders)     │  │  Notifications)  │       │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘       │
└───────────────────────────────────────────────────────────────────────────┘
```

### Architecture Principles

1. **Serverless-First**: All compute uses AWS Lambda to minimize operational overhead and optimize costs
2. **Single Orchestrator Pattern**: Main Lambda function coordinates all AI agent interactions
3. **Event-Driven**: Medicine reminders, appointment notifications use EventBridge scheduling
4. **Security-First**: End-to-end encryption, HIPAA-aligned data handling
5. **Low-Bandwidth Aware**: Response payloads optimized, progressive enhancement for better connectivity
6. **Multi-Region Ready**: Design supports deployment across multiple AWS regions

## Components and Interfaces

### 1. Frontend Layer

**Mobile Application (React Native)**
- Cross-platform iOS/Android support
- Offline-first architecture with local caching
- Camera integration for prescription/report capture
- Voice recording and playback
- Push notification support for reminders
- Bandwidth detection and adaptive UI

**Web Interface (React PWA)**
- Progressive Web App for offline capability
- Responsive design for mobile and desktop
- Feature parity with mobile app
- Accessible via feature phones with basic browsers

**Interface Contract:**
```typescript
interface FrontendAPI {
  // Authentication
  authenticateUser(phoneNumber: string, otp: string): Promise<AuthToken>
  
  // Query submission
  submitTextQuery(query: string, language: string): Promise<Response>
  submitVoiceQuery(audioBlob: Blob, language: string): Promise<Response>
  
  // Medical history
  getMedicalHistory(filters?: HistoryFilters): Promise<MedicalRecord[]>
  addMedicalRecord(record: MedicalRecord): Promise<void>
  
  // OCR
  uploadPrescription(image: Blob): Promise<PrescriptionData>
  uploadLabReport(image: Blob): Promise<LabReportData>
  
  // Providers
  searchProviders(location: GeoLocation, type: ProviderType, specialty?: string): Promise<Provider[]>
  bookAppointment(providerId: string, slot: TimeSlot): Promise<Appointment>
  
  // Reminders
  getMedicineReminders(): Promise<Reminder[]>
  markDoseTaken(reminderId: string): Promise<void>
  
  // User preferences
  setLanguagePreference(language: string): Promise<void>
  setBandwidthMode(mode: 'low' | 'normal'): Promise<void>
}
```

### 2. API Gateway Layer

**Amazon API Gateway (REST API)**
- RESTful endpoints for all client operations
- Request validation and throttling (1000 req/sec per user)
- CORS configuration for web clients
- Integration with AWS Lambda functions

**Amazon Cognito**
- Phone number-based authentication
- OTP verification via SMS
- JWT token generation and validation
- User pool management with MFA support

**Endpoints:**
```
POST   /auth/register          - Register new patient
POST   /auth/verify            - Verify OTP
POST   /query/text             - Submit text query
POST   /query/voice            - Submit voice query
POST   /history/add            - Add medical record
GET    /history/list           - Get medical history
POST   /ocr/prescription       - Upload prescription for OCR
POST   /ocr/labreport          - Upload lab report for OCR
GET    /providers/search       - Search healthcare providers
POST   /appointments/book      - Book appointment
GET    /appointments/list      - List user appointments
GET    /reminders/list         - Get medicine reminders
POST   /reminders/mark-taken   - Mark dose as taken
GET    /user/profile           - Get user profile
PUT    /user/preferences       - Update user preferences
POST   /interactions/check     - Check drug interactions
```

### 3. Application Layer (Lambda Functions)

**Orchestrator Lambda (Main Agent Handler)**
- Receives all queries from API Gateway
- Routes to Amazon Bedrock agent
- Handles language detection and translation
- Manages conversation context
- Returns formatted responses
- Runtime: Python 3.11
- Memory: 1024 MB
- Timeout: 30 seconds

**OCR Processor Lambda**
- Receives prescription/report images from S3 upload events
- Invokes Amazon Textract for text extraction
- Parses extracted text to identify medicines, dosages, test results
- Stores structured data in DynamoDB
- Creates medicine reminders automatically
- Runtime: Python 3.11
- Memory: 512 MB
- Timeout: 60 seconds

**Medicine Reminder Lambda**
- Triggered by EventBridge scheduled rules
- Queries DynamoDB for due reminders
- Sends push notifications via SNS
- Logs reminder delivery status
- Runtime: Python 3.11
- Memory: 256 MB
- Timeout: 10 seconds

**Provider Search Lambda**
- Receives location and search criteria
- Queries Amazon Location Service for nearby providers
- Filters by specialty, ratings, availability
- Returns sorted results
- Runtime: Python 3.11
- Memory: 512 MB
- Timeout: 10 seconds

**Wearable Data Processor Lambda (Optional)**
- Consumes IoT Core messages from wearables
- Validates and stores vital signs
- Triggers alerts for abnormal readings
- Updates health trends
- Runtime: Python 3.11
- Memory: 256 MB
- Timeout: 10 seconds

### 4. AI Agent Core (Amazon Bedrock)

**Agent Configuration:**
- Foundation Model: Anthropic Claude 3 Sonnet
- Reasoning: Chain-of-thought for complex health queries
- Memory: Conversation history stored in DynamoDB
- Tools: Medical History, Symptom Checker, Drug Interaction, Provider Search
- Safety: Medical disclaimer in all responses

**Agent Tools:**

```python
# Tool 1: Medical History Retrieval
def get_medical_history(
    user_id: str,
    category: Optional[str] = None,
    date_range: Optional[DateRange] = None
) -> List[MedicalRecord]:
    """
    Retrieves patient's medical history with optional filtering.
    Args:
        user_id: Patient identifier
        category: Filter by diseases, allergies, prescriptions, reports
        date_range: Filter by date range
    Returns:
        List of medical records
    """
    pass

# Tool 2: Symptom Checker
def check_symptoms(
    symptoms: List[str],
    duration: str,
    severity: str,
    patient_history: Optional[MedicalHistory] = None
) -> SymptomAnalysis:
    """
    Analyzes symptoms and provides preliminary health guidance.
    Args:
        symptoms: List of symptoms described by patient
        duration: How long symptoms have persisted
        severity: Mild, moderate, severe
        patient_history: Optional medical history for personalization
    Returns:
        SymptomAnalysis with possible conditions, urgency, recommendations
    """
    pass

# Tool 3: Drug Interaction Checker
def check_drug_interactions(
    new_medicine: str,
    existing_medicines: List[str],
    allergies: List[str]
) -> InteractionAnalysis:
    """
    Checks for drug-drug and drug-allergy interactions.
    Args:
        new_medicine: Medicine being considered
        existing_medicines: Current prescriptions
        allergies: Known allergies
    Returns:
        InteractionAnalysis with conflicts, severity, recommendations
    """
    pass

# Tool 4: Provider Search
def search_healthcare_providers(
    location: GeoLocation,
    provider_type: str,
    specialty: Optional[str] = None,
    radius_km: int = 10
) -> List[Provider]:
    """
    Searches for nearby healthcare providers.
    Args:
        location: Patient's current location
        provider_type: doctor, hospital, clinic, pharmacy
        specialty: Medical specialty filter
        radius_km: Search radius
    Returns:
        List of providers with details and availability
    """
    pass

# Tool 5: Health Information Lookup
def get_health_information(
    topic: str,
    personalize: bool = False,
    user_id: Optional[str] = None
) -> HealthInfo:
    """
    Retrieves evidence-based health information.
    Args:
        topic: Health topic or condition
        personalize: Whether to personalize based on user history
        user_id: Patient identifier for personalization
    Returns:
        HealthInfo with description, symptoms, treatments, prevention
    """
    pass
```

**Agent Workflow:**
1. Receive patient query (text or transcribed voice)
2. Detect intent (symptom check, history query, provider search, general question)
3. Extract entities (symptoms, medicines, locations, dates)
4. Determine which tools are needed
5. Execute tool calls in sequence or parallel
6. Synthesize response from tool outputs
7. Add medical disclaimer
8. Translate response to patient's language
9. Return text response (and optionally convert to speech)

### 5. AI/ML Services

**Amazon Textract**
- Extracts text from prescription and lab report images
- Detects tables, forms, and key-value pairs
- Confidence scores for each extracted field
- Handles handwritten and printed text

**Amazon Polly**
- Text-to-speech for voice responses
- Neural voices for natural-sounding speech
- Supports Hindi, Tamil, Telugu, Kannada (using closest available voices)
- SSML support for pronunciation of medical terms

**Amazon Translate**
- Real-time translation between English and Regional_Languages
- Custom terminology for medical terms
- Batch translation for health information documents

**Amazon Transcribe**
- Speech-to-text for voice queries
- Custom vocabulary for medical terms in regional languages
- Automatic language detection
- Medical transcription mode for better accuracy

**Amazon Location Service**
- Geocoding and reverse geocoding
- Place search for healthcare providers
- Route calculation for directions
- Geofencing for location-based reminders

### 6. Data Layer

**Amazon S3**
- Bucket structure:
  - `/prescriptions/{user_id}/{timestamp}.jpg` - Prescription images
  - `/lab-reports/{user_id}/{timestamp}.jpg` - Lab report images
  - `/user-documents/{user_id}/` - Other medical documents
- Lifecycle policies: Archive images older than 1 year to Glacier
- Versioning enabled for audit trail
- Server-side encryption with KMS

**Amazon DynamoDB**

**Users Table:**
```
PK: USER#{phone_number}
Attributes:
  - user_id (UUID)
  - name
  - phone_number
  - date_of_birth
  - gender
  - blood_group
  - language_preference
  - location (lat/long)
  - emergency_contact
  - created_at
  - last_login
```

**MedicalHistory Table:**
```
PK: USER#{user_id}
SK: RECORD#{timestamp}#{category}
Attributes:
  - record_id (UUID)
  - category (disease, allergy, prescription, lab_report, consultation)
  - title
  - description
  - date
  - doctor_name
  - hospital_name
  - documents (S3 URLs)
  - created_at
```

**Prescriptions Table:**
```
PK: USER#{user_id}
SK: PRESCRIPTION#{prescription_id}
Attributes:
  - prescription_id (UUID)
  - doctor_name
  - hospital_name
  - date_prescribed
  - medicines (list of medicine objects)
  - duration_days
  - notes
  - image_url (S3)
  - status (active, completed, expired)
  - created_at
```

**Medicines (nested in Prescriptions):**
```
{
  medicine_name: string
  dosage: string
  frequency: string (e.g., "twice daily")
  timing: string (e.g., "after meals")
  duration_days: number
  quantity: number
  reminders_enabled: boolean
}
```

**Reminders Table:**
```
PK: USER#{user_id}
SK: REMINDER#{reminder_id}
Attributes:
  - reminder_id (UUID)
  - prescription_id
  - medicine_name
  - dosage
  - schedule_times (list of times)
  - start_date
  - end_date
  - adherence_log (list of taken/missed)
  - enabled (boolean)
  - created_at
GSI: RemindersByTime (for querying due reminders)
```

**Providers Table:**
```
PK: PROVIDER#{provider_id}
Attributes:
  - provider_id (UUID)
  - name
  - type (doctor, hospital, clinic, pharmacy)
  - specialty
  - location (lat/long)
  - address
  - phone_number
  - email
  - ratings
  - available_slots (for doctors)
  - services (list)
  - created_at
GSI: ProvidersByLocation (for geospatial queries)
```

**Appointments Table:**
```
PK: USER#{user_id}
SK: APPOINTMENT#{appointment_id}
Attributes:
  - appointment_id (UUID)
  - provider_id
  - provider_name
  - appointment_date
  - appointment_time
  - status (scheduled, completed, cancelled)
  - reason
  - notes
  - created_at
GSI: AppointmentsByProvider (for provider's schedule)
```

**Queries Table:**
```
PK: USER#{user_id}
SK: QUERY#{timestamp}
Attributes:
  - query_id (UUID)
  - query_text
  - query_language
  - response_text
  - tools_used (list)
  - processing_time_ms
  - created_at
```

**WearableData Table (Optional):**
```
PK: USER#{user_id}
SK: READING#{timestamp}#{metric_type}
Attributes:
  - device_id
  - metric_type (heart_rate, blood_pressure, blood_glucose, steps, sleep)
  - value
  - unit
  - timestamp
  - created_at
```

**AWS IoT Core (Optional)**
- Device registry for wearables (smartwatches, fitness trackers, glucose monitors)
- MQTT topics:
  - `health/{user_id}/devices/{device_id}/data` - Vital signs
  - `health/{user_id}/alerts` - Alert notifications
- Device shadows for wearable state management
- Rules engine to route messages to Lambda

**EventBridge**
- Scheduled rules for medicine reminders
- Rule pattern: cron expressions for each reminder time
- Targets: Medicine Reminder Lambda
- Dead letter queue for failed notifications

## Data Models

### Core Domain Models

```typescript
// User and Authentication
interface User {
  userId: string;
  phoneNumber: string;
  name: string;
  dateOfBirth: Date;
  gender: 'male' | 'female' | 'other';
  bloodGroup: string;
  languagePreference: Language;
  location: GeoLocation;
  emergencyContact: EmergencyContact;
  createdAt: Date;
  lastLogin: Date;
}

interface EmergencyContact {
  name: string;
  relationship: string;
  phoneNumber: string;
}

interface AuthToken {
  accessToken: string;
  refreshToken: string;
  expiresIn: number;
}

// Medical History
interface MedicalRecord {
  recordId: string;
  userId: string;
  category: 'disease' | 'allergy' | 'prescription' | 'lab_report' | 'consultation';
  title: string;
  description: string;
  date: Date;
  doctorName?: string;
  hospitalName?: string;
  documents: string[]; // S3 URLs
  createdAt: Date;
}

interface MedicalHistory {
  diseases: Disease[];
  allergies: Allergy[];
  prescriptions: Prescription[];
  labReports: LabReport[];
  consultations: Consultation[];
}

interface Disease {
  name: string;
  diagnosedDate: Date;
  status: 'active' | 'resolved' | 'chronic';
  notes: string;
}

interface Allergy {
  allergen: string;
  type: 'drug' | 'food' | 'environmental';
  severity: 'mild' | 'moderate' | 'severe';
  reaction: string;
  diagnosedDate: Date;
}

// Prescriptions and Medicines
interface Prescription {
  prescriptionId: string;
  userId: string;
  doctorName: string;
  hospitalName: string;
  datePrescribed: Date;
  medicines: Medicine[];
  durationDays: number;
  notes: string;
  imageUrl: string; // S3 URL
  status: 'active' | 'completed' | 'expired';
  createdAt: Date;
}

interface Medicine {
  medicineName: string;
  dosage: string;
  frequency: string; // "once daily", "twice daily", "thrice daily"
  timing: string; // "before meals", "after meals", "bedtime"
  durationDays: number;
  quantity: number;
  remindersEnabled: boolean;
}

interface PrescriptionData {
  doctorName: string;
  hospitalName: string;
  date: Date;
  medicines: Medicine[];
  confidence: number; // OCR confidence
}

// Lab Reports
interface LabReport {
  reportId: string;
  userId: string;
  reportType: string;
  testDate: Date;
  labName: string;
  tests: LabTest[];
  imageUrl: string; // S3 URL
  createdAt: Date;
}

interface LabTest {
  testName: string;
  value: string;
  unit: string;
  referenceRange: string;
  status: 'normal' | 'abnormal' | 'critical';
}

interface LabReportData {
  reportType: string;
  testDate: Date;
  labName: string;
  tests: LabTest[];
  confidence: number; // OCR confidence
}

// Reminders
interface Reminder {
  reminderId: string;
  userId: string;
  prescriptionId: string;
  medicineName: string;
  dosage: string;
  scheduleTimes: string[]; // ["08:00", "20:00"]
  startDate: Date;
  endDate: Date;
  adherenceLog: DoseLog[];
  enabled: boolean;
  createdAt: Date;
}

interface DoseLog {
  scheduledTime: Date;
  takenTime?: Date;
  status: 'taken' | 'missed' | 'skipped';
}

// Symptom Checking
interface SymptomAnalysis {
  symptoms: string[];
  possibleConditions: Condition[];
  urgency: 'low' | 'medium' | 'high' | 'emergency';
  recommendations: string[];
  followUpQuestions: string[];
  disclaimer: string;
}

interface Condition {
  name: string;
  confidence: number; // 0-1
  description: string;
  commonSymptoms: string[];
  whenToSeeDoctor: string;
  homeRemedies: string[];
}

// Drug Interactions
interface InteractionAnalysis {
  newMedicine: string;
  conflicts: Conflict[];
  overallSeverity: 'none' | 'minor' | 'moderate' | 'major';
  safeToTake: boolean;
  recommendations: string[];
}

interface Conflict {
  type: 'drug-drug' | 'drug-allergy';
  conflictWith: string;
  severity: 'minor' | 'moderate' | 'major';
  description: string;
  effects: string[];
}

// Healthcare Providers
interface Provider {
  providerId: string;
  name: string;
  type: 'doctor' | 'hospital' | 'clinic' | 'pharmacy';
  specialty?: string;
  location: GeoLocation;
  address: string;
  phoneNumber: string;
  email: string;
  ratings: number; // 0-5
  reviewCount: number;
  distance: number; // km from user
  availableSlots?: TimeSlot[];
  services: string[];
  createdAt: Date;
}

interface TimeSlot {
  date: Date;
  startTime: string;
  endTime: string;
  available: boolean;
}

// Appointments
interface Appointment {
  appointmentId: string;
  userId: string;
  providerId: string;
  providerName: string;
  appointmentDate: Date;
  appointmentTime: string;
  status: 'scheduled' | 'completed' | 'cancelled' | 'no-show';
  reason: string;
  notes: string;
  createdAt: Date;
}

// Query and Response
interface Query {
  queryId: string;
  userId: string;
  queryText: string;
  queryLanguage: Language;
  queryType: 'text' | 'voice';
  timestamp: Date;
}

interface Response {
  queryId: string;
  responseText: string;
  responseLanguage: Language;
  audioUrl?: string; // For voice responses
  toolsUsed: string[];
  processingTimeMs: number;
  confidence: number;
  disclaimer: string;
}

// Health Information
interface HealthInfo {
  topic: string;
  description: string;
  symptoms: string[];
  causes: string[];
  treatments: string[];
  prevention: string[];
  whenToSeeDoctor: string;
  sources: Source[];
}

interface Source {
  title: string;
  url: string;
  organization: string;
}

// Wearable Data (Optional)
interface WearableReading {
  userId: string;
  deviceId: string;
  metricType: 'heart_rate' | 'blood_pressure' | 'blood_glucose' | 'steps' | 'sleep';
  value: number | BloodPressure;
  unit: string;
  timestamp: Date;
  status: 'normal' | 'abnormal' | 'critical';
}

interface BloodPressure {
  systolic: number;
  diastolic: number;
}

// Supporting Types
type Language = 'hi' | 'ta' | 'te' | 'kn' | 'bn' | 'mr' | 'en';

interface GeoLocation {
  latitude: number;
  longitude: number;
}

interface DateRange {
  startDate: Date;
  endDate: Date;
}
```


## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property Reflection

After analyzing all acceptance criteria, I identified several areas where properties can be consolidated:

- **Language consistency properties** (4.4): Response language should match query language
- **Response completeness properties** (2.1, 2.2, 7.2): Multiple properties verify response structure contains required fields - can be combined per domain
- **Alert generation properties** (5.2, 5.4, 5.5, 6.2, 14.3): All test alert triggering under specific conditions - can be combined into general alert property
- **Data compression properties** (2.5, 10.4): Both test compression in low-bandwidth mode - can be combined
- **Interaction checking properties** (6.1, 6.3): Both test drug interaction checking - can be combined

### Core Properties

**Property 1: Medical Record Timestamping and Categorization**

*For any* medical record added to the system, it should be automatically assigned a timestamp and categorized into one of the valid categories (disease, allergy, prescription, lab_report, consultation).

**Validates: Requirements 1.3**

**Property 2: Medical History Filtering**

*For any* medical history query with filters (date range, category, or keyword), all returned records should match the specified filter criteria.

**Validates: Requirements 1.4**

**Property 3: Prescription OCR Completeness**

*For any* prescription image successfully processed by OCR, the extracted data should include medicine names, dosages, doctor name, and hospital name, with processing time under 10 seconds.

**Validates: Requirements 2.1**

**Property 4: Lab Report OCR Completeness**

*For any* lab report image successfully processed by OCR, the extracted data should include test names, values, and reference ranges.

**Validates: Requirements 2.2**

**Property 5: OCR to Medical History Integration**

*For any* successfully completed OCR extraction (prescription or lab report), the extracted structured data should be automatically added to the patient's medical history within 5 seconds.

**Validates: Requirements 2.3**

**Property 6: OCR Quality Validation**

*For any* image that fails quality checks (too blurry, too dark, wrong format, insufficient resolution), the system should return an error response with specific guidance on capturing a better image.

**Validates: Requirements 2.4**

**Property 7: Low-Bandwidth Image Compression**

*For any* image uploaded in Low_Bandwidth_Mode, the compressed image size should be at least 60% smaller than the original and under 200KB, while still being processable by OCR with acceptable accuracy.

**Validates: Requirements 2.5, 10.4**

**Property 8: Symptom Analysis Completeness**

*For any* symptom description provided by a patient, the Symptom_Checker should return possible conditions with confidence levels, urgency assessment, and recommendations.

**Validates: Requirements 3.1**

**Property 9: Symptom Follow-Up Questions**

*For any* symptom description that is ambiguous or lacks sufficient detail, the Symptom_Checker should generate follow-up questions to narrow down potential conditions.

**Validates: Requirements 3.2**

**Property 10: Emergency Symptom Detection**

*For any* symptom description containing emergency indicators (chest pain, difficulty breathing, severe bleeding, loss of consciousness), the system should set urgency to "emergency" and recommend immediate medical attention.

**Validates: Requirements 3.3**

**Property 11: Minor Condition Home Remedies**

*For any* symptom analysis resulting in minor conditions (common cold, minor headache, mild indigestion), the response should include home remedies and self-care advice.

**Validates: Requirements 3.4**

**Property 12: Medical Disclaimer Inclusion**

*For any* symptom checking response, health question answer, or medical advice, the response should include a disclaimer stating that AI guidance does not replace professional medical diagnosis.

**Validates: Requirements 3.5**

**Property 13: Voice Response Generation**

*For any* text response and any supported Regional_Language, the Voice_Interface should successfully generate speech audio in that language with valid audio format and non-zero duration.

**Validates: Requirements 4.3**

**Property 14: Multilingual Response Consistency**

*For any* query in a Regional_Language, the response should be in the same Regional_Language, and all medical terms should be correctly translated.

**Validates: Requirements 4.4**

**Property 15: Input Mode Switching**

*For any* user session, switching between voice and text input modes should preserve conversation context and not lose previous query history.

**Validates: Requirements 4.5**

**Property 16: Automatic Reminder Creation**

*For any* prescription added to medical history containing medicines with dosage schedules, the system should automatically create Medicine_Reminders with correct timing based on the frequency and duration.

**Validates: Requirements 5.1**

**Property 17: Reminder Notification Content**

*For any* Medicine_Reminder that reaches its scheduled time, a push notification should be sent containing the medicine name, dosage, and timing instructions.

**Validates: Requirements 5.2**

**Property 18: Dose Adherence Logging**

*For any* dose marked as taken by the patient, the system should log the action in the medication adherence record with timestamp and status.

**Validates: Requirements 5.3**

**Property 19: Prescription Expiry Alerts**

*For any* active prescription where the end date is within 3 days or the medicine quantity is running low, the system should generate an alert to the patient.

**Validates: Requirements 5.4**

**Property 20: Adherence Reminder Triggering**

*For any* patient who has missed 3 or more consecutive doses of a medicine, the system should send an adherence reminder encouraging them to maintain their medication schedule.

**Validates: Requirements 5.5**

**Property 21: Drug Interaction Checking**

*For any* new medicine being added, the Drug_Interaction_Checker should compare it against both the patient's allergy list and existing prescriptions to identify potential conflicts.

**Validates: Requirements 6.1, 6.3**

**Property 22: Drug-Allergy Conflict Alerting**

*For any* detected drug-allergy or drug-drug conflict, the system should immediately alert the patient with the conflict details and severity level (minor, moderate, major).

**Validates: Requirements 6.2**

**Property 23: Dangerous Interaction Recommendations**

*For any* drug interaction with "major" severity, the system should recommend consulting a doctor before taking the medicine and clearly explain the potential risks.

**Validates: Requirements 6.4**

**Property 24: Provider Search Radius Filtering**

*For any* healthcare provider search with a specified location, all returned providers should be within 10km of that location, sorted by distance.

**Validates: Requirements 7.1**

**Property 25: Provider Information Completeness**

*For any* healthcare provider in search results, the response should include name, specialty, distance, ratings, contact information, and address.

**Validates: Requirements 7.2**

**Property 26: Provider Directions**

*For any* selected healthcare provider, the system should provide directions from the patient's current location using Amazon Location Service.

**Validates: Requirements 7.4**

**Property 27: Provider Specialty Filtering**

*For any* provider search with a specialty filter, all returned providers should match that specialty exactly.

**Validates: Requirements 7.5**

**Property 28: Appointment Slot Display**

*For any* healthcare provider selected for appointment booking, the system should display their available time slots for the next 7 days.

**Validates: Requirements 8.1**

**Property 29: Appointment Confirmation Notifications**

*For any* successfully booked appointment, confirmation notifications should be sent to both the patient and the healthcare provider within 1 minute.

**Validates: Requirements 8.2**

**Property 30: Appointment Reminder Scheduling**

*For any* scheduled appointment, the system should create two reminders: one 24 hours before and one 1 hour before the appointment time.

**Validates: Requirements 8.3**

**Property 31: Appointment Cancellation Flow**

*For any* appointment cancellation by a patient, the system should notify the healthcare provider, update the provider's availability, and mark the appointment as cancelled.

**Validates: Requirements 8.4**

**Property 32: Appointment Rescheduling**

*For any* appointment rescheduling request, the system should check provider availability for the new time slot and only allow rescheduling if the slot is available.

**Validates: Requirements 8.5**

**Property 33: Health Information Source Citations**

*For any* health question response providing medical information, the response should include source citations (research papers, medical guidelines, WHO/CDC resources).

**Validates: Requirements 9.1, 13.4**

**Property 34: Personalized Health Responses**

*For any* health question where the patient's medical history is relevant (e.g., questions about conditions they have), the response should reference their specific medical history.

**Validates: Requirements 9.2**

**Property 35: Professional Consultation Recommendations**

*For any* query that the AI_Agent cannot answer with sufficient confidence or that requires professional diagnosis, the system should recommend consulting a healthcare professional.

**Validates: Requirements 9.3, 13.3**

**Property 36: Conversation Context Preservation**

*For any* follow-up question in a conversation, the AI_Agent should have access to previous messages in the conversation and provide contextually relevant responses.

**Validates: Requirements 9.5**

**Property 37: Low-Bandwidth Data Reduction**

*For any* API response in Low_Bandwidth_Mode compared to normal mode, the payload size should be at least 60% smaller through compression, field reduction, or format optimization.

**Validates: Requirements 10.1**

**Property 38: Response Caching**

*For any* identical query made within a 1-hour window, the second request should be served from cache with response time under 500ms.

**Validates: Requirements 10.2**

**Property 39: Connectivity-Based Response Format**

*For any* query when network connectivity is poor (detected bandwidth < 100 kbps), the response should be text-only without audio generation.

**Validates: Requirements 10.3**

**Property 40: Offline Query Queueing**

*For any* query submitted while offline, it should be stored in the local queue and automatically processed when connectivity is restored, maintaining the original query order.

**Validates: Requirements 10.5**

**Property 41: Query Response Time**

*For any* text query under normal load conditions, the system should return a complete response within 3 seconds.

**Validates: Requirements 12.3**

**Property 42: Multi-Part Query Decomposition**

*For any* query containing multiple distinct questions (detected by multiple question marks or conjunctions), the AI_Agent should decompose it into sub-queries and coordinate responses from multiple tools.

**Validates: Requirements 13.1**

**Property 43: Tool Orchestration Sequence**

*For any* query requiring multiple tools (e.g., medical history + symptom checking), the tools should be called in logical dependency order to build a complete response.

**Validates: Requirements 13.5**

**Property 44: Wearable Device Registration**

*For any* new wearable device attempting to connect to AWS IoT Core, the system should successfully register the device, assign a unique device ID, and authenticate it before accepting data.

**Validates: Requirements 14.1**

**Property 45: Wearable Data Storage**

*For any* vital signs reading received from a wearable device, the data should be validated and stored in DynamoDB with timestamp, metric type, value, and user association.

**Validates: Requirements 14.2**

**Property 46: Abnormal Vital Signs Alerting**

*For any* wearable reading where vital signs exceed normal ranges (heart rate > 100 bpm at rest, blood pressure > 140/90, blood glucose > 200 mg/dL), the system should alert the patient immediately.

**Validates: Requirements 14.3**

**Property 47: Wearable Data Integration in Health Assessments**

*For any* health assessment or recommendation where wearable data is available, the AI_Agent should incorporate recent vital signs trends into its analysis.

**Validates: Requirements 14.4**

**Property 48: Health Trend Report Generation**

*For any* patient with at least 7 days of wearable data, the system should be able to generate a health trend report showing patterns in vital signs over time.

**Validates: Requirements 14.5**

## Error Handling

### Error Categories and Handling Strategies

**1. User Input Errors**
- Invalid image format or corrupted file
- Unsupported language selection
- Malformed query text
- Voice audio too short or inaudible
- Missing required fields in medical records

**Handling:**
- Return 400 Bad Request with specific error message in user's language
- Provide guidance on correct input format
- Log error for monitoring but don't retry automatically
- Suggest alternative input methods (e.g., text instead of voice)

**2. AI Service Errors**
- Textract OCR failure or low confidence
- Bedrock agent timeout or rate limiting
- Transcribe/Translate service unavailable
- Polly voice synthesis failure
- Location Service API errors

**Handling:**
- Implement exponential backoff retry (3 attempts)
- Fall back to cached responses if available
- Return 503 Service Unavailable with retry-after header
- Alert operations team for sustained failures
- Provide degraded functionality (e.g., skip voice if Polly fails)

**3. Data Layer Errors**
- DynamoDB throttling or unavailability
- S3 upload/download failures
- IoT Core connection drops
- EventBridge rule execution failures

**Handling:**
- Implement circuit breaker pattern
- Queue writes for later retry using SQS
- Return cached data for reads when possible
- Degrade gracefully (e.g., skip wearable data if IoT Core unavailable)
- Use DynamoDB auto-scaling to prevent throttling

**4. Medical Safety Errors**
- Critical drug interaction detected
- Emergency symptoms identified
- Abnormal vital signs from wearables
- Prescription dosage exceeds safe limits

**Handling:**
- Immediately alert patient with high-priority notification
- Recommend urgent medical consultation
- Log all safety alerts for audit trail
- Escalate to emergency contact if configured
- Never suppress or downplay safety warnings

**5. Authentication and Authorization Errors**
- Invalid or expired JWT token
- OTP verification failure
- Insufficient permissions
- Account locked due to suspicious activity

**Handling:**
- Return 401 Unauthorized or 403 Forbidden
- Clear client-side auth state
- Prompt user to re-authenticate
- Log suspicious patterns for security monitoring
- Implement rate limiting on auth endpoints

### Error Response Format

```typescript
interface ErrorResponse {
  error: {
    code: string; // Machine-readable error code
    message: string; // Human-readable message in user's language
    details?: any; // Additional context
    retryable: boolean; // Whether client should retry
    retryAfter?: number; // Seconds to wait before retry
    severity?: 'low' | 'medium' | 'high' | 'critical'; // For medical errors
  };
  requestId: string; // For support and debugging
  timestamp: Date;
}
```

### Monitoring and Alerting

- CloudWatch alarms for error rate thresholds (>5% error rate)
- X-Ray tracing for distributed request tracking
- Custom metrics for AI service latency and accuracy
- SNS notifications for critical errors and medical safety alerts
- Dashboard showing real-time system health
- PagerDuty integration for on-call escalation
- Weekly error trend reports

## Testing Strategy

### Dual Testing Approach

SwasthyaMitra requires both unit tests and property-based tests for comprehensive coverage:

**Unit Tests** focus on:
- Specific examples of OCR extraction with known prescriptions
- Edge cases (empty queries, malformed data, boundary conditions)
- Error conditions (service failures, invalid inputs)
- Integration points between components
- Authentication flows and security controls
- Medical safety scenarios (drug interactions, emergency symptoms)

**Property-Based Tests** focus on:
- Universal properties that hold for all inputs (see Correctness Properties above)
- Comprehensive input coverage through randomization
- Language consistency across all supported languages
- Data integrity across all CRUD operations
- Performance characteristics under varied load

### Property-Based Testing Configuration

**Framework:** Hypothesis (for Python Lambda functions)

**Configuration:**
- Minimum 100 iterations per property test
- Seed-based reproducibility for failed tests
- Shrinking enabled to find minimal failing examples
- Timeout: 30 seconds per property
- Health check mode for critical properties

**Test Tagging Format:**
```python
# Feature: swasthyamitra-healthcare-assistant, Property 3: Prescription OCR Completeness
@given(prescription_image=prescription_images())
def test_prescription_ocr_completeness(prescription_image):
    result = ocr_processor.extract_prescription(prescription_image)
    assert result.processing_time < 10.0
    assert result.medicine_names is not None
    assert len(result.medicine_names) > 0
    assert result.doctor_name is not None
    assert result.hospital_name is not None
```

### Test Data Generators

Property-based tests require generators for domain objects:

```python
from hypothesis import strategies as st

# Generator for prescription images with various characteristics
@st.composite
def prescription_images(draw):
    return {
        'data': draw(st.binary(min_size=1000, max_size=500000)),
        'format': draw(st.sampled_from(['jpg', 'png', 'pdf'])),
        'quality': draw(st.sampled_from(['high', 'medium', 'low'])),
        'handwritten': draw(st.booleans())
    }

# Generator for user queries in different languages
@st.composite
def health_queries(draw):
    return {
        'text': draw(st.text(min_size=5, max_size=200)),
        'language': draw(st.sampled_from(['hi', 'ta', 'te', 'kn', 'bn', 'mr'])),
        'query_type': draw(st.sampled_from(['symptom', 'history', 'provider', 'general']))
    }

# Generator for symptom descriptions
@st.composite
def symptom_descriptions(draw):
    symptoms = ['fever', 'cough', 'headache', 'nausea', 'chest pain', 'fatigue']
    return {
        'symptoms': draw(st.lists(st.sampled_from(symptoms), min_size=1, max_size=5)),
        'duration': draw(st.sampled_from(['1 day', '3 days', '1 week', '2 weeks'])),
        'severity': draw(st.sampled_from(['mild', 'moderate', 'severe']))
    }

# Generator for medicine names
@st.composite
def medicines(draw):
    return {
        'name': draw(st.sampled_from(['Paracetamol', 'Ibuprofen', 'Amoxicillin', 'Metformin'])),
        'dosage': draw(st.sampled_from(['500mg', '250mg', '1000mg'])),
        'frequency': draw(st.sampled_from(['once daily', 'twice daily', 'thrice daily'])),
        'duration_days': draw(st.integers(min_value=1, max_value=30))
    }

# Generator for wearable readings
@st.composite
def wearable_readings(draw):
    metric_type = draw(st.sampled_from(['heart_rate', 'blood_pressure', 'blood_glucose', 'steps']))
    if metric_type == 'heart_rate':
        value = draw(st.integers(min_value=40, max_value=200))
    elif metric_type == 'blood_pressure':
        value = {'systolic': draw(st.integers(min_value=80, max_value=200)),
                 'diastolic': draw(st.integers(min_value=50, max_value=130))}
    elif metric_type == 'blood_glucose':
        value = draw(st.integers(min_value=50, max_value=400))
    else:  # steps
        value = draw(st.integers(min_value=0, max_value=50000))
    
    return {
        'metric_type': metric_type,
        'value': value,
        'timestamp': draw(st.datetimes())
    }
```

### Integration Testing

**API Integration Tests:**
- Test complete request/response flows through API Gateway
- Verify authentication and authorization with Cognito
- Test rate limiting and throttling
- Validate error responses
- Test CORS configuration

**AI Service Integration Tests:**
- Test Bedrock agent with real tool calls
- Verify Textract OCR accuracy on test dataset
- Test Polly/Translate/Transcribe with sample inputs
- Measure latency and accuracy
- Test Location Service provider search

**IoT Integration Tests (Optional):**
- Simulate wearable connections to IoT Core
- Test MQTT message routing
- Verify device shadow updates
- Test alert triggering from vital signs data

### Performance Testing

**Load Testing:**
- Simulate 100,000 concurrent users using Artillery or Locust
- Test auto-scaling behavior under load
- Measure response times at different percentiles (p50, p95, p99)
- Identify bottlenecks and resource constraints
- Test Lambda cold start impact

**Stress Testing:**
- Push system beyond normal capacity
- Identify breaking points
- Verify graceful degradation
- Test recovery after overload
- Validate circuit breaker patterns

### Security Testing

**Penetration Testing:**
- Test authentication bypass attempts
- API abuse and rate limit bypass
- Unauthorized data access attempts
- HIPAA compliance validation
- PHI (Protected Health Information) data leakage tests

**Compliance Testing:**
- Verify data encryption at rest and in transit
- Test data deletion workflows (GDPR/right to be forgotten)
- Validate IAM policies and least privilege
- Audit logging completeness
- Test MFA enforcement

### Testing in Low-Bandwidth Conditions

**Network Simulation:**
- Use network throttling tools to simulate 2G/3G speeds
- Test with high latency (500ms+) and packet loss (5-10%)
- Verify compression and caching effectiveness
- Test offline queue functionality
- Measure user experience degradation

### Continuous Testing

**CI/CD Pipeline:**
1. Unit tests run on every commit
2. Property-based tests run on every PR
3. Integration tests run on merge to main
4. Performance tests run nightly
5. Security scans run weekly
6. Full regression suite before production deployment

**Test Coverage Goals:**
- Unit test coverage: >80%
- Property test coverage: All 48 properties implemented
- Integration test coverage: All API endpoints
- Critical path coverage: 100% (auth, OCR, symptom checking, drug interactions)

### Manual Testing

**User Acceptance Testing:**
- Test with real patients in field conditions
- Verify language translations with native speakers
- Test voice interface with regional accents
- Validate OCR accuracy with real prescriptions and lab reports
- Test with medical professionals for accuracy

**Accessibility Testing:**
- Test with screen readers for visually impaired users
- Verify voice interface for illiterate users
- Test with low-end devices (2GB RAM, older Android versions)
- Validate UI in bright sunlight conditions
- Test with elderly users for usability

## Implementation Notes

### Hackathon Scope Considerations

For a 36-hour hackathon with a 4-person team, prioritize:

**Must-Have (MVP):**
1. Basic mobile UI with image upload and text query
2. OCR using Textract for prescriptions
3. Bedrock agent with 3 core tools (history, symptom checker, drug interaction)
4. DynamoDB for user and medical history storage
5. API Gateway + Lambda backend (single orchestrator pattern)
6. Authentication with Cognito (phone number + OTP)
7. Support for 2-3 regional languages (Hindi, Tamil, English)
8. Medicine reminder creation (manual scheduling for demo)

**Should-Have (if time permits):**
1. Voice interface (Transcribe + Polly)
2. Provider search using Location Service
3. Appointment booking (basic flow)
4. Low-bandwidth optimization
5. Caching layer

**Nice-to-Have (post-hackathon):**
1. Full 6-language support
2. Wearable device integration
3. Automated EventBridge reminders
4. Advanced AI agent reasoning
5. Comprehensive property-based tests
6. Health trend reports

### Development Workflow

**Day 1 (Hours 0-12):**
- Set up AWS infrastructure (CDK or Terraform)
- Create basic mobile app shell (React Native)
- Implement authentication flow (Cognito)
- Set up Bedrock agent with one tool
- Create DynamoDB tables
- Implement basic API Gateway endpoints

**Day 1 (Hours 12-24):**
- Implement image upload and OCR (Textract)
- Add medical history storage and retrieval
- Create symptom checker tool for Bedrock agent
- Add drug interaction checker tool
- Build basic UI for query and response
- Test end-to-end flow

**Day 2 (Hours 24-36):**
- Add multilingual support (Translate)
- Implement voice interface (if time)
- Add provider search (Location Service)
- Create medicine reminder UI
- Polish UI and error handling
- Prepare demo and presentation
- Deploy to production
- Create demo video

### Project Folder Structure

```
swasthyamitra/
├── frontend/
│   ├── mobile/                 # React Native app
│   │   ├── src/
│   │   │   ├── screens/
│   │   │   │   ├── AuthScreen.tsx
│   │   │   │   ├── HomeScreen.tsx
│   │   │   │   ├── ChatScreen.tsx
│   │   │   │   ├── HistoryScreen.tsx
│   │   │   │   ├── OCRScreen.tsx
│   │   │   │   ├── RemindersScreen.tsx
│   │   │   │   └── ProvidersScreen.tsx
│   │   │   ├── components/
│   │   │   ├── services/
│   │   │   │   └── api.ts
│   │   │   ├── utils/
│   │   │   └── App.tsx
│   │   └── package.json
│   └── web/                    # React PWA
│       └── src/
├── backend/
│   ├── lambda/
│   │   ├── orchestrator/       # Main agent handler
│   │   │   ├── handler.py
│   │   │   ├── agent.py
│   │   │   ├── tools/
│   │   │   │   ├── medical_history.py
│   │   │   │   ├── symptom_checker.py
│   │   │   │   ├── drug_interaction.py
│   │   │   │   ├── provider_search.py
│   │   │   │   └── health_info.py
│   │   │   └── requirements.txt
│   │   ├── ocr_processor/      # Textract OCR
│   │   │   ├── handler.py
│   │   │   ├── prescription_parser.py
│   │   │   ├── labreport_parser.py
│   │   │   └── requirements.txt
│   │   ├── reminder/           # Medicine reminders
│   │   │   ├── handler.py
│   │   │   └── requirements.txt
│   │   └── wearable/           # IoT data processor (optional)
│   │       ├── handler.py
│   │       └── requirements.txt
│   ├── layers/                 # Lambda layers for shared code
│   │   └── common/
│   │       ├── python/
│   │       │   └── utils.py
│   │       └── requirements.txt
│   └── tests/
│       ├── unit/
│       └── integration/
├── infrastructure/
│   ├── cdk/                    # AWS CDK (TypeScript)
│   │   ├── lib/
│   │   │   ├── api-stack.ts
│   │   │   ├── auth-stack.ts
│   │   │   ├── data-stack.ts
│   │   │   ├── lambda-stack.ts
│   │   │   └── monitoring-stack.ts
│   │   ├── bin/
│   │   │   └── app.ts
│   │   └── package.json
│   └── terraform/              # Alternative: Terraform
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
├── docs/
│   ├── architecture.md
│   ├── api-spec.yaml           # OpenAPI specification
│   ├── deployment.md
│   └── demo-script.md
├── scripts/
│   ├── deploy.sh
│   ├── seed-data.py            # Seed test data
│   └── test-e2e.sh
└── README.md
```

### Backend Starter Code

**Lambda Orchestrator Handler (handler.py):**
```python
import json
import boto3
import os
from agent import BedrockAgent
from tools import medical_history, symptom_checker, drug_interaction, provider_search

# Initialize AWS clients
bedrock = boto3.client('bedrock-agent-runtime')
dynamodb = boto3.resource('dynamodb')
translate = boto3.client('translate')

# Initialize agent
agent = BedrockAgent(
    model_id='anthropic.claude-3-sonnet-20240229-v1:0',
    tools=[
        medical_history.get_tool(),
        symptom_checker.get_tool(),
        drug_interaction.get_tool(),
        provider_search.get_tool()
    ]
)

def lambda_handler(event, context):
    """
    Main orchestrator Lambda function.
    Handles all user queries and routes to Bedrock agent.
    """
    try:
        # Parse request
        body = json.loads(event['body'])
        user_id = event['requestContext']['authorizer']['claims']['sub']
        query_text = body['query']
        language = body.get('language', 'en')
        
        # Translate query to English if needed
        if language != 'en':
            query_text = translate_text(query_text, language, 'en')
        
        # Get user context (medical history summary)
        user_context = get_user_context(user_id)
        
        # Invoke Bedrock agent
        response = agent.invoke(
            query=query_text,
            user_context=user_context,
            user_id=user_id
        )
        
        # Translate response back to user's language
        if language != 'en':
            response['text'] = translate_text(response['text'], 'en', language)
        
        # Log query and response
        log_query(user_id, query_text, response, language)
        
        return {
            'statusCode': 200,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*'
            },
            'body': json.dumps(response)
        }
        
    except Exception as e:
        print(f"Error: {str(e)}")
        return {
            'statusCode': 500,
            'body': json.dumps({'error': str(e)})
        }

def translate_text(text, source_lang, target_lang):
    """Translate text using Amazon Translate."""
    response = translate.translate_text(
        Text=text,
        SourceLanguageCode=source_lang,
        TargetLanguageCode=target_lang
    )
    return response['TranslatedText']

def get_user_context(user_id):
    """Get user's medical history summary for context."""
    table = dynamodb.Table(os.environ['MEDICAL_HISTORY_TABLE'])
    response = table.query(
        KeyConditionExpression='PK = :pk',
        ExpressionAttributeValues={':pk': f'USER#{user_id}'},
        Limit=10,
        ScanIndexForward=False
    )
    
    # Summarize recent medical history
    context = {
        'diseases': [],
        'allergies': [],
        'current_medications': []
    }
    
    for item in response['Items']:
        if item['category'] == 'disease':
            context['diseases'].append(item['title'])
        elif item['category'] == 'allergy':
            context['allergies'].append(item['title'])
        elif item['category'] == 'prescription' and item.get('status') == 'active':
            context['current_medications'].extend([m['medicine_name'] for m in item.get('medicines', [])])
    
    return context

def log_query(user_id, query, response, language):
    """Log query and response to DynamoDB."""
    table = dynamodb.Table(os.environ['QUERIES_TABLE'])
    table.put_item(Item={
        'PK': f'USER#{user_id}',
        'SK': f'QUERY#{int(time.time() * 1000)}',
        'query_text': query,
        'query_language': language,
        'response_text': response['text'],
        'tools_used': response.get('tools_used', []),
        'processing_time_ms': response.get('processing_time_ms', 0),
        'created_at': datetime.utcnow().isoformat()
    })
```

**Agent Logic (agent.py):**
```python
import boto3
import json

class BedrockAgent:
    def __init__(self, model_id, tools):
        self.model_id = model_id
        self.tools = {tool['name']: tool for tool in tools}
        self.bedrock = boto3.client('bedrock-runtime')
    
    def invoke(self, query, user_context, user_id):
        """
        Invoke Bedrock agent with query and context.
        Implements tool calling and response synthesis.
        """
        # Build system prompt with medical context
        system_prompt = self._build_system_prompt(user_context)
        
        # Build tool definitions for Claude
        tool_definitions = [self._format_tool_for_claude(t) for t in self.tools.values()]
        
        # Initial query to Claude
        messages = [{'role': 'user', 'content': query}]
        
        max_iterations = 5
        tools_used = []
        
        for iteration in range(max_iterations):
            # Call Claude
            response = self._call_claude(system_prompt, messages, tool_definitions)
            
            # Check if Claude wants to use a tool
            if response.get('stop_reason') == 'tool_use':
                tool_use = response['content'][-1]
                tool_name = tool_use['name']
                tool_input = tool_use['input']
                
                # Execute tool
                tool_result = self._execute_tool(tool_name, tool_input, user_id)
                tools_used.append(tool_name)
                
                # Add tool result to conversation
                messages.append({'role': 'assistant', 'content': response['content']})
                messages.append({
                    'role': 'user',
                    'content': [{
                        'type': 'tool_result',
                        'tool_use_id': tool_use['id'],
                        'content': json.dumps(tool_result)
                    }]
                })
            else:
                # Claude has final answer
                final_text = response['content'][0]['text']
                
                # Add medical disclaimer
                final_text += "\n\n⚠️ Disclaimer: This AI-generated information is for educational purposes only and does not replace professional medical advice. Please consult a healthcare provider for diagnosis and treatment."
                
                return {
                    'text': final_text,
                    'tools_used': tools_used,
                    'processing_time_ms': response.get('usage', {}).get('total_tokens', 0)
                }
        
        return {
            'text': "I apologize, but I need more information to answer your question. Could you please provide more details?",
            'tools_used': tools_used
        }
    
    def _build_system_prompt(self, user_context):
        """Build system prompt with user's medical context."""
        prompt = """You are SwasthyaMitra, a helpful AI healthcare assistant for Indian patients.

Your role is to:
- Answer health questions with evidence-based information
- Help patients understand their symptoms
- Check for drug interactions
- Find nearby healthcare providers
- Provide health education and preventive care advice

User's Medical Context:
"""
        if user_context.get('diseases'):
            prompt += f"- Known conditions: {', '.join(user_context['diseases'])}\n"
        if user_context.get('allergies'):
            prompt += f"- Allergies: {', '.join(user_context['allergies'])}\n"
        if user_context.get('current_medications'):
            prompt += f"- Current medications: {', '.join(user_context['current_medications'])}\n"
        
        prompt += """
Important guidelines:
- Always prioritize patient safety
- Recommend immediate medical attention for emergency symptoms
- Include medical disclaimers
- Cite sources for medical information
- Be culturally sensitive to Indian healthcare context
- Use simple language that patients can understand
"""
        return prompt
    
    def _call_claude(self, system_prompt, messages, tools):
        """Call Claude via Bedrock."""
        response = self.bedrock.converse(
            modelId=self.model_id,
            messages=messages,
            system=[{'text': system_prompt}],
            toolConfig={'tools': tools}
        )
        return response
    
    def _execute_tool(self, tool_name, tool_input, user_id):
        """Execute a tool and return results."""
        tool = self.tools.get(tool_name)
        if not tool:
            return {'error': f'Tool {tool_name} not found'}
        
        # Add user_id to tool input
        tool_input['user_id'] = user_id
        
        # Execute tool function
        return tool['function'](**tool_input)
    
    def _format_tool_for_claude(self, tool):
        """Format tool definition for Claude's tool use format."""
        return {
            'toolSpec': {
                'name': tool['name'],
                'description': tool['description'],
                'inputSchema': {
                    'json': tool['input_schema']
                }
            }
        }
```

**Example Tool: Medical History (tools/medical_history.py):**
```python
import boto3
import os

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table(os.environ['MEDICAL_HISTORY_TABLE'])

def get_medical_history(user_id, category=None, date_range=None):
    """
    Retrieve patient's medical history.
    """
    query_params = {
        'KeyConditionExpression': 'PK = :pk',
        'ExpressionAttributeValues': {':pk': f'USER#{user_id}'}
    }
    
    # Add category filter if specified
    if category:
        query_params['FilterExpression'] = 'category = :cat'
        query_params['ExpressionAttributeValues'][':cat'] = category
    
    response = table.query(**query_params)
    
    return {
        'records': response['Items'],
        'count': len(response['Items'])
    }

def get_tool():
    """Return tool definition for Bedrock agent."""
    return {
        'name': 'get_medical_history',
        'description': 'Retrieves the patient\'s medical history including diseases, allergies, prescriptions, and lab reports.',
        'function': get_medical_history,
        'input_schema': {
            'type': 'object',
            'properties': {
                'category': {
                    'type': 'string',
                    'enum': ['disease', 'allergy', 'prescription', 'lab_report', 'consultation'],
                    'description': 'Filter by record category'
                },
                'date_range': {
                    'type': 'object',
                    'properties': {
                        'start_date': {'type': 'string', 'format': 'date'},
                        'end_date': {'type': 'string', 'format': 'date'}
                    },
                    'description': 'Filter by date range'
                }
            }
        }
    }
```

**Example Tool: Symptom Checker (tools/symptom_checker.py):**
```python
def check_symptoms(symptoms, duration, severity, user_id=None):
    """
    Analyze symptoms and provide preliminary health guidance.
    """
    # Emergency symptoms that require immediate attention
    emergency_symptoms = [
        'chest pain', 'difficulty breathing', 'severe bleeding',
        'loss of consciousness', 'severe head injury', 'stroke symptoms'
    ]
    
    # Check for emergency symptoms
    is_emergency = any(emerg in ' '.join(symptoms).lower() for emerg in emergency_symptoms)
    
    if is_emergency:
        return {
            'urgency': 'emergency',
            'message': '🚨 EMERGENCY: Please seek immediate medical attention. Call emergency services or go to the nearest hospital.',
            'possible_conditions': [],
            'recommendations': ['Call emergency services immediately', 'Do not delay seeking medical care']
        }
    
    # For non-emergency symptoms, provide general guidance
    # In production, this would use a medical knowledge base or ML model
    possible_conditions = analyze_symptoms_ml(symptoms, duration, severity)
    
    return {
        'urgency': determine_urgency(symptoms, severity),
        'possible_conditions': possible_conditions,
        'recommendations': generate_recommendations(possible_conditions, severity),
        'follow_up_questions': generate_follow_up_questions(symptoms)
    }

def analyze_symptoms_ml(symptoms, duration, severity):
    """
    Use ML model or knowledge base to analyze symptoms.
    Placeholder for actual implementation.
    """
    # This would integrate with a medical knowledge base or ML model
    return [
        {
            'name': 'Common Cold',
            'confidence': 0.7,
            'description': 'Viral infection of the upper respiratory tract',
            'common_symptoms': ['runny nose', 'cough', 'sore throat', 'mild fever'],
            'when_to_see_doctor': 'If symptoms persist beyond 10 days or worsen',
            'home_remedies': ['Rest', 'Stay hydrated', 'Warm liquids', 'Honey for cough']
        }
    ]

def get_tool():
    """Return tool definition for Bedrock agent."""
    return {
        'name': 'check_symptoms',
        'description': 'Analyzes patient symptoms and provides preliminary health guidance with possible conditions and urgency assessment.',
        'function': check_symptoms,
        'input_schema': {
            'type': 'object',
            'properties': {
                'symptoms': {
                    'type': 'array',
                    'items': {'type': 'string'},
                    'description': 'List of symptoms the patient is experiencing'
                },
                'duration': {
                    'type': 'string',
                    'description': 'How long the symptoms have persisted (e.g., "2 days", "1 week")'
                },
                'severity': {
                    'type': 'string',
                    'enum': ['mild', 'moderate', 'severe'],
                    'description': 'Severity of symptoms'
                }
            },
            'required': ['symptoms', 'duration', 'severity']
        }
    }
```

## AWS Service Mapping

| Feature | AWS Service | Purpose |
|---------|-------------|---------|
| User Authentication | Amazon Cognito | Phone number + OTP authentication, JWT tokens |
| API Endpoints | Amazon API Gateway | RESTful API, request validation, throttling |
| Backend Logic | AWS Lambda | Serverless compute for all business logic |
| AI Agent | Amazon Bedrock | Claude 3 Sonnet for reasoning and tool orchestration |
| OCR | Amazon Textract | Extract text from prescriptions and lab reports |
| Voice Input | Amazon Transcribe | Speech-to-text in regional languages |
| Voice Output | Amazon Polly | Text-to-speech in regional languages |
| Translation | Amazon Translate | Multilingual support for regional languages |
| Provider Search | Amazon Location Service | Geocoding and nearby provider search |
| Medical History Storage | Amazon DynamoDB | NoSQL database for user data and medical records |
| Image Storage | Amazon S3 | Store prescription and lab report images |
| Medicine Reminders | Amazon EventBridge | Scheduled rules for reminder notifications |
| Push Notifications | Amazon SNS | Send reminders and alerts to users |
| Wearable Integration | AWS IoT Core | Connect and manage wearable devices (optional) |
| Monitoring | Amazon CloudWatch | Logs, metrics, alarms, dashboards |
| Tracing | AWS X-Ray | Distributed tracing for debugging |
| Secrets Management | AWS Secrets Manager | Store API keys and credentials |
| Infrastructure | AWS CDK/CloudFormation | Infrastructure as code |

## Data Flow

### User Query Flow

```
1. User opens mobile app
   ↓
2. User types/speaks health question in Hindi
   ↓
3. Frontend sends request to API Gateway
   ↓
4. API Gateway validates JWT token with Cognito
   ↓
5. API Gateway invokes Orchestrator Lambda
   ↓
6. Lambda detects language (Hindi) and translates to English
   ↓
7. Lambda retrieves user's medical history from DynamoDB
   ↓
8. Lambda invokes Bedrock agent with query + context
   ↓
9. Bedrock agent analyzes query and decides to use symptom_checker tool
   ↓
10. Lambda executes symptom_checker tool
    ↓
11. Tool returns possible conditions and recommendations
    ↓
12. Bedrock agent synthesizes response with medical disclaimer
    ↓
13. Lambda translates response back to Hindi
    ↓
14. Lambda logs query and response to DynamoDB
    ↓
15. Lambda returns response to API Gateway
    ↓
16. API Gateway returns response to frontend
    ↓
17. Frontend displays response to user
    ↓
18. (Optional) Frontend converts text to speech using Polly
```

### Prescription Upload Flow

```
1. User captures prescription photo with camera
   ↓
2. Frontend compresses image (if low-bandwidth mode)
   ↓
3. Frontend uploads image to S3 via presigned URL
   ↓
4. S3 triggers OCR Processor Lambda
   ↓
5. Lambda downloads image from S3
   ↓
6. Lambda invokes Textract to extract text
   ↓
7. Lambda parses extracted text to identify:
   - Doctor name
   - Hospital name
   - Medicine names
   - Dosages
   - Frequency
   - Duration
   ↓
8. Lambda stores structured prescription data in DynamoDB
   ↓
9. Lambda creates medicine reminders in DynamoDB
   ↓
10. Lambda creates EventBridge rules for each reminder time
    ↓
11. Lambda returns success response
    ↓
12. Frontend displays extracted prescription details
    ↓
13. User confirms or edits details
```

### Medicine Reminder Flow

```
1. EventBridge rule triggers at scheduled time (e.g., 8:00 AM)
   ↓
2. EventBridge invokes Reminder Lambda
   ↓
3. Lambda queries DynamoDB for reminders due now
   ↓
4. For each reminder:
   a. Lambda retrieves user's device token
   b. Lambda sends push notification via SNS
   c. Notification contains: medicine name, dosage, timing
   ↓
5. User receives notification on mobile device
   ↓
6. User taps "Mark as Taken"
   ↓
7. Frontend calls API to log dose adherence
   ↓
8. Lambda updates adherence log in DynamoDB
```

### Wearable Data Flow (Optional)

```
1. Wearable device (smartwatch) measures heart rate
   ↓
2. Device publishes reading to IoT Core via MQTT
   Topic: health/{user_id}/devices/{device_id}/data
   ↓
3. IoT Core rule routes message to Wearable Lambda
   ↓
4. Lambda validates and stores reading in DynamoDB
   ↓
5. Lambda checks if reading is abnormal (heart rate > 100 bpm at rest)
   ↓
6. If abnormal:
   a. Lambda sends alert via SNS
   b. User receives push notification
   c. Alert includes: metric, value, recommendation
   ↓
7. User can view health trends in app
```

## Scalability Approach

### Auto-Scaling Strategy

1. **Lambda Concurrency**
   - Reserved concurrency for critical functions (orchestrator, OCR)
   - Provisioned concurrency to reduce cold starts
   - Auto-scales to handle traffic spikes

2. **DynamoDB Capacity**
   - On-demand pricing for unpredictable traffic
   - Auto-scaling for read/write capacity
   - Global tables for multi-region deployment

3. **API Gateway**
   - Built-in auto-scaling
   - Rate limiting per user (1000 req/min)
   - Throttling to protect backend

4. **S3**
   - Unlimited storage capacity
   - CloudFront CDN for static assets
   - Transfer acceleration for uploads

5. **Bedrock**
   - Managed service with auto-scaling
   - Request quotas can be increased
   - Fallback to cached responses if quota exceeded

### Performance Optimization

1. **Caching**
   - API Gateway caching for frequent queries
   - DynamoDB DAX for read-heavy workloads
   - CloudFront for static content
   - Application-level caching in Lambda

2. **Database Optimization**
   - DynamoDB GSIs for efficient queries
   - Composite sort keys for flexible filtering
   - Batch operations for bulk writes
   - Connection pooling

3. **Lambda Optimization**
   - Increase memory for faster execution
   - Use Lambda layers for shared code
   - Minimize cold starts with provisioned concurrency
   - Async processing for non-critical tasks

4. **Image Optimization**
   - Compress images before upload
   - Use WebP format for better compression
   - Lazy loading in frontend
   - Thumbnail generation for previews

## Cost Optimization Strategy

### Estimated Monthly Costs (10,000 active users)

| Service | Usage | Cost |
|---------|-------|------|
| Lambda | 5M requests, 512MB, 3s avg | $40 |
| API Gateway | 5M requests | $17.50 |
| DynamoDB | 10GB storage, on-demand | $30 |
| S3 | 50GB storage, 200GB transfer | $15 |
| Textract | 50,000 pages | $75 |
| Bedrock | Claude 3 Sonnet, 5M tokens | $75 |
| Polly | 2M characters | $8 |
| Translate | 5M characters | $75 |
| Transcribe | 10,000 minutes | $24 |
| Location Service | 100,000 requests | $40 |
| IoT Core | 1,000 devices (optional) | $8 |
| EventBridge | 1M events | $1 |
| SNS | 500,000 notifications | $0.50 |
| CloudWatch | Logs and metrics | $10 |
| **Total** | | **~$419/month** |

### Cost Optimization Tactics

1. **Reduce AI Service Costs**
   - Cache common queries and responses
   - Use smaller Bedrock models for simple queries
   - Batch Translate requests
   - Compress Polly audio files

2. **Optimize Lambda**
   - Right-size memory allocation
   - Use ARM architecture (Graviton2) for 20% cost savings
   - Minimize execution time
   - Use Lambda layers to reduce deployment size

3. **DynamoDB Optimization**
   - Use on-demand for unpredictable traffic
   - Archive old data to S3 Glacier
   - Use DynamoDB Streams instead of polling
   - Optimize item sizes

4. **S3 Cost Reduction**
   - Lifecycle policies to move old images to Glacier
   - Delete temporary files after processing
   - Use S3 Intelligent-Tiering
   - Compress images before storage

5. **Free Tier Usage**
   - Lambda: 1M free requests/month
   - DynamoDB: 25GB free storage
   - S3: 5GB free storage
   - CloudWatch: 10 custom metrics free

6. **Monitoring and Alerts**
   - Set up billing alarms
   - Monitor cost anomalies
   - Review Cost Explorer monthly
   - Identify and eliminate waste

## Security & Privacy Practices

### Data Protection

1. **Encryption**
   - TLS 1.3 for all data in transit
   - AES-256 encryption for data at rest (S3, DynamoDB)
   - KMS for key management
   - Field-level encryption for sensitive data

2. **Access Control**
   - IAM roles with least privilege
   - Cognito user pools for authentication
   - API Gateway authorizers
   - Resource-based policies

3. **Data Privacy**
   - HIPAA-aligned architecture
   - PHI (Protected Health Information) handling
   - Data residency in Indian regions
   - Right to be forgotten (data deletion)

4. **Audit and Compliance**
   - CloudTrail for API audit logs
   - CloudWatch Logs for application logs
   - VPC Flow Logs for network traffic
   - Regular security audits

### Security Best Practices

1. **API Security**
   - Rate limiting and throttling
   - Input validation and sanitization
   - CORS configuration
   - API key rotation

2. **Lambda Security**
   - No hardcoded credentials
   - Use Secrets Manager for sensitive data
   - VPC for database access
   - Security scanning of dependencies

3. **Database Security**
   - DynamoDB encryption at rest
   - Fine-grained access control
   - Backup and point-in-time recovery
   - No public access

4. **Monitoring and Alerting**
   - CloudWatch alarms for suspicious activity
   - GuardDuty for threat detection
   - Security Hub for compliance
   - Incident response plan

## 1-Minute Hackathon Pitch

**Problem:**
500 million Indians lack access to quality healthcare information. Medical records are fragmented, language barriers prevent health literacy, and rural areas have limited doctor availability. Patients struggle with medication adherence and don't know when symptoms require urgent care.

**Solution:**
SwasthyaMitra is an AI-powered healthcare assistant that puts personalized health management in every Indian's pocket. Using Amazon Bedrock's AI agents, we provide:
- Lifelong medical history storage
- Instant prescription OCR and medicine reminders
- AI symptom checking in 6 regional languages
- Drug interaction alerts to prevent dangerous combinations
- Voice interface for illiterate users
- Nearby doctor and hospital discovery

**Technology:**
Built entirely on AWS serverless architecture - Lambda, Bedrock, Textract, DynamoDB, Cognito. Low-bandwidth optimized for rural connectivity. HIPAA-aligned security for sensitive health data.

**Impact:**
- Reduce preventable medication errors by 40%
- Increase treatment adherence by 50%
- Provide 24/7 health guidance to underserved communities
- Empower patients with their own health data
- Bridge the urban-rural healthcare gap

**Business Model:**
Freemium for individuals, B2B partnerships with hospitals and insurance companies, government subsidies for rural deployment.

**Ask:**
We're seeking partnerships with healthcare providers, government health departments, and investors to scale SwasthyaMitra to 100 million users across India.

---

**Demo Script:**
1. Show user registration with phone OTP
2. Upload prescription photo → instant OCR extraction
3. Ask symptom question in Hindi → get AI response with recommendations
4. Check drug interaction → show alert for dangerous combination
5. Search nearby hospitals → show map with providers
6. Show medicine reminder notification
7. Display medical history timeline

**Wow Factor:**
"Watch as I speak to SwasthyaMitra in Tamil about my fever, and it instantly understands, checks my medical history for allergies, and gives me personalized advice - all in under 3 seconds!"

## Future Enhancements

### Phase 2 (3-6 months)
1. Telemedicine integration for video consultations
2. Prescription delivery marketplace
3. Health insurance claim automation
4. Lab test booking and report integration
5. Mental health support chatbot
6. Nutrition and diet planning

### Phase 3 (6-12 months)
1. AI-powered diagnosis assistance for doctors
2. Clinical decision support system
3. Population health analytics
4. Epidemic outbreak prediction
5. Medical research data contribution
6. Integration with national health stack (ABDM)

### Phase 4 (12+ months)
1. Expansion to other countries (Bangladesh, Nepal, Sri Lanka)
2. Chronic disease management programs
3. Elderly care monitoring
4. Maternal and child health tracking
5. Genomics and personalized medicine
6. Medical education and training platform
