C4Component
    title Component Diagram — TutorPlatform

    Person(user, "Student / Tutor", "Користувач платформи")

    System_Boundary(tutor_platform, "TutorPlatform") {
        Component(web, "WebClient", "SPA / Frontend", "Клієнтський веб-інтерфейс")
        Component(gateway, "APIGateway", "REST API", "Єдина точка входу, маршрутизація та перевірка токенів")
        
        Component(auth, "AuthService", "Module", "Реєстрація, автентифікація та керування сесіями")
        Component(catalog, "CatalogService", "Module", "Пошук та фільтрація репетиторів і предметів")
        Component(booking, "BookingService", "Module", "Бронювання слотів та керування життєвим циклом уроку")
        Component(payment, "PaymentService", "Module", "Обробка платежів та прийом webhook-підтверджень")
        Component(notification, "NotificationService", "Module", "Формування та відправка сповіщень")
        
        Component(dal, "DataAccessLayer", "Repository", "Інкапсуляція доступу до даних (IDataRepository)")
        ComponentDb(db, "PostgreSQL", "Relational DB", "Зберігання сутностей Student, Tutor, Subject, Lesson")
    }

    System_Ext(oauth, "OAuthProvider", "Зовнішній провайдер автентифікації (Google)")
    System_Ext(pay_gw, "PaymentGateway", "Зовнішній платіжний шлюз")
    System_Ext(notif_prov, "NotificationProvider", "Зовнішній сервіс Email / Push розсилок")

    Rel(user, web, "Використовує", "HTTPS")
    Rel(web, gateway, "Викликає API", "REST / JSON")

    Rel(gateway, auth, "Використовує", "IAuthAPI")
    Rel(gateway, catalog, "Використовує", "ICatalogAPI")
    Rel(gateway, booking, "Використовує", "IBookingAPI")

    Rel(booking, payment, "Ініціює оплату", "IPaymentAPI")
    Rel(booking, notification, "Запитує сповіщення", "INotificationSender")

    Rel(auth, dal, "Читає/записує дані", "IDataRepository")
    Rel(catalog, dal, "Читає дані", "IDataRepository")
    Rel(booking, dal, "Читає/записує дані", "IDataRepository")
    Rel(payment, dal, "Оновлює статус оплати", "IDataRepository")
    Rel(dal, db, "SQL-запити", "TCP / JDBC")

    Rel(auth, oauth, "Автентифікує через", "OAuth 2.0 / OIDC")
    Rel(payment, pay_gw, "Створює платіж", "HTTPS / REST")
    Rel(pay_gw, payment, "Надсилає статус оплати", "Webhook / IPaymentWebhook")
    Rel(notification, notif_prov, "Відправляє повідомлення", "SMTP / Push API")
