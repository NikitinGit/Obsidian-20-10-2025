
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

   >[!question]- переключиться на локальные настройки application.properties
 **IntelliJ IDEA** →  
`Run > Edit Configurations > Enveroumnt ...`  
> вписать SPRING_PROFILES_ACTIVE=local 
![[Pasted image 20260521204043.png]] 

>[!question]- во вкладке Commit куча лишних файлов в Unversioned Files (out/production/...)
>Это вывод компилятора IntelliJ (артефакты сборки), коммитить не нужно. Удалить директорию и добавить в `.gitignore`:
>```bash
>rm -rf out/
>echo '/out/' >> .gitignore
>```
>Чтобы не появлялись снова — `File > Project Structure > Project > Compiler output` (либо оставить `out/`, раз он теперь в `.gitignore`).


