
# IntelliJ IDEA

1. зайти в мавен и нажать clean compile и отключить тесты 
2. file/ invalidate cashes
3. В мавен гуи первая кнопка Sync all maven project

>[!question]- как собрать мавен проект со сбросом кеша
>mvn clean install -DskipTests -U 

>[!question]- Как сгенерировать конструктор
> ## **1. Горячие клавиши (Alt + Insert)**
> 1. **Установите курсор внутри класса**, где нужно создать конструктор.
>Нажмите: Windows/Linux:** `Alt + Insert` В появившемся меню выберите: **Constructor** → затем
>поля (можно выбрать несколько `Ctrl/Cmd + клик`).  
>Или **Lombok: AllArgsConstructor / NoArgsConstructor RequiredArgsConstructor**, если используется Lombok. 
>Через контекстное меню ПКМ Generte - constructor

>[!question]- поменять настройки кодировки на утф8 
>Settings - > Editor ->  File Encodings 

   >[!question]- увлечить стек в джава
java -Xss512k TestLinuxApplication - 512  кб
 mvn spring-boot:run -Dspring-boot.run.jvmArguments="-Xss512k"  
 **IntelliJ IDEA** →  
`Run > Edit Configurations > VM options`  
→ впиши `-Xss2m`
java -Xss1m TestLinuxApplication - с Мб

