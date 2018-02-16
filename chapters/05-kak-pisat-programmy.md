# Как писать программы на Прологе

Самый лучший совет как писать код на Прологе был приведён Ричардом О`Кифе в его книге The Craft of Prolog:

> Elegance is not optional.

Следуйте этому совету! Если ваша программа на Прологе выглядит не элегантно, остановитесь и подумайте, как можно её улучшить. Эта глава содержит некоторые советы как следует писать код на Прологе.

## С чего начать?

Рассмотрим снова определение предиката list_list_together/3, который был рассмотрен в главе Как читать программы на Прологе. Как самому придти к такому определению?

Когда вы пишете предикат на Прологе, думайте о том, при каких условиях он должен быть верным. Например, в случае с предикатом list_list_together/3, мы можем рассудить следующим образом: во-первых, мы хотим описать отношение между списками. Из рекурсивного определения списка мы знаем, что нам необходимо будет рассмотреть по крайней мере два разных случая:

пустой список, представленный атомом []
составной терм в форме [L|Ls]
Эти два случая образуют "скелет" определения нашего предиката:

    list_list_together([], Bs, Cs) :-
        ...
    list_list_together([L|Ls], Bs, Cs) :-
        ...

Эти два правила соответствуют двум случаям, которые у нас возникли, а именно пустому и непустому спискам в качестве первого аргумента. Зададим себе вопрос: при каких условиях верны эти два правила. Подумав можно заметить, что первое правило верно, если Bs = Cs. Мы можем записать это правило так:

    list_list_together([], Bs, Cs) :-
        Bs = Cs.

Подобную унификацию можно перенести в голову предложения, и переписать правило в форме факта:

    list_list_together([], Bs, Bs).

Применим тоже самое рассуждение и ко второму правилу: When does it hold that Cs is the concatenation of [L|Ls] and Bs? A bit of reflection tells us: This holds if Cs is of the form [L|Rest] and Rest is the concatenation of Ls and Bs. We use the built-in predicate (',')/2 to express this conjunction of conditions. To describe that Rest is the concatenation of Ls and Bs, we use list_list_together(Ls, Bs, Rest), since this is precisely the relation that ought to hold in this case.

The whole clause therefore becomes:

    list_list_together([L|Ls], Bs, Cs) :-
        Cs = [L|Rest],
        list_list_together(Ls, Bs, Rest).

Это пример предиката, чьё определение ссылается на самого себя. Подобные предикаты называются рекурсивными. Note how recursive definitions naturally arise from considering the conditions that make such predicates true.

Как и прежде унификацию можно перенести в голову правила, упростив тем самым его запись:

    list_list_together([L|Ls], Bs, [L|Rest]) :-
        list_list_together(Ls, Bs, Rest).

В качестве упражнения читатель может попробовать подобрать более удачные имена для переменных.

When beginners write their first Prolog programs, a common mistake is to ask the wrong question: "What should Prolog do in this case?". This question is misguided: Especially as a beginner, you will not be able to grasp the actual control flow for the different modes of invocation. In addition, this question typically limits you to only one possible usage mode of your predicate. Therefore, do not fall into this trap! Instead, think about the conditions that make the relation hold, and provide a clear declarative description of these conditions. If you manage to state these conditions correctly, you often naturally obtain very general predicates that can be used in several directions. Thus, when writing Prolog code, better ask: What are the cases and conditions that make this predicate true?

Вы можете подумать: всё это, конечно, здорово и работает для простеньких отношений. But what if I want to actually "do" something, such as incrementing a counter, removing an element etc.? И на этот вопрос ответ всё тот же: Think in terms of relations between the entities you are describing. To express a modification of something, you should define a relation between different states of something, and state the conditions that make this relation hold. See Thinking in States for more information.

## Выбор имени для предикатов

Хорошее имя предиката подсказывает роль каждого аргумента. В идеале предикат может быть использован во всех направлениях. Это означает, что любой аргумент может быть переменной --- частично или полностью связанной. Эта гибкость должна быть выражена в имени предиката, обычно путем выбора существительных для описания аргументов.

Примеры удачных имён:

- `списко_длина/2`, связывает список с его длиной
- `целое_слудующее/2`, свазывает целое число с числом следующим за ним
- `студент_курс_оценка/3`, связывает студентов с курсами, которые они посещают, и полученными там оценками.

В этих случаях имена предикатов настолько ясны, что дополнительные описания кажутся излишними. Заметим также что длинные_имена_использующие_знак_подчёркивания_читаются_легко, а вот именаНаписанныеСлитноВРазныхРегистрахЧитаютсяЗначительноТруднее.

По этим причинам можно назвать следующие примеры менее удачными:

- `длина/2`: Каким по порядку аргументом идёт длина, первым или вторым?
- `следущееЦелое/2`: Не так легко прочесть, как `следущее_целое/2`, и не так легко понять как `целое_следущее/2`.
- `прочесть_оценки/3`: Бессмыслица в случае, если оценки уже связанные переменные.

## Именование переменных

Имена переменных в Прологе дожны начинаться либо с заглавной буквы, либо со знака подчёркивания. Последнее правило полезно знать, если вы, например, преподаёте Prolog в Японии. В отличии от имён предикатов, имена переменных в Прологе иногда записывают в СмешаныхРегистрах. Тем не менее, почти во всех случаях такие имена переменных состоян не более чем двух слитно написанных слов.

Некоторые предикаты Пролога описывают последовательность изменения состояний, чтобы выразить изменение состояния чистым способом. В таких случаях может быть очень полезно следующее соглашение: начальное состояние обозначается как `Состояние0`, следующее за ним состояние --- `Состояние1` и т. д. Это перечисление продолжается до конечного состояния, которое мы называем `Состояние`. Полностью последовательность будет выглядеть следующим образом:

    Состояние0 -> Состояние1 -> ... -> Состояние

Of course, the prefix State can denote any other entity that is being described. For example, if multiple elements are inserted into an association list, we may have the sequence:

    Assoc0 → Assoc1 → Assoc2 → ... → Assoc

When writing higher-order predicates, it is good practice to denote with C_N a closure C that is called with N additional arguments. For example, the first argument of maplist/2 could be called Pred_1, because it is invoked with one additional argument.

## Отступы

Пролог очень простой язык: есть весьма ограниченный набор языковых конструктов, и несколько общепринятых способов отступа. Однако, независимо от стандарта оформления кода, всегда следует придерживаться одного золотого правила, а именно, никогда не ставить `(;)/2` в конце строки. Иначе точку с запятой ";" будет легко спутать с запятой ",", а поскольку `,/2` сама почти всегда встречается в конце строки, хорошей практикой будет поместить ";" либо в начале строки, либо посреди между двумя целями дизъюнкции, так, чтобы её лечге было отличить от конъюнкции.

## Дополнительные материалы

Covington et al., Coding Guidelines for Prolog, содержит несколько интересных наблюдений касательно программирования на Прологе.
