
https://www.google.com/search?sourceid=chrome&ccb=1&q=%D0%B2%D1%81%D0%B5+%D1%81%D0%B5%D1%82%D0%B5%D0%B2%D1%8B%D0%B5+%D0%BF%D1%80%D0%BE%D1%82%D0%BE%D0%BA%D0%BE%D0%BB%D1%8B+%28TCP%2C+UDP%2C+RUDP+...%29+%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%B0%D0%B5%D1%82+%D0%BD%D0%B0+%D0%BE%D1%81%D0%BD%D0%BE%D0%B2%D0%B5+%D1%81%D0%BE%D0%BA%D0%B5%D1%82%D0%BE%D0%B2+%284+%D1%87%D0%B8%D1%81%D0%B5%D0%BB+-+%D0%B0%D0%B9%D0%BF%D0%B8+%D0%B0%D0%B4%D1%80%D0%B5%D1%81+%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80%D0%B0%2C+%D0%BF%D0%BE%D1%80%D1%82+%D1%81%D0%B5%D0%B2%D0%B5%D1%80%D0%B0%2C+%D0%B0%D0%B9%D0%BF%D0%B8+%D0%BA%D0%BB%D0%B8%D0%B5%D0%BD%D1%82%D0%B0%2C+%D0%BF%D0%BE%D1%80%D1%82+%D0%BA%D0%BB%D0%B8%D0%B5%D0%BD%D1%82%D0%B0%29+%3F&sca_esv=a58e26e22b62a0d5&sxsrf=APpeQntksi_3Qy3xJRJUpppibyQmCUkzcA%3A1783670514830&source=chrome.ob&fbs=ABfTbFXwfYg-AsVmc75aAkBispm6k7RIMh52cWHUntfypj3niv0vpWpy8F5Hmr7ocaLGY8xJhfulKPXEEekLZl0_KQ3ifU6t75W-_0fVRA-YNxtw5LIxHYJubfwQQUan3_SDFyQlFdmipnDmHrW9hhJK3livnylzojg6CcHX-9ScBlZ6TDnvQw9NkKrt5Di_1pbcrm3PTlDieGo1qvIOxNMIGpnEMfiUFDxNr9rMSdv8pLAZ2bqTDb-BO-evtgyukLg0AhAFGM7SajrE0XPtjxESZ5dN2Atx1g&aep=1&ntc=1&sa=X&ved=2ahUKEwiV4bq70seVAxXClYkEHWZSOEIQ2J8OegQIDBAD&biw=2048&bih=1056.800048828125&dpr=1.25&cs=1&hl=en-US&mstk=AUtExfA6pQC5UR7j_rxD05TeUONKfxNqURKKJH8atxopuZAkm5a7pxNMU6oGDFPJbxGF-frscxns_WbS3QTpyhc5WWjoAb3TZavpHL0fyn-eZF_ykw3hWIITS_CSAHziVDSsfU8Jf1W49zEiqK5vzGUbS1X5GgfYlVsgxUWMgkbi5OZmv7O3ncVTXsNMQmPy_TF_i9JBjCZ979CI51boZX7fIiZHd6Z4rPO0hGTxmGYP0bpSmZgCZMdSTXZH0oXeC-NnTtGPkfy_OKvWDg&csuir=1&mtid=wqdQaq6lH9HcptQPw7fAsAM&udm=50&cru=1

1. [ ] пример сприн приложения , как там с потоками все реализовано - Spring MVC 

>[!question]- первый калаут - база про тсп, сокет и место его в OSI
> **Все ли сетевые протоколы работают на основе сокета (4 числа: IP сервера, порт сервера, IP клиента, порт клиента)?**
>
> Нет. Это верно только для **транспортного уровня** стека TCP/IP — TCP, UDP, RUDP, SCTP используют сокет-пару (точнее 5-tuple с учётом протокола), чтобы ОС понимала, какому процессу отдать пакет.
>
> Ниже транспортного уровня понятия "порт" не существует:
> - **Сетевой уровень (IP, ICMP, IPsec)** — работает только по паре IP-адресов. У ICMP (ping) нет портов, есть Type/Code.
> - **Канальный уровень (Ethernet, Wi-Fi)** — работает по паре MAC-адресов, вообще без IP и портов.
> - **ARP / OSPF / BGP** — служебные протоколы маршрутизации, тоже без портов.
>
> | Уровень | Примеры | Идентификация | Есть порты? |
> |---|---|---|---|
> | Транспортный | TCP, UDP, RUDP, SCTP | сокет-пара (IP+порт клиента, IP+порт сервера) | Да |
> | Сетевой | IP, ICMP, IPsec | пара IP-адресов | Нет |
> | Канальный | Ethernet, Wi-Fi | пара MAC-адресов | Нет |
>
> Отдельно: **Unix Domain Sockets** — тоже "сокет" в терминологии программирования, но это IPC внутри одной машины, без IP и портов вообще (адресация через путь к файлу, например `/var/run/mysql.sock`).

>[!question]- почему стек называется TCP/IP, если TCP и UDP в корне разные, а IP вообще сетевой уровень
> Историческая причина: изначально (70-е, Серф и Кан) это был один монолитный протокол TCP, который потом разделили на:
> - **IP** — **Internet Protocol** (сетевой уровень - примеры IPv4, IPv6) доставка пакетов от точки А до точки Б (сетевой уровень)
> - **TCP** — надёжность, порядок, контроль ошибок (транспортный уровень)
>
> Название закрепилось по двум главным протоколам, UDP в название не попал, чтобы не звучало громоздко.
>
> TCP и UDP при этом не чужие друг другу — общий фундамент:
> - оба **упаковываются внутрь IP-пакета** (роутерам всё равно, TCP там или UDP)
> - оба используют **одинаковое пространство портов** (0–65535)
> - у обоих похожая структура заголовка (контрольные суммы и т.д.)
>
> Разница только в том, что TCP добавляет поверх IP надёжность и порядок доставки, а UDP — нет.

>[!question]- обязателен ли порядок пакетов в TCP, можно ли в приложении обработать их не по порядку
> **Нет, порядок в классическом TCP обойти нельзя** — это зашито в протокол на уровне ядра ОС. Приложение (Java и любое другое) физически не может получить пакет №2, пока не пришёл потерянный пакет №1. Это называется **Head-of-Line Blocking (HoL)**.
>
> Как это работает под капотом:
> 1. Сервер получает пакеты №2, №3 — пакет №1 потерялся.
> 2. ОС по Sequence Number видит дыру, складывает №2 и №3 в буфер ожидания, запрашивает у клиента повтор №1.
> 3. Java-код на `inputStream.read()` / `readLine()` в этот момент **блокируется** (спит) — ОС не отдаёт уже пришедшие данные, пока цепочка не соберётся по порядку.
> 4. Когда №1 доходит — ОС склеивает 1,2,3 и только тогда будит приложение.
>
> Для Java-приложения TCP выглядит как непрерывный, идеально упорядоченный поток байт — оно в принципе не знает, что были потери.
>
> **Если нужно обрабатывать пакеты не по порядку** — TCP не подходит, нужен UDP (с ручной сборкой) или **QUIC (HTTP/3)**, который поверх UDP делает мультиплексирование потоков: если потерялся пакет одного потока (например, одной картинки), другие потоки продолжают доставляться, блокируется только пострадавший поток.

>[!question]- получается транспортный уровень полностью во власти ОС, приложение ничего не решает?
> Не совсем — приложение может как настраивать штатный TCP от ОС, так и полностью забрать транспортный уровень себе.
>
> **1. Настройка сокета (Java `Socket`/`Optons`)** — приложение отдаёт ОС конкретные команды по поведению TCP:
>
> ```java
> Socket socket = new Socket();
>
> // Отключить алгоритм Нагла — данные уходят сразу, без накопления в буфер (важно для игр/бирж)
> socket.setTcpNoDelay(true);
>
> // Явно задать размер приёмного буфера (окно TCP) для этого сокета
> socket.setReceiveBufferSize(128 * 1024);
>
> // Keep-Alive — ОС сама шлёт пустые проверочные пакеты, чтобы роутеры не рвали простаивающее соединение
> socket.setKeepAlive(true);
>
> // При закрытии — не ждать TIME_WAIT (~2 мин), рвать соединение сразу через RST
> socket.setSoLinger(true, 0);
>
> // Если чтение зависло (например ждём потерянный пакет) дольше 5 сек — бросить SocketTimeoutException,
> // а не ждать вечно
> socket.setSoTimeout(5000);
>
> socket.connect(new InetSocketAddress("example.com", 80), 2000);
> ```
>
> **2. Полный отказ от TCP от ОС:**
> - **UDP (`DatagramSocket`)** — ОС отдаёт разрозненные пакеты как есть, а нумерацию, повторные запросы, сборку — приложение реализует само (так по сути устроен QUIC/HTTP3, Netty-протоколы поверх UDP).
> - **Raw Sockets** — приложение получает "сырые" биты вплоть до заголовков IP/TCP и само их парсит и считает контрольные суммы (так работают Nmap, Wireshark). В Java такое сложно сделать напрямую из соображений безопасности, обычно это C/C++.

>[!question]- если отключить алгоритм Нагла (`setTcpNoDelay(true)`), перестаёт ли работать механизм обнаружения потерянных пакетов (№452 и т.п.)?
> Нет, это два независимых механизма TCP, друг от друга никак не зависят.
>
> - **Алгоритм Нагла** — политика **только на стороне отправителя**: копить мелкие куски данных в буфере и отправлять их одним крупным пакетом, или слать каждый `write()`/`send()` сразу отдельным пакетом. Это вопрос "когда нажать на курок", а не "что происходит с пакетом дальше".
> - **Обнаружение потери и повторная отправка** (Seq/Ack, ретрансмит, буфер на приёме для сборки строго по порядку — см. калаут про пакет №452) — фундаментальное, всегда включённое свойство самого протокола TCP на уровне ядра ОС. У него нет переключателя, и `setTcpNoDelay` его не касается.
>
> Пока используется `Socket`/`ServerSocket` (то есть TCP), гарантия "все байты долетят и будут отданы приложению строго по порядку" держится железно независимо от настроек Нагла:
> - если один из (теперь более мелких и частых) пакетов потеряется — Seq/Ack-механизм всё так же его обнаружит и переотправит;
> - получатель всё так же не отдаст приложению данные не по порядку — `readLine()`/`read()` так же заблокируется на дыре, как и раньше.
>
> Разница от `setTcpNoDelay(true)` — только в *количестве и размере* пакетов, летящих по сети (и, как следствие, в задержке отправки), а не в надёжности или порядке доставки.

>[!question]- как в Java реально отправлять/принимать данные через сокет
> Для Java-приложения TCP-соединение выглядит как непрерывный поток байт (`InputStream`/`OutputStream`), а не как отдельные "пакеты" — их нарезкой и сборкой занимается ОС.
>
> ```java
> Socket socket = new Socket();
> socket.setTcpNoDelay(true);
> socket.setSoTimeout(5000);
> socket.connect(new InetSocketAddress("example.com", 80), 2000);
>
> // ОТПРАВКА
> OutputStream output = socket.getOutputStream();
> String httpRequest = "GET / HTTP/1.1\r\nHost: example.com\r\nConnection: close\r\n\r\n";
> output.write(httpRequest.getBytes(StandardCharsets.UTF_8));
> output.flush(); // выталкиваем байты из буфера Java в ОС
>
> // ПРИЁМ
> BufferedReader reader = new BufferedReader(
>         new InputStreamReader(socket.getInputStream(), StandardCharsets.UTF_8));
> String line;
> while ((line = reader.readLine()) != null) {
>     System.out.println(line);
> }
> ```
>
> Нюансы:
> - `write()` не гарантирует, что весь текст уйдёт одним TCP-пакетом — ОС сама режет на куски (~1460 байт под MTU).
> - `readLine()`/`read()` блокируются, пока ОС не соберёт достаточно байт для ответа Java-коду.
> - Если во время чтения потеряется пакет — выполнение зависнет прямо внутри `readLine()`, пока не сработает `SoTimeout`.

>[!question]- как сделать полностью асинхронный обмен: сервер и клиент каждый в отдельных потоках на приём/отправку
> TCP-сокет **full duplex** — входящий и исходящий поток независимы, поэтому чтение и запись безопасно разносятся по разным потокам без взаимной блокировки.
>
> **Сервер** — на каждого клиента два потока: один только `readLine()` из сокета, другой только пишет (например, из консоли):
>
> ```java
> Socket clientSocket = serverSocket.accept();
>
> Thread readerThread = new Thread(() -> {
>     try (BufferedReader in = new BufferedReader(
>             new InputStreamReader(clientSocket.getInputStream(), StandardCharsets.UTF_8))) {
>         String message;
>         while ((message = in.readLine()) != null) {
>             System.out.println("[СЕРВЕР ПОЛУЧИЛ]: " + message);
>         }
>     } catch (IOException e) { /* ... */ }
> });
>
> Thread writerThread = new Thread(() -> {
>     try (PrintWriter out = new PrintWriter(
>             new OutputStreamWriter(clientSocket.getOutputStream(), StandardCharsets.UTF_8), true);
>          Scanner scanner = new Scanner(System.in)) {
>         while (!clientSocket.isClosed()) {
>             if (scanner.hasNextLine()) out.println(scanner.nextLine());
>         }
>     } catch (IOException e) { /* ... */ }
> });
>
> readerThread.start();
> writerThread.start();
> ```
>
> **Клиент (Swing GUI)** — отправка происходит в GUI-потоке (по клику/Enter), а чтение крутится в отдельном фоновом потоке, чтобы UI не зависал; обновление `chatArea` из фонового потока — только через `SwingUtilities.invokeLater`:
>
> ```java
> // Фоновый поток — только чтение
> Thread networkReader = new Thread(() -> {
>     try (BufferedReader in = new BufferedReader(
>             new InputStreamReader(socket.getInputStream(), StandardCharsets.UTF_8))) {
>         String message;
>         while ((message = in.readLine()) != null) {
>             String finalMessage = message;
>             SwingUtilities.invokeLater(() -> chatArea.append("[СЕРВЕР]: " + finalMessage + "\n"));
>         }
>     } catch (IOException e) { /* соединение потеряно */ }
> });
> networkReader.start();
>
> // GUI-поток — только отправка, вызывается по клику кнопки / Enter
> private void sendMessage() {
>     String text = inputField.getText().trim();
>     if (!text.isEmpty() && out != null) {
>         out.println(text);
>         chatArea.append("[ВЫ]: " + text + "\n");
>         inputField.setText("");
>     }
> }
> ```
>
> ⚠️ **Важное ограничение учебного примера**: `serverSocket.accept()` вызывается **один раз** — сервер обслуживает только первого подключившегося клиента. Для нескольких клиентов нужен цикл `while (true) { accept(); new Thread(...).start(); }`.

>[!question]- практика: "клиент получает сообщения от сервера, но сам отправить не может" — в чём было дело
> Диагностика показала, что **сам код рабочий** — `sendMessage()`/`PrintWriter.println()` реально доставляет сообщение серверу (проверено и напрямую сырым сокетом, и через реальный класс `AsyncGuiClient` вызовом `sendMessage()` — сервер печатал `[СЕРВЕР ПОЛУЧИЛ]`).
>
> Реальные причины подобных "не работает":
> 1. **Оба потока сервера пишут в один `System.out`/читают `System.in`** без синхронизации — строка `[СЕРВЕР ПОЛУЧИЛ]` реально приходит, но теряется в потоке повторяющихся `[СЕРВЕР ОТПРАВИТЬ]: ` от писательского потока, легко пропустить глазами.
> 2. **`accept()` вызывается один раз** — если сервер или клиент перезапускали без явной остановки старого процесса (Stop в IntelliJ), можно словить `BindException: Address already in use`, либо (что коварнее) клиент тихо подключается к **старому** зависшему процессу сервера, консоль которого уже не видна/не активна. Проверка: `ss -ltnp | grep 8081` — кто реально слушает порт.
> 3. В конкретном случае причина была совсем банальной: **текст пытались ввести не в то поле** — оно физически не было видно, потому что сливалось с фоном окна (см. следующий калаут).

>[!question]- практика: поле ввода было невидимым (сливалось с фоном окна) — почему и как починили
> Симптом: `JTextField` визуально неотличим от фона окна — текст, набранный туда, никак не виден, создаётся ложное впечатление "клиент не может отправлять".
>
> Первая попытка — явно задать цвета компоненту:
> ```java
> inputField.setBackground(Color.WHITE);
> inputField.setForeground(Color.BLACK);
> ```
> Не помогло — поле осталось тёмным, как и остальное окно.
>
> **Причина**: на Linux Swing по умолчанию может подхватывать **GTK Look&Feel**, которая рисует компоненты через нативный GTK-рендер и в ряде тем **игнорирует явные `setBackground()`/`setForeground()`** у `JTextField` — известная особенность/баг `GTKLookAndFeel`.
>
> **Рабочее решение** — принудительно поставить кроссплатформенный (Metal) L&F перед созданием окна, тогда заданные цвета применяются гарантированно:
> ```java
> public static void main(String[] args) {
>     try {
>         UIManager.setLookAndFeel(UIManager.getCrossPlatformLookAndFeelClassName());
>     } catch (Exception ignored) {
>     }
>     SwingUtilities.invokeLater(() -> new AsyncGuiClient().setVisible(true));
> }
> ```
> После этого поле ввода стало белым с чёрным текстом независимо от системной темы — подтверждено визуально.