# Activity-діаграма бізнес-процесу (activity.md)

Діаграма відображає потік дій (workflow) для сценарію бронювання та оплати заняття (`UC-2` + `UC-4`) з урахуванням Swimlanes, розгалужень, паралельного виконання та обміну Webhook-повідомленнями.

```mermaid
graph TD
    %% --- SWIMLANES (SUBGRAPHS) ---

    subgraph Student ["Swimlane: Student (Учень)"]
        node_start(("● Початок"))
        act_select["Обирає слот для заняття"]
        act_pay["Вводить реквізити та здійснює платіж"]
        act_get_notify["Отримує сповіщення про запис"]
        act_see_error["Бачить повідомлення про помилку / зайнятість"]
        node_end_success(("◉ Кінець (Успіх)"))
        node_end_fail(("◉ Кінець (Відмова)"))
    end

    subgraph BookingService ["Swimlane: BookingService (Сервіс бронювання)"]
        act_check_slot["Перевіряє доступність слоту"]
        dec_slot{"Слот вільний?"}
        act_save_draft["Фіксує чернетку бронювання (Pending_Payment)"]
        act_recv_callback["Отримує Webhook про результат оплати"]
        dec_pay{"Оплата успішна чи тайм-аут?"}
        
        %% Fork & Join nodes (поки ще ромбами — виправимо у 2-му коміті)
        fork_node{"== FORK (Паралельний старт) =="}
        join_node{"== JOIN (Синхронізація) =="}
        
        act_confirm["Переводить заняття у статус Confirmed"]
        act_send_push["Надсилає Push/Email сповіщення"]
        act_set_avail["Звільняє слот (статус Available)"]
    end

    subgraph PaymentService ["Swimlane: PaymentService (Платіжний сервіс)"]
        act_gen_url["Формує платіжну сесію"]
        act_proc_pay["Проводить транзакцію"]
        act_send_callback["Надсилає Webhook про результат оплати"]
    end

    %% --- WORKFLOW CONNECTIONS ---

    node_start --> act_select
    act_select --> act_check_slot
    act_check_slot --> dec_slot

    %% Decision 1: Перевірка слоту
    dec_slot -- Ні --> act_see_error
    act_see_error --> node_end_fail

    dec_slot -- Так --> act_save_draft
    act_save_draft --> act_gen_url
    act_gen_url --> act_pay
    
    act_pay --> act_proc_pay
    act_proc_pay --> act_send_callback
    act_send_callback --> act_recv_callback

    %% Decision 2: Результат платежу
    act_recv_callback --> dec_pay

    %% Гілка Помилки
    dec_pay -- Ні / Timeout --> act_set_avail
    act_set_avail --> act_see_error

    %% Гілка Успішної оплати (FORK / JOIN)
    dec_pay -- Так --> fork_node
    
    fork_node --> act_confirm
    fork_node --> act_send_push
    
    act_send_push --> act_get_notify
    
    act_confirm --> join_node
    act_get_notify --> join_node
    
    join_node --> node_end_success
