# Компонентна діаграма системи (components.md)

```mermaid
graph TD
    subgraph TutorPlatform ["System: TutorPlatform"]
        WebClient["WebClient (UI)"]
        APIGateway["APIGateway"]
        AuthService["AuthService"]
        CatalogService["CatalogService"]
        BookingService["BookingService"]
        PaymentService["PaymentService"]
        NotificationService["NotificationService"]
        DataAccessLayer["DataAccessLayer"]
        Database[("Database (PostgreSQL)")]

        %% Помилка 1: зовнішні сервіси всередині межі системи
        PaymentGateway["PaymentGateway (External)"]
        NotificationProvider["NotificationProvider (External)"]
        OAuthProvider["OAuthProvider (External)"]
    end

    %% Помилка 2: порушення шарів (прямі звернення)
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
