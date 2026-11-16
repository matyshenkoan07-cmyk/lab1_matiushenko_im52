# Компонентна діаграма системи (components.md)

```mermaid
graph TD
    subgraph TutorPlatform ["System Boundary: TutorPlatform"]
        WebClient["«component» WebClient"]
        APIGateway["«component» APIGateway"]
        AuthService["«component» AuthService"]
        CatalogService["«component» CatalogService"]
        BookingService["«component» BookingService"]
        PaymentService["«component» PaymentService"]
        NotificationService["«component» NotificationService"]
        DataAccessLayer["«component» DataAccessLayer"]
        Database[("«database» PostgreSQL")]
    end

    %% Зовнішні інтеграції поза межами системи
    subgraph ExternalSystems ["External Integrations (Out of Boundary)"]
        OAuthProvider["«external» OAuthProvider"]
        PaymentGateway["«external» PaymentGateway"]
        NotificationProvider["«external» NotificationProvider"]
    end

    WebClient --> APIGateway
    WebClient --> CatalogService

    APIGateway --> AuthService
    APIGateway --> BookingService

    AuthService --> OAuthProvider
    AuthService --> DataAccessLayer

    CatalogService --> DataAccessLayer

    BookingService --> PaymentService
    BookingService --> NotificationService
    BookingService --> Database

    PaymentService --> PaymentGateway
    NotificationService --> NotificationProvider

    DataAccessLayer --> Database
