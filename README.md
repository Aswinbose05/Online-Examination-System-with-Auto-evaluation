# 🎓 Cloud-Based Online Examination System with NLP Auto-Evaluation

A complete Final Year Project implementing a cloud-ready online exam portal with JWT authentication, role-based access (Admin & Student), and automatic answer evaluation using TF-IDF Cosine Similarity NLP.

---

## 📐 Architecture

```
┌─────────────────────────────────────────────────┐
│                   Frontend                       │
│         HTML + CSS + JavaScript (SPA)            │
│         Served on port 5500 / any static server  │
└────────────────────┬────────────────────────────┘
                     │ REST API (HTTP/JSON)
┌────────────────────▼────────────────────────────┐
│           Spring Boot Backend (Java)             │
│    JWT Auth · RBAC · JPA · REST Controllers      │
│              Runs on port 8080                   │
└──────────┬──────────────────────┬───────────────┘
           │ JPA (H2/MySQL)       │ REST (HTTP)
┌──────────▼──────────┐  ┌────────▼──────────────┐
│   H2 / MySQL DB     │  │  Python NLP Service   │
│  (in-memory/file)   │  │  Flask + TF-IDF +     │
│                     │  │  Cosine Similarity    │
└─────────────────────┘  └───────────────────────┘
                                port 5000
```

---

## 📁 Project Structure

```
exam-system/
├── backend/                        # Spring Boot (Java 17, Maven)
│   ├── pom.xml
│   └── src/main/java/com/examportal/
│       ├── ExamPortalApplication.java
│       ├── config/
│       │   ├── DataInitializer.java    # Seeds admin + sample data
│       │   └── SecurityConfig.java     # JWT + CORS config
│       ├── controller/
│       │   ├── AuthController.java
│       │   ├── AdminController.java
│       │   └── StudentController.java
│       ├── dto/
│       │   ├── AuthDTO.java
│       │   ├── ExamDTO.java
│       │   └── SubmissionDTO.java
│       ├── entity/
│       │   ├── User.java
│       │   ├── Exam.java
│       │   ├── Question.java
│       │   ├── Submission.java
│       │   └── Answer.java
│       ├── repository/             # Spring Data JPA repos
│       ├── security/
│       │   ├── JwtUtil.java
│       │   └── JwtAuthFilter.java
│       └── service/
│           ├── AuthService.java
│           ├── CustomUserDetailsService.java
│           ├── ExamService.java
│           ├── NlpService.java         # Calls Python NLP microservice
│           └── SubmissionService.java  # Auto-evaluation logic
│
├── nlp-service/                    # Python Flask NLP Microservice
│   ├── app.py                      # Flask routes
│   ├── nlp_evaluator.py            # TF-IDF + Cosine Similarity engine
│   ├── requirements.txt
│   ├── download_nltk_data.py
│   └── Procfile                    # For cloud deployment
│
├── frontend/                       # Vanilla HTML/CSS/JS SPA
│   ├── index.html                  # Single page app
│   ├── css/
│   │   └── style.css
│   └── js/
│       ├── api.js                  # API client
│       ├── utils.js                # Timer, toast, anti-cheat
│       ├── auth.js                 # Login/register/routing
│       ├── admin.js                # Admin dashboard
│       └── student.js              # Student exam & results
│
├── docker-compose.yml              # Docker orchestration
└── README.md
```

---

## 🚀 Quick Start (Local Development)

### Prerequisites
- Java 17+
- Maven 3.8+
- Python 3.9+
- pip

---

### Step 1 — Start the NLP Service

```bash
cd nlp-service

# Install dependencies
pip install -r requirements.txt

# Download NLTK data (first time only)
python download_nltk_data.py

# Start Flask server
python app.py
# → Runs on http://localhost:5000
```

Test it:
```bash
curl -X POST http://localhost:5000/evaluate \
  -H "Content-Type: application/json" \
  -d '{"student_answer":"Java uses OOP concepts like inheritance and polymorphism","model_answer":"Java is an object-oriented language supporting inheritance, polymorphism, encapsulation and abstraction","max_marks":10}'
```

---

### Step 2 — Start the Spring Boot Backend

```bash
cd backend

# Build and run
mvn spring-boot:run

# → Runs on http://localhost:8080
# → H2 console: http://localhost:8080/h2-console
```

On startup, sample data is automatically created:
| Role    | Username  | Password    |
|---------|-----------|-------------|
| Admin   | admin     | admin123    |
| Student | student1  | student123  |
| Student | student2  | student123  |

---

### Step 3 — Open the Frontend


# Option 2: Use a local server (recommended - avoids CORS)
python -m http.server 5500
# → Open http://localhost:5500
```

> **Note:** If you open `index.html` directly (file://), ensure the Spring Boot CORS config allows `null` origin or use a local server.

---

## 🔧 Configuration

### Switch to MySQL (Production)

In `backend/src/main/resources/application.properties`:

```properties
# Comment out H2
# spring.datasource.url=jdbc:h2:mem:examdb...
# spring.h2.console.enabled=true
# spring.jpa.database-platform=org.hibernate.dialect.H2Dialect

# Uncomment MySQL
spring.datasource.url=jdbc:mysql://localhost:3306/examdb?useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=yourpassword
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.database-platform=org.hibernate.dialect.MySQLDialect
spring.jpa.hibernate.ddl-auto=update
```

Create the MySQL database:
```sql
CREATE DATABASE examdb;
```

### Change JWT Secret
In `application.properties`:
```properties
app.jwt.secret=YourVeryLongSecretKeyHere_AtLeast32Chars
app.jwt.expiration=86400000
```

### Change NLP Service URL
```properties
nlp.service.url=http://your-nlp-service-host:5000
```

---

## 🐳 Docker Deployment

```bash
# Build and run all services
docker-compose up --build

# Stop
docker-compose down
```

Services exposed:
- Frontend: http://localhost:80
- Backend: http://localhost:8080
- NLP Service: http://localhost:5000

---

## ☁️ Cloud Deployment

### Railway / Render (Backend)

1. Push to GitHub
2. Connect repo to Railway/Render
3. Set environment variables:
   - `SPRING_DATASOURCE_URL`
   - `SPRING_DATASOURCE_USERNAME`
   - `SPRING_DATASOURCE_PASSWORD`
   - `APP_JWT_SECRET`
   - `NLP_SERVICE_URL`

### Render (NLP Service)

1. New Web Service → connect repo → set root to `nlp-service/`
2. Build: `pip install -r requirements.txt && python download_nltk_data.py`
3. Start: `gunicorn app:app`

### AWS Elastic Beanstalk (Backend)

```bash
cd backend
mvn package
# Upload target/exam-portal-backend-1.0.0.jar to Elastic Beanstalk
```

---

## 📡 API Endpoints

### Auth
| Method | URL | Description |
|--------|-----|-------------|
| POST | `/api/auth/login` | Login (returns JWT) |
| POST | `/api/auth/register` | Register student |
| GET | `/api/auth/health` | Health check |

### Admin (requires `ROLE_ADMIN`)
| Method | URL | Description |
|--------|-----|-------------|
| POST | `/api/admin/exams` | Create exam |
| GET | `/api/admin/exams` | List all exams |
| PUT | `/api/admin/exams/{id}/toggle` | Activate/deactivate |
| DELETE | `/api/admin/exams/{id}` | Delete exam |
| POST | `/api/admin/exams/{id}/questions` | Add question |
| GET | `/api/admin/exams/{id}/questions` | List questions |
| DELETE | `/api/admin/questions/{id}` | Delete question |
| GET | `/api/admin/submissions` | All submissions |
| GET | `/api/admin/submissions/{id}` | Submission detail |

### Student (requires `ROLE_STUDENT`)
| Method | URL | Description |
|--------|-----|-------------|
| GET | `/api/student/exams` | Available exams |
| GET | `/api/student/exams/{id}/start` | Start exam (get questions) |
| POST | `/api/student/exams/submit` | Submit answers |
| GET | `/api/student/submissions` | My submissions |
| GET | `/api/student/submissions/{id}` | My result detail |

### NLP Service
| Method | URL | Description |
|--------|-----|-------------|
| POST | `/evaluate` | Evaluate single answer |
| POST | `/batch-evaluate` | Evaluate multiple answers |
| POST | `/preprocess` | Debug text preprocessing |
| GET | `/health` | Service health |

---

## 🧠 NLP Evaluation Details

The evaluation pipeline:
```
Student Answer → Lowercase → Remove Punctuation → Tokenize
             → Remove Stopwords → Stem/Lemmatize → TF-IDF Vector
             → Cosine Similarity with Model Answer → Score
```

**Scoring Formula:**
```
combined_similarity = (tfidf_cosine_sim × 0.7) + (keyword_overlap × 0.3)
final_similarity    = combined_similarity × length_penalty_factor
score               = round(final_similarity × max_marks, 0.5)
```

**Feedback Levels:**
| Similarity | Level | Feedback |
|-----------|-------|---------|
| ≥ 85% | Excellent | Full marks range |
| 70–84% | Good | Minor gaps noted |
| 50–69% | Average | Key concepts missing |
| 30–49% | Below Average | Needs improvement |
| 10–29% | Poor | Major review needed |
| < 10% | Very Poor | Insufficient answer |

**Fallback:** If the NLP service is offline, the backend uses a basic word-overlap algorithm automatically. No crashes.

---

## 🔐 Security Features

- JWT Bearer token authentication
- Role-based access control (ADMIN / STUDENT)
- BCrypt password hashing
- CORS configuration
- Exam answers hidden from students until after submission

## 🛡️ Anti-Cheat Features

- Countdown timer with auto-submit on expiry
- Tab-switch detection (violation counter)
- Browser beforeunload warning
- One submission per student per exam

---

## 🗄️ Database Schema

```sql
users       (id, username, password, email, full_name, role, is_active, created_at)
exams       (id, title, description, duration_minutes, total_marks, passing_marks,
             start_time, end_time, is_active, is_random, random_question_count, created_by, created_at)
questions   (id, question_text, type, marks, option_a..d, correct_answer, model_answer,
             explanation, exam_id)
submissions (id, student_id, exam_id, started_at, submitted_at, total_score, max_score,
             percentage, passed, status)
answers     (id, submission_id, question_id, student_answer, score_obtained, max_score,
             is_correct, nlp_similarity, feedback, evaluated)
```

---

## 📋 Technologies Used

| Layer | Technology |
|-------|-----------|
| Backend | Java 17, Spring Boot 3.2, Spring Security, Spring Data JPA |
| Auth | JWT (jjwt 0.11.5), BCrypt |
| Database | H2 (dev), MySQL (prod) |
| NLP Service | Python 3.9+, Flask 3.0, NLTK, scikit-learn |
| NLP Algorithm | TF-IDF Vectorization + Cosine Similarity |
| Frontend | HTML5, CSS3, Vanilla JavaScript |
| Build | Maven 3.8+ |
| Deployment | Docker, Railway, Render, AWS EB |

---

## 👥 Team / Credits

Final Year Project — Computer Science / Information Technology  
**Project Title:** Cloud-Based Online Examination System with Auto Evaluation using NLP

---
