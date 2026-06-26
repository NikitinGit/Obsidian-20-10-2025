# Email (mail) в проекте StrikerStat

Отправка писем: SMTP через Spring `JavaMailSender`. По умолчанию — Gmail SMTP. В отличие от SMS, **никакой модерации шаблонов нет** — текст письма произвольный.

>[!question]- Что такое SMTP
> **SMTP = Simple Mail Transfer Protocol** — простой протокол передачи почты. Стандартный протокол **отправки** писем: клиент (наше приложение) подключается к почтовому серверу (smtp.gmail.com:465), аутентифицируется и сдаёт письмо на доставку. Работает поверх TCP, обычно через TLS (порт 465 = SSL, 587 = STARTTLS). SMTP отвечает только за **отправку/пересылку**; для чтения почты клиентом используются другие протоколы — IMAP/POP3. В коде SMTP-клиент скрыт за Spring `JavaMailSender` (реализация `jakarta.mail`).

## Обзор и где это в коде

>[!question]- Где в проекте отправляется email
> Один низкоуровневый отправитель и несколько доменных сервисов поверх него:
> - `service/auth/AuthMailService` — **основной** отправитель (SMTP-клиент): подтверждение email, восстановление доступа, тестовое письмо, произвольное HTML-письмо.
> - `service/open_events/MasterClassUnregisteredBidEmailService` — письма по заявкам на мастер-классы.
> - `service/account/AccountAppealService` — письма по апелляциям аккаунта.
> - `service/challenge/ChallengeNotificationWorker` — шлёт email-уведомления о вызовах (через `AuthMailService.sendHtmlEmail`).
> Реально в SMTP ходит только `AuthMailService` (через `JavaMailSender`); остальные зовут его.

>[!question]- Какая зависимость в pom.xml отвечает за почту
> **`spring-boot-starter-mail`** — он тащит `JavaMailSender` и `jakarta.mail` (SMTP-транспорт, `MimeMessage`, `MimeMessageHelper`). Отдельной сторонней библиотеки нет — это стандартный Spring Mail поверх SMTP. (Сравни с SMS: там вообще нет SDK, голый REST через `RestTemplate` — см. [[SMS]].)

## AuthMailService

>[!question]- Что умеет AuthMailService — основные методы
> - `sendConfirmEmail(User)` / `sendConfirmEmail(recipient, hash, acstatus)` — письмо «подтвердите email» со ссылкой `…/verify-email?hash=…`.
> - `sendRestoreAccessEmail(User, pendingAction)` — восстановление доступа `…/restore-access-by-email?code=…`.
> - `sendTestConfirmEmail(recipient)` — тестовое письмо (для админки).
> - `sendHtmlEmail(recipient, subject, html)` — **универсальная** отправка произвольного HTML (добавлено под уведомления о вызовах).
> - приватный `send(recipient, subject, html, link)` — всё сводится сюда: `MimeMessageHelper.setText(html, true)` (HTML), `setFrom`, `setTo`, `setSubject`, `mailSender.send(...)`.
> Возвращает `DeliveryResult(boolean success, boolean configured, String message, String link)`.

>[!question]- Когда письмо вообще отправляется (isConfigured)
> `isConfigured()` → `isMailConfigured()` = заданы **все три**: `spring.mail.host`, `spring.mail.username`, `spring.mail.password`. Если что-то пусто — `send` НЕ ходит в SMTP, возвращает `DeliveryResult(success=false, configured=false, …)` и пишет в лог ссылку (чтобы в деве можно было перейти руками). Воркер уведомлений при `!isConfigured` откладывает запись (доставит, когда настроят).

>[!question]- Что происходит при ошибке отправки
> `send` ловит `MailException | MessagingException`, логирует `ERROR` и возвращает `DeliveryResult(success=false, configured=true, message=…)`. Исключение наружу не пробрасывается — вызывающий код смотрит на `success`.

## Конфигурация SMTP

>[!question]- Как настроен SMTP (application.properties)
> Дефолт — **Gmail** (можно переопределить env-переменными):
> ```properties
> spring.mail.host=smtp.gmail.com        # ${PRIMARY_MAIL_HOST:${MAIL_HOST:${GMAIL_HOST:…}}}
> spring.mail.port=465
> spring.mail.username=strikerstat@gmail.com
> spring.mail.password=<app password>    # пароль приложения Gmail (НЕ обычный пароль)
> spring.mail.protocol=smtp
> spring.mail.properties.mail.smtp.auth=true
> spring.mail.properties.mail.smtp.ssl.enable=true      # implicit SSL (порт 465)
> spring.mail.properties.mail.smtp.starttls.enable=false
> app.mail.from-address=strikerstat@gmail.com
> app.mail.from-name=Сайт Strikerstat
> app.auth.public-base-url=https://strikerstat.com      # база для ссылок в письмах
> ```
> Все значения переопределяемы через env (`PRIMARY_MAIL_*` / `MAIL_*` / `GMAIL_*`).

>[!question]- Порт 465 vs 587 — почему важно
> - **465 = implicit SSL** → нужен `mail.smtp.ssl.enable=true` и `starttls.enable=false` (как сейчас).
> - **587 = STARTTLS** → наоборот, `starttls.enable=true`, `ssl.enable=false`.
> Если порт и SSL-настройки **не совпадают** — TLS-рукопожатие рвётся (`SSLHandshakeException: Remote host terminated the handshake`). Это первая причина «письма не уходят» при смене SMTP.

## Email-уведомления о вызовах (challenge)

>[!question]- Как идут email-уведомления о вызовах
> Так же, как SMS, через outbox: `ChallengeNotificationService.notify` пишет строку в `event_challenge_notifications` (`channel=EMAIL`, `is_sent=false`), а `ChallengeNotificationWorker` (`@Scheduled`) шлёт её через `AuthMailService.sendHtmlEmail`. Тема/текст — i18n `challenge.notification.email.subject` / `challenge.notification.email.message` (HTML со ссылкой `<a href="{0}">`, `{0}` = диплинк в ЛК).

>[!question]- Чем email-уведомление отличается от SMS-уведомления о вызове
> - **email шлётся на ВСЕ события** вызова (PENDING/ACCEPTED/DECLINED/WITHDRAWN) — текст общий («есть обновление по вашему вызову»).
> - **SMS — только на PENDING** (новый вызов), и текст конкретный («Вам бросили вызов…»). См. [[SMS]].
> - email **со ссылкой** в ЛК; SMS — **без ссылки** (под шаблон провайдера).
> - email-ключи заполнены в `messages.properties` И `messages_en.properties`; SMS — только ru.

>[!question]- На каком языке уходит challenge-email
> Текст резолвится через `MessageService.get` → `LocaleContextHolder.getLocale()`. Письмо формируется в **фоновом воркере без HTTP-запроса** → локаль = дефолт JVM, а не язык получателя. en-ключи заполнены, так что при дефолтной локали сервера `en` письмо уйдёт по-английски. (Для SMS этим управляли через отсутствие en-ключа; для email оставили оба.)

>[!question]- Как воркер выбирает адрес получателя
> `resolveEmail(user)` — первый из `user.login` / `user.normalizedEmail` / `user.newEmail`, содержащий `@`. Если ни одного валидного — снимаем запись без доставки.

## Ошибки и доставка

>[!question]- Что значит ошибка `550 non-local recipient verification failed`
> Это ответ SMTP-сервера: **адрес получателя невалиден/недоставляем** (`SendFailedException: Invalid Addresses`). Постоянная ошибка — ретраить бесполезно. На препроде ловили её на тестовых адресах вида `f36990@example.com`.

>[!question]- Почему лог засорялся стектрейсами при отправке challenge-email (и как починили)
> Раньше воркер при неудаче оставлял запись `is_sent=false` и пытался снова **каждый тик** → `AuthMailService` логировал полный стектрейс по каждому битому адресу бесконечно. Починили: внешнее уведомление **best-effort** — после реальной попытки запись снимается из очереди **без повторов** (успех → `is_sent=true,sent_at=now`; неудача → снять, `sent_at=null`). Теперь ошибка логируется один раз на письмо, надёжный канал — in-app.

>[!question]- Как протестировать отправку письма
> Через админку: `POST /api/admin-panel/technical-tests/mail/registration` (тестовое письмо), `GET /…/status` показывает `mailConfigured`, `mailFrom`, `publicBaseUrl`. Доступ — пользователь с `users.acstatus='orgAdmin'` (детали — в [[SMS]] раздел «Админка»).

## Чеклист «письмо не уходит»

>[!question]- Письмо не приходит — что проверить по порядку
> 1. **`SSLHandshakeException` / `Remote host terminated the handshake`** → несоответствие порт↔SSL (465=SSL, 587=STARTTLS) или файрвол блокирует порт.
> 2. **`550 …recipient verification failed` / `Invalid Addresses`** → невалидный адрес получателя, доставки не будет (best-effort снимет из очереди).
> 3. **`configured=false` в логе** → не заданы `spring.mail.host/username/password`.
> 4. **Gmail-specific**: нужен **App Password** (не обычный пароль), включён доступ; иначе `AuthenticationFailedException`.
> 5. Письма нет вовсе и ошибок нет → проверь, что воркер включён (`jobs.challenge-notifications.enabled`), не сработал ли лимит/«спал»/опт-аут (та же логика, что у SMS).

## Связанное
- [[SMS]] — второй канал внешних уведомлений; общий воркер `ChallengeNotificationWorker`, та же очередь и best-effort-логика.
- [[Proxy Object]] — воркер читает `challenge.event` вне транзакции (`JOIN FETCH`).