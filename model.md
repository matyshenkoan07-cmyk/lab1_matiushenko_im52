# Модель даних (ER-діаграма)

```mermaid
erDiagram
    USER ||--o{ TUTOR_PROFILE : "has"
    USER ||--o{ LESSON : "participates"
    
    USER {
        int id PK
        string full_name
        string email
        string role
        datetime created_at
    }

    TUTOR_PROFILE {
        int id PK
        int tutor_id FK
        string experience
        decimal price_per_hour
        text biography
    }

    SUBJECT {
        int id PK
        string name
        string description
    }

    LESSON {
        int id PK
        int student_id FK
        int tutor_id FK
        datetime start_time
        int duration_minutes
        string status
        decimal price
    }
