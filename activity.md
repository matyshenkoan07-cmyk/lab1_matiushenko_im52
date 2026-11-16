# Activity-діаграма бізнес-процесу (activity.md)

Діаграма відображає потік дій (workflow) для сценарію бронювання та оплати заняття (`UC-2` + `UC-4`) з урахуванням 3 Swimlanes, розгалужень, смуг синхронізації (`FORK`/`JOIN`), події часу та обміну повідомленнями (`send`/`accept`).

```mermaid
graph TD
    subgraph Student ["Swimlane: Student (Учень)"]
        node_start(("● Початок"))
        act_select["Обирає слот для заняття"]
        act_pay["Вводить реквізити та здійснює платіж"]
        accept_notify[\"accept: Отримати сповіщення про запис"\]
        act_see_error["Бачить повідомлення про помилку / скасування"]
        node_end_success(("◉ Кінець (Успіх)"))
        node_end_fail(("◉ Кінець (Відмова)"))
    end

    subgraph BookingService ["Swimlane: BookingService (Сервіс бронювання)"]
        act_check_slot["Перевіряє доступність слоту"]
        dec_slot{"Слот вільний?"}
        act_save_draft["Фіксує чернетку бронювання (Pending_Payment)"]
        timer_timeout(("⏳ Минуло 15 хв без callback"))
        accept_callback[\"accept: Webhook про результат оплати"\]
        dec_pay{"Оплата успішна?"}
        act_set_avail["Звільняє слот (статус Available)"]
        fork_bar["▬▬▬ FORK (Паралельний старт) ▬▬▬"]
        join_bar["▬▬▬ JOIN (Синхронізація) ▬▬▬"]
        act_confirm["Переводить заняття у статус Confirmed"]
        send_notify[/"send: Push/Email сповіщення"/]
    end

    subgraph PaymentService ["Swimlane: PaymentService (Платіжний сервіс)"]
        act_gen_url["Формує платіжну сесію"]
        act_proc_pay["Проводить транзакцію"]
        send_callback[/"send: Webhook про результат оплати"/]
    end

    node_start --> act_select
    act_select --> act_check_slot
    act_check_slot --> dec_slot

    dec_slot -- Ні --> act_see_error
    act_see_error --> node_end_fail

    dec_slot -- Так --> act_save_draft
    act_save_draft --> act_gen_url
    act_gen_url --> act_pay

    act_save_draft --> timer_timeout
    timer_timeout --> act_set_avail

    act_pay --> act_proc_pay
    act_proc_pay --> send_callback
    send_callback -.-> accept_callback

    accept_callback --> dec_pay

    dec_pay -- Ні --> act_set_avail
    act_set_avail --> act_see_error

    dec_pay -- Так --> fork_bar

    fork_bar --> act_confirm
    fork_bar --> send_notify

    send_notify -.-> accept_notify

    act_confirm --> join_bar
    accept_notify --> join_bar

    join_bar --> node_end_success
```
