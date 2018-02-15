# Как читать программы на Прологе #

Совершенно возможно написать программу на Прологе за один присест, и даже если вы перепишете её после этого пару раз, прочитана она будет вами и другими разработчиками несравненно большее количество раз. Следовательно, важно знать, как же лучше всего читать программы на Прологе, как понять, что именно они значат?

Существует несколько способов прочтения чистых программ на Прологе, и мы объясняем некоторые из них в этой главе.

## Декларативное прочтение ##

Declaratively, Prolog programs state what holds. Программы на Прологе состоят из утверждений, и каждое утверждение является либо фактом, либо правилом. Факты выражают то, что истинно всегда. Правила выражают то, что истинно при выполнении определённых условий.

Декларативно правило Head :- Body. читается как: «Если Body верно, то и Head тоже верна». В терминах математической логики это же правило читается как «Из Body следует Head», и записывается как Body → Head или Head ← Body. Интересно заметить, что знак :- на самом деле был выбран как намёк на стрелку ←. Since Body defines the conditions under which Head holds, it can also be regarded as a constraint on the set of solutions. This way to read Prolog programs is also called concluding reading.

A major advantage of this approach is that it is easy to explain, understand and use. You state what holds under what conditions, and the Prolog engine finds solutions for you. A disadvantage of this approach is that it does not explain why logically equivalent program variants may exhibit different performance or termination characteristics.

### Пример ###

Рассмотрим предикат `list_list_together/3`, описывающий объединение двух списков:

    list_list_together([], Bs, Bs).
    list_list_together([A|As], Bs, [A|Cs]) :-
            list_list_together(As, Bs, Cs).

Давайте прочитаем это определение декларативно, то есть в терминах отношений между аргументами, точно так, как записано в двух утверждениях этого предиката:

- Объединение пустого списка `[]` и любого другого списка `Bs` равно тому же самому списку `Bs`.
- Если объединение `As` и `Bs` равно `Cs`, то объединение `[A|As]` и `Bs` равно `[A|Cs]` для любого `A`.

Заметьте насколько универсально такое прочтение: It is applicable if any arguments are instantiated, and also if they are not.

## Процедуральное прочтение ##

To complement the declarative approach of reading Prolog programs, we can also read them procedurally. This means that we take into account the actual computation strategy of the Prolog engine. Operationally, invocation of a Prolog predicate is similar to a procedure or function call in other languages. However, two critical differences remain: First, Prolog variables are truly logical variables. Second, Prolog provides backtracking as a built-in feature, and will exhaustively try alternatives.

For these reasons, understanding a Prolog program procedurally is significantly harder than understanding the control flow of many other programming languages. In particular, when tracing Prolog, you need to take into account:

- _instantiation_ of variables,
- _aliasing_ between variables,
- _alternatives_ found on backtracking.

The need to keep track of these complexities and their interactions is a major drawback of this approach. An advantage of the approach is its potential to explain different performance characteristics and termination properties of program variants.

Note also that a procedural reading almost invariably implies a particular direction of use, and therefore typically does not do justice to the full generality of logical relations.

### Пример ###

Let us read list_list_together/3 (shown above) procedurally for the query ?- list_list_together([x,y], [z], Cs).

Применимо ли первое правило? No, because [x,y] does not unify with [].
Применимо ли второе правило? Note that due to the way resolution works, we must introduce fresh variables
when considering a clause. So, let us use A', As', Bs', and Cs' for the variables A, As, Bs and Cs that appear in the clause head. The answer is: Yes, the second clause applies with the bindings A'=x, As'=[y], Bs'=[z], Cs=[A'|Cs']. This is already rather cumbersome and error-prone, and it gets even harder to keep track of all bindings as we proceed further.
Carrying on, we consider the goal list_list_together([y], [z], Cs'), repeating the same questions for this goal.
Применимо ли первое правило? No, because [y] does not unify with [].
Применимо ли второе правило? We need to rename the variables again. Let us use A'', As'', Bs'' and Cs''. Yes, the clause applies with A''=y, As''=[], Bs''=[z] and Cs'=[A''|Cs''].
Now the whole ordeal once more, as we consider the goal list_list_together([], [z], Cs'').
Применимо ли первое правило? Yes, at last! Note that we again need to introduce fresh variables of course. Let us use Bs''' to denote the single variable of the first clause at this step of the computation. So the first clause applies with Bs'''=[z] and Cs''=Bs'''. This means that at last a solution is found and reported as a binding for the original variable Cs which is the only variable that appears in the query. If you have carefully followed this trace (as I am sure you have, since it is so enjoyable to read), you know that Cs was unified with [A'|Cs']. Since A' was unified with x, and Cs' was unified with [A''|Cs''], and further A'' was unified with y, this makes Cs the same as [x,y|Cs'']. As we just mentioned, Cs'' was unified with Bs''', and Bs''' is [z]. Thus, the solution we found is Cs=[x,y,z], and this solution is reported by the toplevel.
We still need to consider the second clause too though: No, it does not apply, because [] does not unify with [_|_].
Это только для одного конкретного случая! Accurately covering all possible modes of invocation with a procedural reading is extremely complex, and typically would take an extremely elaborate explanation. In general, you will not be able to carry this approach through, because there are too many cases to consider.

## Program slicing ##

Program slicing is a simple and powerful technique that uses very general properties of pure Prolog to study the effects of generalizations and specializations of a program.

Examples of such properties are:

- removing a goal can make the program at most more general, never more specific
- removing a clause can make the program at most more specific, never more general
- inserting false/0 between any two goals in a rule lets us ignore the procedural effects of all goals after that point.

In a very precise sense, program slices ar explanation that answer why we observe certain phenomena.

We illustrate this with a simple example. Suppose a programmer has written list_length/2, relating a list to its length as follows:

    list_length([_|Ls], N) :-
        list_length(Ls, N0),
        N #> 0,
        N #= N0 + 1.
    list_length([], 0).

The predicate works exactly as intended if the list is sufficiently instantiated. For example:

    ?- list_length([], L).
    L = 0.

    ?- list_length([_,_,_], L).
    L = 3.

However, the predicate does not generate a single answer for the most general query:

    ?- list_length(Ls, L).

Program slicing helps us to see the reason. Strikeout text is used for parts that are not relevant for the behaviour we observe:

    list_length([_|Ls], N) :-
        list_length(Ls, N0),
        false,
        N #> 0,
        N #= N0 + 1.

    list_length([], 0) :-
        false.

The remaining fragment is by itself already responsible for the nontermination. No change in the strikeout parts can prevent it.

Program slices can be generated automatically and are a powerful way to locate the causes of nontermination and other unintended properties in Prolog programs. See for example Stefan Kral et al., Slicing zur Fehlersuche in Logikprogrammen, WLP 2000.