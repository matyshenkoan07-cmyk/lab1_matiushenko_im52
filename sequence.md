# Діаграма послідовності (sequence.md)

Діаграма деталізує процес бронювання та оплати заняття (`UC-2` + `UC-4`) з урахуванням основного та альтернативних потоків.

```mermaid
sequenceDiagram
    autonumber
    actor Student as Учень (Student)
    participant UI as ClientUI
    participant GW as APIGateway
    participant BS as BookingService
    participant PS as PaymentService
    participant DB as Database

    %% Основний потік: Запит на бронювання
    Student->>UI: Обирає слот та тисне "Забронювати"
    UI->>GW: postBookingSlot(tutor_id, subject_id, start_time)
    GW->>BS: createBooking(student_id, tutor_id, subject_id, start_time)
    
    %% Перевірка наявності та створення чернетки
    BS->>DB: checkSlotAvailability(tutor_id, start_time)
    DB-->>BS: Слот вільний (true)
    
    BS->>DB: saveLessonDraft(status='Pending_Payment')
    DB-->>BS: booking_id
    
    %% Ініціалізація оплати
    BS->>PS: initiatePayment(booking_id, amount)
    PS-->>BS: посилання_на_оплату
    BS-->>GW: відповідьБронювання(booking_id, посилання_на_оплату)
    GW-->>UI: відповідьБронювання(booking_id, посилання_на_оплату)
    UI-->>Student: Перенаправлення на сторінку оплати

    %% Обробка результату платежу
    Student->>PS: Вводить реквізити та підтверджує платіж
    
    alt Успішна оплата
        PS->>BS: processPaymentCallback(booking_id, success=true)
        BS->>DB: updateLessonStatus(booking_id, status='Confirmed')
        DB-->>BS: Статус оновлено (true)
        BS-->>UI: notifyBookingConfirmed(booking_id)
        UI-->>Student: Відображення підтвердження бронювання
        
        opt Сповіщення користувача
            BS->>Student: Надсилання email/Push нагадування
        end

    else Помилка оплати або тайм-аут
        PS->>BS: processPaymentCallback(booking_id, success=false)
        BS->>DB: updateLessonStatus(booking_id, status='Available')
        DB-->>BS: StatusUpdated (true)
        BS-->>UI: notifyPaymentFailed(booking_id)
        UI-->>Student: Відображення помилки та повернення слоту в розклад
    end
