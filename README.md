Алгоритмическая угадайка от Google: 1 000 000$ как я решил задачу и улучшил свой алгоритм трижды
Недавно я наткнулся на интересную задачку, которая, по слухам, используется на собеседованиях в Google. На первый взгляд – простая угадайка: нужно отгадать комбинацию из нескольких элементов, получая после каждой попытки лишь подсказку о числе совпадений. Но стоило мне углубиться, как стало ясно: эта задача отлично тренирует стратегическое мышление, анализ вероятностей и применение эвристик. Изначально я придумал решение, а затем трижды его улучшил, добившись стопроцентного результата и снизив среднее число попыток. В этой статье я подробно расскажу, как размышлял, какие гипотезы проверял, и как шаг за шагом превратил "угадайку" в чётко работающий алгоритм.

Условия задачи
Для начала опишем условия задачи кратко и ясно:

Есть список из 100 адресов. Каждый адрес состоит из 5 параметров: страна, город, улица, номер дома, номер квартиры.
Программа случайно загадывает один адрес из этого списка.
У пользователя есть до 10 попыток угадать адрес.
В каждой попытке пользователь называет один из 100 адресов и спрашивает: «Правильно ли я угадал?».
Если адрес не совпадает с загаданным, программа не сообщает, какие именно поля не угаданы. Вместо этого она возвращает подсказку: число от 0 до 5 – сколько параметров из пяти совпало, или -1 (минус один), если не совпало ничего.
Цель – разработать алгоритм, который гарантированно найдет загаданный адрес не более чем за 10 попыток, используя числа совпавших параметров как подсказки.
Другими словами, из 100 возможных адресов нужно за 10 вопросов однозначно вычислить правильный, опираясь только на информацию о количестве совпадений по каждому предложенному варианту.

От адресов к числам: упрощение модели
Чтобы не запутаться в названиях стран и улиц, я представил адреса в виде чисел. Разобьем диапазон 0–49 на пять категорий по 10 чисел в каждой:

0–9 – условные коды стран,
10–19 – коды городов,
20–29 – улицы,
30–39 – номера домов,
40–49 – номера квартир.
Каждый адрес можно задать пятиэлементным массивом, взяв по одному числу из каждой категории. Например, комбинация [3, 17, 22, 37, 42] соответствует некому адресу (страна №3, город №17, улица №22, дом №37, квартира №42). Всего различных комбинаций пять по десять – миллион:

10×10×10×10×10=106=1 000 000.10×10×10×10×10=106=1000000.

<img width="710" height="585" alt="image" src="https://github.com/user-attachments/assets/17a3a38e-b2a0-417b-9e69-832bc544e9cb" />

Алгоритмическая угадайка от Google: 1 000 000$ как я решил задачу и улучшил свой алгоритм трижды
На первый взгляд, угадать правильный адрес из миллиона вариантов нереально. Но! По условию игры мы заранее получаем список из 100 возможных адресов, среди которых загаданный. Это ключевое упрощение: задача сводится не к поиску по всему миллиону, а к поиску одного элемента из заданных 100. Значит, можно заранее проанализировать эти 100 кандидатов и воспользоваться любой информацией о них.

🧩 Наблюдение: Заранее известный ограниченный список кандидатов – это наше скрытое преимущество. Вместо бесполезных угадываний вслепую по всему пространству, мы можем проанализировать частоту появления отдельных элементов в этих 100 комбинациях, прикинуть вероятность их участия в загаданном адресе и выстроить стратегию исключения. По сути, задача превращается в игру на выбывание, где каждый ход дает новую информацию для отсечения части оставшихся кандидатов.

Шаг 1: Взвешивание чисел и сила нулевого совпадения
Первую стратегию я построил вокруг идеи «веса» числа. Я прошёлся по всем 100 адресам и подсчитал, сколько раз встречается каждое из 50 возможных чисел (0–49). Получилось, что одни числа (например, код какой-то страны или номер квартиры) встречаются в списке чаще, а другие – реже. Эти частоты можно воспринимать как «вес»: чем чаще число появляется в выборке, тем выше шанс, что именно оно могло быть загадано.

Далее я вычислил вес каждой комбинации – как сумму весов входящих в неё пяти чисел. Комбинации с самым большим суммарным весом состоят из самых часто встречающихся чисел. Логика простая: если какие-то элементы популярны среди 100 адресов, есть вероятность, что загаданный адрес содержит несколько из них. Я решил в первую попытку предлагать комбинацию с максимальным весом, т.е. составленную из самых популярных чисел.

Однако наша цель – не обязательно угадать с первого раза. Парадоксально, но в идеале первая попытка должна не угадать адрес, а полностью промахнуться. Зачем это нужно? Если программа ответит, что совпадений 0 (или -1, что эквивалентно “ничего не совпало”), это лучшая новость для нас! Такой ответ означает, что ни одно из пяти самых популярных чисел не входит в загаданную комбинацию.

Почему нулевое совпадение – праздник? Потому что эти 5 чисел встречаются во многих адресах из списка. Получив 0 совпадений, мы с уверенностью вычеркиваем все комбинации, которые содержат хоть одно из этих пяти чисел. А таких, как правило, очень много – зачастую половина или даже две трети от всех кандидатов.

Другими словами, мы сознательно взяли самые "популярные" элементы, чтобы в случае неудачи одним махом отсечь максимальное количество адресов. После первого же хода список кандидатов может сократиться на 50–70%. Мы изначально надеемся на промах, потому что эта осознанная неудача резко упрощает дальнейший поиск.

<img width="521" height="822" alt="image" src="https://github.com/user-attachments/assets/26d829dc-d405-49d6-ab2a-50114a4649c6" />


✅ Результат (после шага 1): Такой подход оказался весьма неплохим. В симуляции 1000 игр алгоритм сумел найти правильный адрес за ≤10 попыток в примерно 87.5% случаев (875 игр из 1000), при среднем числе попыток около 6.3. Заметим, это достижимо без какой-либо сложной логики анализа совпадений – мы использовали только веса чисел и агрессивно удаляли комбинации при ответах с нулевым совпадением.

Но 87.5% успеха – это ещё не 100%, да и иногда алгоритму могло потребоваться вплотную к 10 попыткам. Как повысить надежность решения?

Шаг 2: Пересчёт весов после каждого хода
Одно из ключевых улучшений – сделать алгоритм адаптивным. После каждого хода (особенно после такого удачного промаха на 0 совпадений) наш список возможных комбинаций заметно уменьшается. Это значит, что частоты чисел меняются: если раньше какое-то число встречалось в 12 адресах из 100, то после удаления половины списка оно может остаться всего в 5 адресах из 50, и его относительная важность падает.

Если продолжать использовать изначальные веса, мы будем руководствоваться устаревшей информацией. Многие наивные решения этим грешат – оставляют веса статичными. Это грубая ошибка. Алгоритм должен пересчитывать вес каждого числа и каждой оставшейся комбинации после каждого хода, исходя из обновленного списка кандидатов. Проще говоря, на каждом шаге мы снова оцениваем, какие из оставшихся чис наиболее вероятны, и выбираем следующую комбинацию с максимальным суммарным весом.

Такой динамический пересчёт делает стратегию «живой». Алгоритм непрерывно подстраивается под актуальную ситуацию: каждый новый ход основан на самой свежей информации о распределении элементов. Каждый следующий шаг – это новая, на данный момент оптимальная гипотеза.

На практике пересчёт весов значительно повысил эффективность. Алгоритм стал реже тратить попытки впустую и еще активнее отсекал кандидатов. В тех же 1000 симуляций доля игр, выигрываемых за ≤10 ходов, выросла примерно до 99%. То есть добавление адаптивности почти устранило неудачные случаи. Тем не менее, оставался крошечный процент особо сложных сценариев, когда за 10 попыток адрес все же не угадывался.

Шаг 3: Учёт частичных совпадений (логический фильтр)
На предыдущих шагах мы радикально использовали информацию о 0 совпадений (полном промахе). Но что делать, если ответ не ноль, а какое-то число от 1 до 4? Ранее алгоритм просто сохранял эту цифру «в память» и продолжал дальше, не делая мгновенных выводов. Настало время использовать и частичные совпадения в полную силу.

Представим, что наша попытка дала, скажем, 1 совпадение. Это значит, что из пяти названных чис ровно одно присутствует в загаданном адресе, а остальные четыре – нет. Это ценный факт, который позволяет наложить дополнительные ограничения на возможные адреса.

✂ Пример: Предположим, мы спросили про комбинацию [3, 11, 26, 35, 43] и получили в ответ 1 совпадение. Назовём множество этих пяти чис M. Теперь рассмотрим любой оставшийся кандидат на ответ:

Если этот кандидат содержит два или более числа из M, он не может быть верным адресом. Почему? Потому что в нашем эксперименте из M подошло только одно число, а у кандидата их совпало бы как минимум два – противоречие.
С другой стороны, если какой-то кандидат вовсе не содержит ни одного из чис из M, то у него с нашей попыткой было бы 0 совпадений, что тоже противоречит известному результату (должно быть ровно одно).
Вывод: единственный способ для кандидата оставаться в игре – содержать ровно одно число из набора M. Все комбинации, которые не удовлетворяют этому условию, можно смело вычеркивать.

Обобщая этот подход: если какая-то проверенная комбинация дала k совпадений, то любая потенциально правильная комбинация должна совпадать с ней ровно в k элементах. Кандидаты, у которых пересечение по элементам больше или меньше k, исключаются.

После добавления логического анализа частичных совпадений алгоритм стал отсекать неверные варианты буквально на каждом шаге. Процент успешного угадывания подскочил до 99.9%, и среднее число попыток еще немного снизилось. Тем не менее, оставались крайне редкие случаи, когда к десятой (последней допустимой) попытке оставался больше чем один кандидат. Требовался финальный рывок, гарантирующий победу.

Алгоритмическая угадайка от Google: 1 000 000$ как я решил задачу и улучшил свой алгоритм трижды
Шаг 4: Логический вывод ответа на последнем шаге (метки правды)
Чтобы довести точность до абсолютных 100%, я реализовал особый прием на случай, если к последней попытке выбор всё еще не однозначен. К этому моменту у нас накоплено достаточно информации о загаданном адресе в виде результатов всех предыдущих попыток. Каждая такая попытка с неполным успехом – это как улика, которую можно использовать.

Я назвал эти улики «метки правды». Метка правды хранит:

комбинацию, которую мы пробовали,
и точное число совпавших в ней элементов с загаданным адресом (от 1 до 4).
На 10-й попытке (если до неё дело вообще дошло) алгоритм делает следующее: перебирает все оставшиеся кандидаты и проверяет их на соответствие всем меткам правды. Кандидат должен удовлетворять каждому условию: скажем, с первой меткой у него должно совпадать ровно столько-то элементов, со второй – ровно столько-то, и т.д. По сути, мы ищем комбинацию, которая одновременно даст ровно нужное число совпадений со всеми нашими предыдущими запросами.

Естественно, такая комбинация будет только одна – именно загаданный адрес. Мы не «угадываем» наудачу, а логически вычисляем правильный ответ. В десятую попытку программа предлагает именно этот оставшийся вариант, который проходит по всем меткам правды.

<img width="606" height="817" alt="image" src="https://github.com/user-attachments/assets/561991f5-7e9a-4000-8a01-a672150f7572" />


✅ Результат (после шага 4): Алгоритм стал безошибочным. В дополнительных 1000 симуляций каждая игра завершилась успехом не позднее десятой попытки. Даже самые хитрые случаи с множеством пересекающихся частичных совпадений завершались победой. Метки правды и проверка кандидатов по ним обеспечили 100% точность алгоритма.

Стоит отметить, что таких ситуаций (неопределённость до последнего хода) осталось очень мало – в большинстве случаев правильный адрес находился раньше. Но теперь у нас был гарантирующий механизм «на случай войны».

Алгоритмическая угадайка от Google: 1 000 000$ как я решил задачу и улучшил свой алгоритм трижды

<img width="1076" height="809" alt="image" src="https://github.com/user-attachments/assets/bc83b072-d829-4413-b52c-1a153805769b" />

Шаг 5: Раннее логическое угадывание для сокращения числа попыток
Достигнув стопроцентной надежности, я задался новым вопросом: могу ли я сделать алгоритм быстрее в среднем, не жертвуя успехом? В реальных задачах важна не только гарантия решения, но и эффективность – меньше шагов значит меньше ресурсов или времени.

Идея оптимизации оказалась спрятана в уже реализованном механизме. Я подумал: если мы собираем метки правды по ходу игры, почему бы не попытаться использовать их до последней попытки? Раньше я воспринимал логический перебор на основе меток как «крайнее средство» – мол, если уж до 10-го хода не нашлось, тогда проверяем всех. Но зачастую комбинации меток становятся достаточно определяющими уже к 5-6 ходу.

Теперь алгоритм начал на каждой итерации (начиная примерно с 3-й или 4-й попытки, когда накопилось хоть немного информации) пытаться вывести ответ логически:

Собираем текущие метки правды (все попытки, где 1–4 совпадений).
Проверяем оставшиеся кандидатуры: есть ли среди них комбинация, которая удовлетворяет всем имеющимся меткам?
Если находим такую уникальную комбинацию, мы делаем следующий ход не по максимальному весу, а сразу предлагаем этого кандидата как предполагаемый ответ.
Проще говоря, как только частичная информация становится достаточной, алгоритм переключается с эвристики на чистую логику и выводит ответ напрямую, не дожидаясь принудительно десятого шага.

✅ Результат (после шага 5): Абсолютная угадываемость осталась 100%, зато среднее число попыток заметно снизилось. В тех же тестах среднее значение упало примерно с 6.3 до 5.7.

Алгоритм научился раньше распознавать ситуацию, когда он уже фактически знает ответ, и не тратит лишние ходы. Особенно ощутимо это в партиях, где 2–3 метки правды логически перекрывают друг друга и однозначно указывают на единственный возможный адрес – нередко верный ответ удается получить уже к 5-й или 6-й попытке.

Главное отличие этого шага от предыдущего в том, что мы не ждём последней попытки, а используем накопленные сведения на лету. Если шаг 4 был «страховкой» на случай затяжной игры, то шаг 5 – это про ускорение игры, когда она и так складывается удачно.

Выводы
В итоге мне удалось разработать алгоритм, который:

Всегда находит правильный ответ за 10 попыток, используя комбинацию статистического подхода и логического анализа.
Адаптивно подстраивается после каждого хода, пересчитывая вероятности на основе оставшихся вариантов.
Извлекает максимум информации из каждого ответа: нулевые совпадения сразу отсекают большие пласты данных, частичные совпадения накладывают строгие фильтры на оставшиеся варианты.
Эффективен в среднем: большинство игр завершается значительно раньше десятого хода.
На этой задаче я в очередной раз убедился, как важно сочетать простые эвристики (частотный вес, популярность элементов) с строгой логикой (точное соответствие всем собранным условиям). Сначала эвристика быстро сужает область поиска, а логический вывод доводит дело до конца. Такой подход может пригодиться во многих областях, где нужно оптимизировать поиск или диагностику при ограниченном числе попыток и частичной обратной связи. И, конечно, приятно осознавать, что даже головоломку из собеседования можно превратить в отточенный алгоритм с гарантиями 😊.
