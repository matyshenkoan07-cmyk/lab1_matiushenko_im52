# Компонентна діаграма системи (components.md)

```mermaid
graph TD
    subgraph TutorPlatform ["System Boundary: TutorPlatform"]
        WebClient["«component» WebClient"]
        APIGateway["«component» APIGateway"]

        %% Provided / Required інтерфейси сервісів
        IAuthAPI(["«interface» IAuthAPI"])
        ICatalogAPI(["«interface» ICatalogAPI"])
        IBookingAPI(["«interface» IBookingAPI"])
        IPaymentAPI(["«interface» IPaymentAPI"])
        IPaymentWebhook(["«interface» IPaymentWebhook"])
        INotificationSender(["«interface» INotificationSender"])
        IDataRepository(["«interface» IDataRepository"])

        AuthService["«component» AuthService"]
        CatalogService["«component» CatalogService"]
        BookingService["«component» BookingService"]
        PaymentService["«component» PaymentService"]
        NotificationService["«component» NotificationService"]
        DataAccessLayer["«component» DataAccessLayer"]
        Database[("«database» PostgreSQL")]

        %% Вхідний шар
        WebClient -. "required" .-> APIGateway

        APIGateway -. "required" .-> IAuthAPI
        APIGateway -. "required" .-> ICatalogAPI
        APIGateway -. "required" .-> IBookingAPI

        IAuthAPI --- AuthService
        ICatalogAPI --- CatalogService
        IBookingAPI --- BookingService

        %% Взаємодія між бізнес-сервісами
        BookingService -. "required" .-> IPaymentAPI
        BookingService -. "required" .-> INotificationSender

        IPaymentAPI --- PaymentService
        IPaymentWebhook --- PaymentService
        INotificationSender --- NotificationService

        %% Доступ до даних через інтерфейс репозиторію
        AuthService -. "required" .-> IDataRepository
        CatalogService -. "required" .-> IDataRepository
        BookingService -. "required" .-> IDataRepository

        IDataRepository --- DataAccessLayer
        DataAccessLayer --> Database
    end

    subgraph ExternalSystems ["External Integrations (Out of Boundary)"]
        OAuthProvider["«external» OAuthProvider"]
        PaymentGateway["«external» PaymentGateway"]
        NotificationProvider["«external» NotificationProvider"]
    end

    %% Зовнішні інтеграції
    AuthService -. "OAuth 2.0" .-> OAuthProvider
    PaymentService -. "init payment" .-> PaymentGateway
    PaymentGateway -. "required (webhook)" .-> IPaymentWebhook
    NotificationService -. "push/email" .-> NotificationProvider
