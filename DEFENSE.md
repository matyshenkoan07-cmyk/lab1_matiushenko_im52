# DEFENSE — Завдання 5: Архітектура (Component)

**Намір і критерії (1–2 речення):**  
Було побудовано UML Component Diagram для платформи онлайн-репетиторства `TutorPlatform` у синтаксисі Mermaid `flowchart` (`components.md`) на основі складеної специфікації `spec.md`. Критерії: чітка межа системи з виносом зовнішніх провайдерів (`OAuthProvider`, `PaymentGateway`, `NotificationProvider`), явні `provided`/`required`-інтерфейси, суворе шарування залежностей (`UI` → `APIGateway` → `Services` → `DataAccessLayer` → `PostgreSQL`) та узгодженість із моделями Лабораторних 1–4.

---

### Топ-3 розбіжності (`spec` ↔ артефакт) + коміт-виправлення

* **Порушення межі системи (`AC-1`):** у початковій генерації зовнішні інтеграції (`PaymentGateway`, `NotificationProvider`, `OAuthProvider`) знаходилися всередині контейнера `TutorPlatform` замість виносу за межу системи; уточнено межі та винесено ці вузли в окремий блок `External Integrations (Out of Boundary)`.  
  **Коміт:** `<SHA-1>` (`fix: move external providers outside TutorPlatform system boundary (AC-1)`)
* **Порушення шарування залежностей (`AC-3`):** у сирій діаграмі `WebClient` звертався напряму до `CatalogService` в обхід `APIGateway`, а `BookingService` мав прямий зв'язок із `PostgreSQL` в обхід `DataAccessLayer`; усі обхідні зв'язки видалено та перенаправлено через `APIGateway` і `DataAccessLayer`.  
  **Коміт:** `<SHA-2>` (`fix: enforce layered dependencies through APIGateway and DataAccessLayer (AC-3)`)
* **Відсутність явних `provided`/`required` інтерфейсів і вхідного webhook (`AC-2`, `AC-4`):** початкова діаграма містила лише прямі стрілки між компонентами без типізованих контрактів і зворотного виклику від платіжного шлюзу; додано 7 інтерфейсів (`IAuthAPI`, `ICatalogAPI`, `IBookingAPI`, `IPaymentAPI`, `IPaymentWebhook`, `INotificationSender`, `IDataRepository`) із розділенням реалізації (`provided`) та залежностей (`required`).  
  **Коміт:** `<SHA-3>` (`fix: add explicit provided/required interfaces and incoming payment webhook (AC-2, AC-4)`)

---

### Ключове рішення — які альтернативи зважив і чому обрав цю (ADR)

Для вибору архітектурного стилю та нотації діаграми порівнювалися три варіанти (детально зафіксовано в `ADR-005.md`):

1. **Мікросервісна архітектура з окремими БД та брокером повідомлень (Kafka / RabbitMQ):** відхилено через надмірну складність для поточної версії та розходження з єдиною схемою БД (`PostgreSQL`), спроєктованою в Лабораторній 1.
2. **PlantUML Component Diagram / draw.io (XML/SVG):** відхилено через відсутність нативного рендерингу PlantUML у GitHub Markdown та незручний для рев'ю в Git diff громіздкий XML-код.
3. **Модульний моноліт із єдиною БД (`PostgreSQL`) та Mermaid `flowchart` (обрано):** поєднує пряму спадкоємність із моделлю даних Лаби 1, нативний рендеринг у GitHub, прозорий текстовий diff та чітку візуалізацію меж системи й `provided`/`required`-інтерфейсів.

---

### Перевірка — узгодженість із усім ланцюгом моделей (як перевірив)

Усі внутрішні сервіси ізольовані в `TutorPlatform`, зовнішні провайдери винесені назовні, а кожна міжкомпонентна взаємодія проходить через визначений інтерфейс (`provided`/`required`) без прямого доступу UI чи сервісів до `PostgreSQL`. Компонент `DataAccessLayer` через інтерфейс `IDataRepository` інкапсулює роботу з базою даних `PostgreSQL` та всіма чотирма базовими сутностями (`Student`, `Tutor`, `Subject`, `Lesson`) з Лабораторної 1. Сервіси `AuthService` (`IAuthAPI`), `CatalogService` (`ICatalogAPI`) та `BookingService` (`IBookingAPI`) повністю покривають сценарії `UC-1..UC-4` (реєстрація/вхід, пошук репетитора, бронювання слоту та оплата) з Лабораторної 2. Порядок викликів `WebClient` → `APIGateway` → `BookingService` → `PaymentService` → `PaymentGateway` та зворотний виклик через `IPaymentWebhook` точно відтворюють послідовність повідомлень із sequence-діаграми Лабораторної 3, а розділення відповідальності між `BookingService`, `PaymentService` та `NotificationService` (`INotificationSender`) відповідає трьом бізнес-доріжкам та паралельним діям (`FORK`/`JOIN`) підтвердження уроку (`Confirmed`) і відправки сповіщень з Лабораторної 4.

---

**Здача:** (а) посилання на GitHub PR; (б) цей заповнений `DEFENSE.md`.
