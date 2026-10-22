# Специфікація динаміки сценарію (spec.md)

## 1. Обраний Use Case
* **Прецедент використання:** `UC-2: Бронювання заняття` (із розширенням `UC-4: Обробка оплати`).
* **Мета:** Описати динаміку взаємодії компонентів системи під час створення замовлення на урок, перевірки доступності слоту, обробки платежу та оновлення статусів у базі даних.

---

## 2. Учасники сценарію (Учасники та Сервіси)

* **`Student` (Актор):** Учень, який ініціює процес бронювання через веб-інтерфейс.
* **`ClientUI` (Компонент):** Фронтенд-застосунок (клієнтський інтерфейс).
* **`APIGateway` (Компонент):** Вхідний маршрутизатор та точка аутентифікації API.
* **`BookingService` (Сервіс):** Сервіс бізнес-логіки для управління розкладом та уроками.
* **`PaymentService` (Сервіс):** Сервіс для обробки транзакцій та взаємодії з платіжними провайдерами.
* **`Database` (База даних):** Реляційна БД (таблиці `Lesson`, `TutorProfile`, `User`).

---

## 3. Покроковий опис сценарію

### 3.1. Основний успішний потік (Main Flow — Success)
1. **Запит бронювання:** Учень обирає вільний слот у календарі та натискає «Забронювати». `ClientUI` надсилає запит `postBookingSlot(tutor_id, subject_id, start_time)` до `APIGateway`.
2. **Маршрутизація:** `APIGateway` перевіряє токен авторизації та викликає метод `createBooking(student_id, tutor_id, subject_id, start_time)` у `BookingService`.
3. **Перевірка слоту:** `BookingService` робить запит до `Database` методом `checkSlotAvailability(tutor_id, start_time)`.
4. **Створення чернетки:** `Database` підтверджує, що слот вільний. `BookingService` виконує `saveLessonDraft(status='Pending_Payment')` у `Database` та повертає `booking_id`.
5. **Ініціалізація оплати:** `BookingService` звертається до `PaymentService` викликаючи `initiatePayment(booking_id, amount)`.
6. **Перенаправлення на оплату:** `PaymentService` повертає платіжне посилання, яке через `APIGateway` та `ClientUI` відображається учню.
7. **Підтвердження платежу:** Учень здійснює оплату. `PaymentService` отримує Webhook та надсилає `processPaymentCallback(booking_id, success=true)` до `BookingService`.
8. **Фіналізація:** `BookingService` оновлює статус заняття в `Database` викликаючи `updateLessonStatus(booking_id, status='Confirmed')`. `Database` повертає успішний результат, і `ClientUI` відображає підтвердження бронювання.

### 3.2. Альтернативний потік — Слот вже зайнятий (Alternative Flow 1 — Slot Unavailable)
* **Умова:** Під час перевірки слоту в БД виявляється, що інший користувач уже забронював цей час.
* **Перебіг:** `Database` повертає `SlotTakenError`. `BookingService` повертає помилку через `APIGateway` до `ClientUI`. Процес переривається, створювати запис у БД та проводити оплату не потрібно.

### 3.3. Альтернативний потік — Помилка оплати (Alternative Flow 2 — Payment Failure)
* **Умова:** Платіжна система повертає відмову в транзакції або вичерпано 15-хвилинний тайм-аут.
* **Перебіг:** `PaymentService` передає `processPaymentCallback(booking_id, success=false)` до `BookingService`. `BookingService` викликає `updateLessonStatus(booking_id, status='Available')` у `Database`, звільняючи слот для інших користувачів, та повертає `PaymentFailedError` у `ClientUI`.
