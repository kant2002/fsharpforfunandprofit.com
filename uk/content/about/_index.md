---
layout: page
title: "About this site"
description:
hasComments: 1
date: 2020-01-01
---

The goal of this site is to introduce .NET developers to the joys of functional programming, and F# in particular.

Please be aware of the [Terms and Conditions](/about/terms/) when using this site.

I hope that this site will live up to its title and demonstrate that not only is F# a lot of fun to program in, but it is also useful for mainstream business and enterprise software. F# is not just an academic exercise, it is meant to be useful.

Мій підхід неапологічно .NET орієнтований і не академічний. Наприклад:

* I will often use C# style naming rather in preference to the more cryptic names in functional programming, especially in the introductory posts.
* Я намалюю аналогії та приклади з об'єктноорієнтованих концепцій таких як шаблони дизайну.
* I will avoid many of the more sophisticated concepts (monads, lazy vs. eager evaluation, etc) and focus on concepts that are most useful to newcomers from the OO world: algebraic types, pattern matching, higher-order functions, etc.

## Про мене

I am a developer and architect working with the [**fsharpWorks** consultancy](https://fsharpworks.com/). I have over 20 years experience in a wide variety of areas from high-level UX/HCI to low-level database implementations.

I have written serious code in many languages, my favorites being Smalltalk, Python, and more recently, F# (hence this site).


## FAQ

**Більшість прикладів коду використовують функціональний стиль, а не об'єктноорієнтований стиль. F# може використовувати обидва. Чому акцент на функціональному? Хіба тобі не подобається ООП?**

Я люблю ООП і протягом багатьох років був серйозним Смолтолківцем.  But for F#, I focus on the functional side more than the OOP side because, for people coming from a C# or Java background, that's where all the new* concepts are. Things like type inference, function composition and so on, don't work well with the OO style. And these concepts are precisely the things that I think people coming to F# should understand.

{{<footnote "*">}} "New" to mainstream developers, that is. The FP concepts go back to the \[1920s\](http://en.wikipedia.org/wiki/Moses_Sch%C3%B6nfinkel) and for use in programming languages, as early as the \[1960s\](http://en.wikipedia.org/wiki/ISWIM) (earlier if you count lisp). ML, the direct ancestor of F#, was created in the \[early 1970s\](http://en.wikipedia.org/wiki/ML_programming_language).
{{</footnote>}}

**Why are you writing these long series rather than short focused posts?**

This is not really a blog, more a set of pages with the aim of explaining how F# works. There are many excellent blogs on FP and F#, but personally, when I am learning a language, I prefer a more structured approach. There are so many new* concepts in F# that trying to understand it piecemeal can be an exercise in frustration. Places like [Stack Overflow](http://stackoverflow.com/questions/tagged/f%23) have lots of good answers to direct questions, but some of the questions show a [serious](http://stackoverflow.com/questions/11086368/declaring-a-variable-without-assigning) [misunderstanding](http://stackoverflow.com/questions/3779098/f-mutable-function-arguments) of what functional programming is all about, and there is not the space to explain why. So rather than just answer a single question in isolation, I'd rather change the context so that those questions are not even considered.

**Some of the posts have really important ideas buried in the middle of the page. Why not pull this stuff into posts of their own?**

Again, the answer is context. For example, the section on why [Option.None is not the same as null](/posts/the-option-type/#option-is-not-null) comes at the bottom of the page containing the explanation of what an option type actually is (and a diagram as well!) I think that if you read through the entire page, rather than trying to get a quick answer, then the content of that last section (and the answer to the "null" question) will be self-evident.

{{< linktarget "banned" >}}

I do highlight the most important bits on the [contents page](/site-contents/) as well, so they are not completely buried!



## Заборонені слова

Many innocent people might visit this site, so to avoid causing offence, certain obnoxious words and phrases are strongly discouraged.

These words include: "endofunctor", "anamorphism", "existential quantification", "beta reduction", "category theory", "final coalgebra", "Kleisli arrows", "Curry--Howard correspondence" and worst of all, the five letter word beginning with "m".

Repeated use of these words will result in banning. You will be exiled and forced to spend time with [other incorrigible elements](http://www.haskell.org/haskellwiki/Haskell).**

{{<footnote "**">}}
Seriously! As I make clear on the home page, this site is not targeted at mathematicians or Haskell programmers. It is targeted at the vast numbers of C#, VB and Python programmers who are coming to functional programming for the first time. F# is a fantastically accessible language for the average enterprise programmer, but mathematical jargon puts a lot of people off ([*"a monad is a monoid in the category of endofunctors, what's the problem?"*](http://james-iry.blogspot.co.uk/2009/05/brief-incomplete-and-mostly-wrong.html)), and I think it is much better to explain F# with concepts from within its native environment, rather than using terminology that originated elsewhere and is often not applicable. For example, treating F# computation expressions like Haskell monads can often make things more confusing. Doing this also helps to bypass the ["how can you get anything done in language X? It doesn't even have y."](http://en.wikipedia.org/wiki/Paul_Graham_(computer_programmer)#Blub) debate and focus on what F# can do well.
{{</footnote>}}


## Зворотний зв'язок

Будь ласка, допоможіть мені покращити цей сайт. Я з радістю отримую будь-яку конструктивну критику і коментарі.

If you have comments or suggestions about this site as whole, please leave a comment on this page (below), or you can contact me directly at `scottw at fsharpforfunandprofit.com`.

If you'd like to get occasional updates about the site, please [subscribe to the newsletter](\subscribe.html).


## Подяки

First of all, many thanks to Don Syme and the rest of the F# team who have created a great language which integrates so well with .NET. Завдяки їм існує надія, що цілком функціональна мова може нарешті досягти мейнстриму.

Second, thank you to all the F# enthusiasts who have written books, blogs, and contributed to [StackOverflow](http://stackoverflow.com/questions/tagged/f%23) and hubfs.net (a.k.a. [fpish](http://fpish.net)). I have learned a lot from you all.

Дуже дякую тим, хто надіслав мені поправки до багатьох одруків на сайті, зі спеціальною подякою Андрію ван Мелебруку.

І нарешті, спеціальна подяка *вам* і всім людям, які читають цей сайт. Сподіваюся, вам сподобається. Дякуємо за читання!


