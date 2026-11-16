# Промпт для генерації UML Component Diagram (`13-components-prompt.md`)

## Контекст та Мета
Потрібно побудувати UML Component Diagram у декларативному синтаксисі Mermaid (`graph TD`) для платформи онлайн-репетиторства на основі затвердженого `spec.md`, об'єднавши моделі з Лабораторних 1–4.

## Вимоги до діаграми:
1. **Межа системи (`AC-1`):** Чітко відокремити внутрішній контейнер `System Boundary: TutorPlatform` (модулі `WebClient`, `APIGateway`, `AuthService`, `CatalogService`, `BookingService`, `PaymentService`, `NotificationService`, `DataAccessLayer`, `PostgreSQL`) від зовнішніх інтеграцій (`OAuthProvider`, `PaymentGateway`, `NotificationProvider`) у блоці `External Integrations`.
2. **Інтерфейси `provided` / `required` (`AC-2`):** Явно зобразити вузли інтерфейсів (`IAuthAPI`, `ICatalogAPI`, `IBookingAPI`, `IPaymentAPI`, `IPaymentWebhook`, `INotificationSender`, `IDataRepository`) із суцільними лініями реалізації (`---`) та пунктирними стрілками залежностей (`-. "required" .->`).
3. **Шарування залежностей (`AC-3`):** `WebClient` звертається виключно до `APIGateway`; бізнес-сервіси взаємодіють із `PostgreSQL` лише через `IDataRepository` та `DataAccessLayer` без прямих обхідних зв'язків.
4. **Зовнішні взаємодії (`AC-4`):** Відобразити ініціацію оплати від `PaymentService` до `PaymentGateway`, вхідний webhook через `IPaymentWebhook`, OAuth-авторизацію та відправку сповіщень.
5. **Поза обсягом (`AC-5`):** Не додавати кешування (Redis), брокери повідомлень (Kafka/RabbitMQ) та CDN.

Згенеруй Mermaid-код компонентної діаграми для файлу `components.md`.
