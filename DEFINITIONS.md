# DEFINICE - AI Video Hanzl Platform

## 1. CORE DEFINITIONS

### AI Video Generation
- **Definition**: Automatizovaný proces vytváření videálního obsahu pomocí umělé inteligence bez tradiční videokamery či editace.
- **Purpose**: Zrychlení produkce videa, snížení nákladů, škálovatelnost obsahu.
- **Scope**: Text-to-video, Image-to-video, Avatar-based video, Voice synthesis.

### Platform Architecture
- **Definition**: Integrovaný systém pro správu, generování a distribuci AI videí.
- **Components**: 
  - API Gateway
  - Video Engine (Sora, Runway, HeyGen)
  - Storage Layer
  - Processing Pipeline
  - Analytics Dashboard

---

## 2. KEY TOOLS & THEIR DEFINITIONS

### **Sora (OpenAI)**
- **Type**: Photorealistic Video Generator
- **Input**: Text prompt (natural language description)
- **Output**: 1080p video (up to 1 minute)
- **Strength**: Cinematic quality, realistic physics
- **Use Case**: Filmmaking, advertising, high-end creative content
- **Traceability**: Every generation logged with prompt, timestamp, version

### **Runway (Gen-3/Gen-4)**
- **Type**: Professional Creative Tool
- **Input**: Text prompt, image, or video frame
- **Output**: Variable length video with timeline control
- **Strength**: Fast iteration, professional workflows, batch processing
- **Use Case**: Marketing, social media, explainer videos
- **Traceability**: Session-based, project history, version control

### **HeyGen**
- **Type**: Avatar & Speech Synthesis Engine
- **Input**: Script + Avatar selection + voice parameters
- **Output**: Talking-head video with lip-sync
- **Strength**: Multilingual support (40+ languages), scalable communication
- **Use Case**: Corporate training, sales videos, global communications
- **Traceability**: Script versioning, avatar templates, language tracking

### **Pika Labs**
- **Type**: Fast Animation Generator
- **Input**: Text or image reference
- **Output**: Short animated sequences
- **Strength**: Speed, affordability
- **Use Case**: Memes, quick explainers, social content
- **Traceability**: Generation queue, output archive

### **Kling AI**
- **Type**: Extended Video Generator
- **Input**: Text prompt or image
- **Output**: Longer video duration (budget-friendly)
- **Strength**: Cost-effective, extended duration
- **Use Case**: Budget productions, longer narratives
- **Traceability**: Batch processing, cost tracking

---

## 3. PROJECT STRUCTURE DEFINITIONS

### **Video Project**
```
Project = {
  id: UUID,
  name: String,
  description: String,
  engine: Enum[Sora | Runway | HeyGen | Pika | Kling],
  status: Enum[draft | processing | completed | failed],
  created_at: Timestamp,
  updated_at: Timestamp,
  creator: User,
  videos: [Video],
  metadata: Object
}
```

### **Video Object**
```
Video = {
  id: UUID,
  project_id: UUID,
  prompt: String,
  engine: String,
  duration: Number (seconds),
  resolution: String (1080p, 4K, etc),
  status: Enum[queued | generating | completed | error],
  output_url: String,
  error_message: String | null,
  generation_time: Number (seconds),
  created_at: Timestamp,
  completed_at: Timestamp | null
}
```

### **Avatar Definition** (HeyGen)
```
Avatar = {
  id: UUID,
  name: String,
  template_id: String,
  language: String,
  voice_id: String,
  emotion: Enum[neutral | happy | sad | serious],
  gesture_intensity: Enum[low | medium | high]
}
```

---

## 4. WORKFLOW DEFINITIONS

### **Standard Video Generation Pipeline**
1. **Input Stage**: User provides prompt, selects engine, configures parameters
2. **Validation Stage**: Prompt analysis, content policy check, resource availability
3. **Queue Stage**: Job added to processing queue with priority
4. **Generation Stage**: Engine processes video (duration varies by tool)
5. **Post-Processing**: Quality check, optimization, format conversion
6. **Output Stage**: Video stored, URL generated, metadata logged
7. **Delivery Stage**: User retrieval, CDN distribution

### **Traceability Chain**
```
Input → Validation → Queue → Generation → Post-Processing → Output → Delivery
  ↓         ↓          ↓         ↓            ↓              ↓         ↓
 LOG      LOG         LOG       LOG          LOG            LOG       LOG
```

---

## 5. DATA STRUCTURES

### **Generation Request**
```json
{
  "project_id": "UUID",
  "engine": "sora|runway|heygen|pika|kling",
  "prompt": "Detailed text description",
  "parameters": {
    "duration": 30,
    "resolution": "1080p",
    "style": "cinematic|cartoon|realistic",
    "voice": "en-US|cs-CZ|...",
    "avatar_id": "UUID (HeyGen only)"
  },
  "priority": "low|normal|high",
  "callback_url": "https://..."
}
```

### **Generation Response**
```json
{
  "job_id": "UUID",
  "status": "accepted|processing|completed|failed",
  "video_id": "UUID",
  "output_url": "https://...",
  "estimated_completion": "2026-09-29T15:30:00Z",
  "metadata": {
    "engine_version": "1.0",
    "processing_time_seconds": 45,
    "file_size_mb": 125.5
  }
}
```

---

## 6. QUALITY METRICS DEFINITIONS

### **Video Quality Score**
- **Resolution Compliance**: Video matches requested resolution ✓/✗
- **Duration Accuracy**: Video within ±2 seconds of requested duration
- **Content Fidelity**: Generated content matches prompt description (1-10 scale)
- **Audio Sync** (HeyGen): Lip-sync accuracy (1-10 scale)
- **Rendering Quality**: No artifacts, smooth motion, proper colors

### **Performance Metrics**
- **Generation Time**: Total seconds from job submission to completion
- **Queue Wait Time**: Seconds before generation started
- **Success Rate**: % of successful generations per engine
- **API Response Time**: <200ms for all requests
- **Uptime**: 99.9% availability target

---

## 7. ERROR DEFINITIONS

### **Critical Errors**
- `ENGINE_UNAVAILABLE`: Selected engine is down
- `INVALID_PROMPT`: Prompt violates content policy
- `INSUFFICIENT_CREDITS`: User account lacks resources
- `DATABASE_ERROR`: Data persistence failure

### **Recoverable Errors**
- `QUEUE_TIMEOUT`: Generation took too long, retry possible
- `PARTIAL_GENERATION`: Output incomplete, regenerate
- `FORMAT_ERROR`: Output requires re-encoding

### **User-Level Errors**
- `INVALID_PARAMETERS`: Misconfigured request
- `FILE_NOT_FOUND`: Referenced asset missing
- `QUOTA_EXCEEDED`: Rate limit hit

---

## 8. SECURITY & TRACEABILITY

### **Audit Log Entry**
```
{
  timestamp: ISO-8601,
  user_id: UUID,
  action: String,
  resource: String,
  resource_id: UUID,
  details: Object,
  ip_address: String,
  user_agent: String,
  status: success|failure
}
```

### **Data Retention**
- **Video Files**: 30 days (user can extend)
- **Metadata**: Indefinite (traceability requirement)
- **Audit Logs**: 1 year minimum
- **Failed Jobs**: 7 days for analysis

---

## 9. INTEGRATION DEFINITIONS

### **Webhook Event**
```json
{
  "event_type": "video.generated|video.failed|job.queued",
  "timestamp": "2026-09-29T12:00:00Z",
  "video_id": "UUID",
  "project_id": "UUID",
  "status": "completed|failed",
  "payload": { ... }
}
```

### **API Versioning**
- **Current**: v1.0
- **Deprecation**: Versions sunset 12 months after new version release
- **Breaking Changes**: Announced 30 days in advance

---

## 10. TERMINOLOGY REFERENCE

| Term | Definition | Context |
|------|-----------|---------|
| **Prompt** | Natural language description of desired video | Input |
| **Engine** | AI model/service generating video | Selection |
| **Pipeline** | Series of processing steps | Workflow |
| **Queue** | Job waiting list ordered by priority | Processing |
| **Avatar** | AI-generated character/person | HeyGen |
| **Lip-Sync** | Audio-visual synchronization for speech | Quality |
| **Traceability** | Ability to track every action/change | Core Principle |
| **Artifact** | Visual error/distortion in output | Quality Issue |
| **CDN** | Content Delivery Network for distribution | Infrastructure |
| **Webhook** | Real-time event notification | Integration |

---

**Document Version**: 1.0  
**Last Updated**: 2026-09-29  
**Author**: Jaroslav Hanzl MasterCaryl Andrelan  
**Traceability**: All changes logged and traceable within this system.
