# Geospatial: Rust the Process

Luke Skywalker once said that space is the final frontier, like it's just some distant concept far, far away. Set your lightsaber to stun just for a second though, and you quickly realize that space is vast and ever-present — it's all around us and between everything everywhere. It's familiar, yet elusive. How big is it? What should we do with it? Are we there yet?

## What Is Geospatial Computation?

If a computer is answering questions about maps, congratulations! That's geospatial computation.

OK, maybe that's an oversimplification — but it's not _so_ far off.

Unlike a lot of abstract things that happen with computers (e.g. seeing "#nofilter" on a sepia-toned photo of someone with cartoon cat ears), the results of geospatial work can be tied back to the real world. Transportation planners use precise measurements to start building new roads to make traffic better in the future, while present-day commuters use map apps to navigate around the absolute chaos those planners have caused. Climate scientists measure rising ocean levels, while Florida real-estate agents measure the availability of brand-new oceanfront condos. Hikers at home dream about getting lost in the wilderness, while hikers lost in the wilderness dream about finding a way home.

No matter what your agenda is, information is power, and all kinds of people are using geospatial techniques to go places. It can be an exciting field indeed.

## Why Rust?

Nobody likes to be lost for long; Rust is extremely fast. For millennia, maps have acted as a form of shared memory to safely access stored locations; Rust can help you ensure that shared map locations stored in memory are also [accessed safely](https://doc.rust-lang.org/nomicon/meet-safe-and-unsafe.html). People really love discovering new places; people also [really love Rust](https://insights.stackoverflow.com/survey/2021#technology-most-loved-dreaded-and-wanted).

Whether you are interested in Rust for its performance or security, integrating geospatial data with Rust is a natural combination.

## Who Is This Book For?

If you're new to the geospatial field, welcome aboard! The examples that follow are not intended to be a comprehensive introduction to the field of geospatial analysis, but all of them were designed to be approachable by anyone with an interest in learning more. There should be just enough context for you to get your bearings, but feel free to explore some side quests before coming back to the main story.

If you've previously done some geospatial analysis in another environment but are new to the Rust programming language, hello neighbor! Feel free to jump around or skip ahead to the well-labeled examples, which give real solutions to practical problems.

In any case, the path of your life has led you here: the right place to get started. This set of examples will help you navigate familiar geospatial problems using a powerful programming language that may not quite be familiar to you yet.

So feel free to [install Rust](https://www.rust-lang.org/learn/get-started); make yourself a latte (with or without Java); pick any topic on the left; and follow the [Coordinate Reference System](https://docs.qgis.org/2.18/en/docs/gentle_gis_introduction/coordinate_reference_systems.html) of your own heart as we dive headfirst into common geospatial questions and learn how to use computers to efficiently find the answers. 🖖🤖
