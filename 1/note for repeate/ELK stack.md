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
65. 
66. 
67. 
68. ![[Pasted image 20260926190111.png]]
69. ![[Pasted image 20260926190131.png]]
70. ![[Pasted image 20260926190258.png]]
71. ![[Pasted image 20260926190545.png]]
72. ![[Pasted image 20260926190622.png]]
73. ![[Pasted image 20260926190855.png]]
74. ![[Pasted image 20260926190930.png]]
75. ![[Pasted image 20260926191007.png]]
76. ![[Pasted image 20260926191414.png]]
77. ![[Pasted image 20260926191506.png]]
78. ![[Pasted image 20260926191541.png]]
79. в поде может быть нескольок контейнеров ![[Pasted image 20260926191559.png]]
80. борг стал кубернетисом п оистории, докер появился  в 2013 г ,докер и куб написаны на го ?![[Pasted image 20260926192244.png]]
81. ![[Pasted image 20260926192539.png]]
82. ![[Pasted image 20260926192851.png]]
83. ![[Pasted image 20260926192954.png]]
84. как дебажить некоторые приложения внутри куба ?
85. ![[Pasted image 20260926194539.png]]
86. ![[Pasted image 20260926194557.png]]
87. ![[Pasted image 20260926194619.png]]
88. ![[Pasted image 20260926194644.png]]
89. ![[Pasted image 20260926194730.png]]
90. ![[Pasted image 20260926194810.png]]
91. ![[Pasted image 20260926194853.png]]
92. ![[Pasted image 20260926194918.png]]
93. ![[Pasted image 20260926195012.png]]
94. ![[Pasted image 20260926195026.png]]
95. ![[Pasted image 20260926195055.png]]
96. ![[Pasted image 20260926195111.png]]
97. ![[Pasted image 20260926195137.png]]
98. ![[Pasted image 20260926195201.png]]
99. ![[Pasted image 20260926195228.png]]
100. ![[Pasted image 20260926195316.png]]
101. ![[Pasted image 20260926195345.png]]
102. ![[Pasted image 20260926195429.png]]
103. ![[Pasted image 20260926195440.png]]
104. ![[Pasted image 20260926195453.png]]
105. ![[Pasted image 20260926195639.png]]
106. ![[Pasted image 20260926195733.png]]
107. ![[Pasted image 20260926195818.png]]
108. ![[Pasted image 20260926195941.png]]
109. ![[Pasted image 20260926200002.png]]
110. ![[Pasted image 20260926200016.png]]
111. ![[Pasted image 20260926200032.png]]
112. ![[Pasted image 20260926200142.png]]
113. ![[Pasted image 20260926200155.png]]
114. ![[Pasted image 20260926200243.png]]
115. ![[Pasted image 20260926200305.png]]
116. ![[Pasted image 20260926200404.png]]
117. ![[Pasted image 20260926200419.png]]
118. ![[Pasted image 20260926200434.png]]
119. ![[Pasted image 20260926200508.png]]
120. ![[Pasted image 20260926200550.png]]
121. ![[Pasted image 20260926200601.png]]
122. ![[Pasted image 20260926200811.png]]
123. ![[Pasted image 20260926200946.png]]
124. ![[Pasted image 20260926201102.png]]
125. ![[Pasted image 20260926201134.png]]
126. ![[Pasted image 20260926201216.png]]
127. ![[Pasted image 20260926201305.png]]
128. ![[Pasted image 20260926201326.png]]
129. ![[Pasted image 20260926201446.png]]
130. ![[Pasted image 20260926201539.png]]
131. 