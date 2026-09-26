# ELK Stack (или Elastic Stack)
https://tproger.ru/articles/primenenie-elk-steka-dlya-logirovaniya-i-monitoringa-prilozhenij 
1. [ ] установи на впс по инструкции в https://gitinsky.com/elkstack#:~:text=%D1%87%D0%B5%D1%80%D0%B5%D0%B7%20%D0%BC%D0%BD%D0%BE%D0%B3%D0%BE%D0%B4%D0%BE%D0%BA%D1%83%D0%BC%D0%B5%D0%BD%D1%82%D0%BD%D1%8B%D0%B5%20API-,%D0%A7%D1%82%D0%BE%20%D1%82%D0%B0%D0%BA%D0%BE%D0%B5%20Logstash,%D1%83%D0%BF%D1%80%D0%BE%D1%89%D0%B0%D0%B5%D1%82%D1%81%D1%8F%20%D0%B4%D0%BE%D1%81%D1%82%D1%83%D0%BF%20%D0%BA%20%D1%80%D0%B0%D0%B7%D0%BD%D1%8B%D0%BC%20%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D0%BC. 
2. [x] разобраться частью какого стэка является графана в проекте страйкер стат - прометеус



>[!question]- ELK Stack (или Elastic Stack) 
>это популярный набор инструментов для сбора, анализа и визуализации данных, особенно логов и метрик, состоящий из **E**lasticsearch (поисковый движок), **L**ogstash (конвейер для обработки данных) и **K**ibana (интерфейс для визуализации). 
>- **Дистрибутив jvm:** Обычно используется **OpenJDK**. Начиная с версии 7.x, Elasticsearch поставляется со встроенной (bundled) JVM, которую разработчики настоятельно рекомендуют использовать для стабильности.
>юз кейс - мониторинг приложений, девопс 

>[!question]- **Elasticsearch**
> сердце стека. Это поисковая и аналитическая система, где хранятся и индексируются все данные. Она позволяет очень быстро искать информацию в огромных массивах. 
> Это распределенная RESTful-система на основе JSON, которая сочетает в себе функции NoSQL-базы данных (сохраняет все собранные данные), поисковой системы и аналитической системы.  
> Написан на **Java**. В его основе лежит библиотека поискового движка Apache Lucene, также созданная на Java. 

>[!question]- **Logstash**
>инструмент для обработки данных. Он собирает данные из разных источников одновременно, фильтрует их, преобразует и отправляет в Elasticsearch.
>Написан на **JRuby** (реализация языка Ruby, работающая поверх JVM). Это позволяет ему использовать как библиотеки Ruby, так и Java-пакеты. 

>[!question]- **Kibana**
> графический интерфейс. Это окно в мир ваших данных, где вы строите графики, диаграммы и дашборды на основе того, что лежит в Elasticsearch. Написана на **TypeScript** и **JavaScript** (Node.js).

>[!question]- **Beats**
>  Эти легковесные агенты написаны на **Go (Golang)**. Их специально сделали не на Java, чтобы они потребляли минимум системных ресурсов на конечных серверах. 


![[Pasted image 20260109165447.png]]


1. ![[Pasted image 20260926160355.png]]
2. ![[Pasted image 20260926160414.png]]
3. ![[Pasted image 20260926160640.png]]
4. ![[Pasted image 20260926160721.png]]
5. ![[Pasted image 20260926160754.png]]
6. ![[Pasted image 20260926160923.png]]
7. ![[Pasted image 20260926161135.png]]
8. ![[Pasted image 20260926161242.png]]
9. ![[Pasted image 20260926161323.png]]
10. ![[Pasted image 20260926161341.png]]
11. ![[Pasted image 20260926161353.png]]
12. ![[Pasted image 20260926161411.png]]
13. ![[Pasted image 20260926161444.png]]
14. ![[Pasted image 20260926161655.png]]
15. ![[Pasted image 20260926161720.png]]
16. ![[Pasted image 20260926161805.png]]
17. ![[Pasted image 20260926162012.png]]
18. ![[Pasted image 20260926164406.png]]
19. ![[Pasted image 20260926164522.png]]
20. ![[Pasted image 20260926164649.png]]
21. ![[Pasted image 20260926164727.png]]
22. ![[Pasted image 20260926164947.png]]
23. ![[Pasted image 20260926165124.png]]
24. ![[Pasted image 20260926165210.png]]
25. ![[Pasted image 20260926165234.png]]
26. ![[Pasted image 20260926165404.png]]
27. ![[Pasted image 20260926165521.png]]
28. ![[Pasted image 20260926165704.png]]
29. ![[Pasted image 20260926165719.png]]
30. ![[Pasted image 20260926165911.png]]
31. ![[Pasted image 20260926170017.png]]
32. ![[Pasted image 20260926170149.png]]
33. ![[Pasted image 20260926170242.png]]
34. ![[Pasted image 20260926170340.png]]
35. ![[Pasted image 20260926170412.png]]
36. ![[Pasted image 20260926170513.png]]
37. ![[Pasted image 20260926170555.png]]
38. ![[Pasted image 20260926170716.png]]
39. ![[Pasted image 20260926170841.png]]
40. ![[Pasted image 20260926170908.png]]
41. ![[Pasted image 20260926171007.png]]
42. ![[Pasted image 20260926171109.png]]
43. ![[Pasted image 20260926171218.png]]
44. ![[Pasted image 20260926171246.png]]
45. ![[Pasted image 20260926171318.png]]
46. ![[Pasted image 20260926171520.png]]
47. ![[Pasted image 20260926171558.png]]
48. ![[Pasted image 20260926171614.png]]
49. ![[Pasted image 20260926171652.png]]
50. ![[Pasted image 20260926171728.png]]
51. ![[Pasted image 20260926171800.png]]
52. ![[Pasted image 20260926171934.png]]
53. ![[Pasted image 20260926172041.png]]
54. = ![[Pasted image 20260926172108.png]]
55. ![[Pasted image 20260926172218.png]]
56. ![[Pasted image 20260926172336.png]]
57. ![[Pasted image 20260926172507.png]]
58. ![[Pasted image 20260926172551.png]]
59. ![[Pasted image 20260926172701.png]]
60. ![[Pasted image 20260926172720.png]]
61. ![[Pasted image 20260926172819.png]]
62. ![[Pasted image 20260926172833.png]]
63. ![[Pasted image 20260926172905.png]]
64. 