# Прецеденти використання (Use Cases)

## 1. Опис Use Cases

* **UC-1: Аутентифікація (Auth)** — Вхід та реєстрація користувачів у системі.
* **UC-2: Бронювання заняття (Book Lesson)** — Вибір вільного слоту та створення запису про заняття.
* **UC-3: Перевірка доступності слоту (Check Slot Availability)** — Внутрішня перевірка відсутності накладок у розкладі.
* **UC-4: Обробка оплати (Process Payment)** — Взаємодія із зовнішньою платіжною системою для підтвердження транзакції.
* **UC-5: Налаштування розкладу (Manage Schedule)** — Додавання та редагування вільних слотів репетитором.

## 2. Use Case Diagram (Mermaid)

```mermaid
graph LR
    %% Актори
    Student["Учень (Student)"]
    Tutor["Репетитор (Tutor)"]
    User["Користувач (User)"]
    %% Audit Fix: External Actor Payment Gateway
    PaymentGateway["Платіжна система (Payment Gateway)"]:::external

    %% Generalization (Узагальнення)
    Student --> User
    Tutor --> User

    %% Прецеденти (Use Cases)
    UC1(("UC-1: Аутентифікація"))
    UC2(("UC-2: Бронювання заняття"))
    UC3(("UC-3: Перевірка доступності слоту"))
    UC4(("UC-4: Обробка оплати"))
    UC5(("UC-5: Налаштування розкладу"))

    %% Зв'язки Актор ↔ Use Case
    User --> UC1
    Student --> UC2
    Tutor --> UC5
    UC4 --> PaymentGateway

    %% Include та Extend
    UC2 .->|"«include»"| UC3
    %% Audit Fix: Extend relationship for asynchronous payment
    UC4 .->|"«extend»"| UC2

    classDef external fill:#f9f,stroke:#333,stroke-width:2px;
