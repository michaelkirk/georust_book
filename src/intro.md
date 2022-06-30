# Answering geospatial questions with Rust

Someone once said that space is the final frontier, like it's just some far off thing. But space is everywhere — all around us and between everything everywhere. It's so familiar, and yet there are so many questions to be answered about it. How big is it? What should we do with it? Who gets what? Are we there yet?

# Why geospatial

Geospatial computation just means getting the computer to answer questions about maps.

Ok, maybe that's an oversimplification — but it's not _so_ far off.

The reason I like geospatial computation as a field is, unlike a lot of things that happen with computers, the results of geospatial work can be tied back to the real world we all share. Transportation planners are striving to get us home safe, climate scientists are measuring catastrophe, and, yes, also the worst kinds of extractionists are working hard in the opposite direction. Information is power, and all kinds of people are using geospatial techniques to forward their agendas. It can be an exciting field indeed.

# Why Rust

Nobody likes to be lost for long; Rust is extremely fast. For millennia, maps have acted as a form of shared memory to safely access stored locations; Rust can help you ensure that shared map locations stored in memory are also [accessed safely](https://doc.rust-lang.org/nomicon/meet-safe-and-unsafe.html). People really love discovering new places; people also [really love Rust](https://insights.stackoverflow.com/survey/2021#technology-most-loved-dreaded-and-wanted).

Whether you are interested in Rust for its performance or security, integrating geospatial data with Rust is a natural combination.

# Who is this book for

If you're new to the field of geospatial, welcome! The examples that follow are  not intended to be a comprehensive introduction to the field of geospatial analysis, but I have tried to include just enough context for you to get your bearings, so that the examples should be approachable by anyone with an interest in the field.

If you've previously done some geospatial analysis in another environment, but are new to the Rust programming language, hello neighbor! My intent is that the subsequent pages aren't too tedious, but feel free to gloss over the text and skip ahead to the well labeled examples, which give real solutions to practical problems.

In any case, the path of your life has led you here: the right place to get started. This set of examples will help you navigate geospatial problems using a powerful programming language that may not quite be familiar to you yet.

So feel free to [install Rust](https://www.rust-lang.org/learn/get-started); make yourself a latte (but leave the Java behind); pick any topic on the left; and follow the Coordinate Reference System of your own heart as we dive headfirst into common geospatial questions, and try to get the computer to answer them. 🖖🤖

## DRAFT NOTES

re: "the computer" - I know this probably sounds a little awkward, but it's this intentionally sort of retro framing that _I_ like to use. I think back to the 90's when we got our first computer and it was always "go use _the_ computer to (get directions/look something up/etc.)". These days it's just "get directions" "look up", etc. and the "use the computer" part is implied. I think because interaction with the machine has become so mediated by "apps" or whatever that we "use the app", rather than the computer, which means we can only do what the apps allow. But programming and composing your own solutions, like we're doing in this book, is a little bit more like "use the computer". IDK, maybe it's not effective and we should get rid of it, but I wanted to at least explain it if it seemed inexplicable.

re: star trek references ("the final frontier...") - This was just easy to use, but does it pander too much to an insider crowd rather than drawing in a more diverse audience?

re: headings - I used really matter-of-fact headings. They are boring. Id love to "spice them up" but in the introduction when people are trying to decide if they want to proceed with the book, I think maybe we should weigh more towards clarity of their intent more so than pun-pertunity.

re: the java joke is pretty good, but I'm considering removing it. In reality a lot of really good GIS software is in Java and probably it shouldn't be "left behind". And... I don't want to alienate anyone over it.

re: georust - I removed references to it from the intro. I'm not sure yet if we should mention it explicitly. There's just plenty of things that exist outside of github.com/georust/* that are useful, and to talk about "georust plus some other crates" seems like a mouthful with little benefit.
