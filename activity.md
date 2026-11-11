# Activity-діаграма бізнес-процесу (activity.md)

Діаграма відображає потік дій (workflow) для сценарію бронювання та оплати заняття (`UC-2` + `UC-4`) з урахуванням Swimlanes, розгалужень, паралельного виконання та обміну Webhook-повідомленнями.

```mermaid
graph TD
    %% --- SWIMLANES (SUBGRAPHS) ---

    subgraph Student ["Swimlane: Student (Учень)"]
        node_start(("● Початок"))
        act_select["Обирає слот та тисне 'Забронювати'"]
        act_pay["Вводить реквізити та здійснює платіж"]
        act_get_notify["Отримує email/push сповіщення"]
        act_see_confirm["Переглядає підтвердження бронювання"]
        act_see_error["Бачить повідомлення про помилку"]
        node_end_success(("◉ Кінець (Успіх)"))
        node_end_fail(("◉ Кінець (Помилка)"))
    end

    subgraph ClientUI ["Swimlane: ClientUI (Фронтенд)"]
        act_post_slot["Надсилає postBookingSlot(...)"]
        act_redirect["Перенаправляє на сторінку оплати"]
        act_show_success["Відображає успішне бронювання"]
        act_show_fail["Відображає помилку оплати / зайнятості"]
    end

    subgraph BookingService ["Swimlane: BookingService (Бекенд)"]
        act_create_req["Отримує createBooking(...)"]
        act_check_req["Запитує checkSlotAvailability(...)"]
        act_save_draft["Викликає saveLessonDraft(status='Pending_Payment')"]
        act_init_pay["Викликає initiatePayment(...)"]
        act_recv_callback["Отримує processPaymentCallback(...)"]
        
        %% Fork & Join nodes
        fork_node{"== FORK (Паралельний старт) =="}
        join_node{"== JOIN (Синхронізація) =="}
        
        act_send_push["Надсилає push/email сповіщення"]
        act_set_avail["Викликає updateLessonStatus(status='Available')"]
    end

    subgraph PaymentService ["Swimlane: PaymentService (Платіжний сервіс)"]
        act_gen_url["Генерує посилання на оплату"]
        act_proc_pay["Проводить транзакцію"]
        act_send_callback["Надсилає Webhook callback"]
    end

    subgraph Database ["Swimlane: Database (База даних)"]
        act_db_check["Перевіряє доступність у таблиці Lesson"]
        act_db_draft["Зберігає чернетку уроку"]
        act_db_confirm["Оновлює статус уроку на 'Confirmed'"]
        act_db_avail["Оновлює статус уроку на 'Available'"]
    end

    %% --- WORKFLOW CONNECTIONS ---

    node_start --> act_select
    act_select --> act_post_slot
    act_post_slot --> act_create_req
    act_create_req --> act_check_req
    act_check_req --> act_db_check

    %% Decision 1: Перевірка слоту
    act_db_check --> dec_slot{"Слот вільний?"}
    
    dec_slot -- Ні --> act_show_fail
    act_show_fail --> act_see_error
    act_see_error --> node_end_fail

    dec_slot -- Так --> act_save_draft
    act_save_draft --> act_db_draft
    act_db_draft --> act_init_pay
    act_init_pay --> act_gen_url
    act_gen_url --> act_redirect
    act_redirect --> act_pay
    
    act_pay --> act_proc_pay
    act_proc_pay --> act_send_callback
    act_send_callback --> act_recv_callback

    %% Decision 2: Результат платежу та Тайм-аут
    act_recv_callback --> dec_pay{"Оплата успішна чи тайм-аут?"}

    %% Гілка Помилки / Тайм-ауту
    dec_pay -- Ні / Timeout --> act_set_avail
    act_set_avail --> act_db_avail
    act_db_avail --> act_show_fail

    %% Гілка Успішної оплати (FORK / JOIN)
    dec_pay -- Так --> fork_node
    
    fork_node --> act_db_confirm
    fork_node --> act_send_push
    
    act_send_push --> act_get_notify
    
    act_db_confirm --> act_confirm_resp["BookingService отримує підтвердження БД"]
    act_confirm_resp --> join_node
    act_get_notify --> join_node
    
    %% Фінальне підключення успішного завершення
    join_node --> act_show_success
    act_show_success --> act_see_confirm
    act_see_confirm --> node_end_success
