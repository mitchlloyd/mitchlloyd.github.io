---
layout: post
title:  "Still Programing Part 2: Can't Someone Else Do It?"
date:   2015-11-01 19:28:35 -0800
---

We've covered some common thoughts about eliminating programmers by making their
jobs easy enough for anyone to do in [part 1]({{page.previous.url}}).
Now we need to tackle the question of whether computers will learn to program
and eliminate programming jobs.

We Don't Need Computers That Can Program
----------------------------------------

First you should know that we already have software that can write code.  Today
programmers write specifications in languages that they prefer and then
computers write the instructions that they will execute later. That's called
[compiling](https://en.wikipedia.org/wiki/Compiler). We even have compilers
that can watch how code is being used as it runs and rewrite it on the fly to
perform better. We also have programs that can write human-readable code (this
is called code generation), but that doesn't tend to be very useful -- it just
saves a little typing.

For the foreseeable future humans will need to tell a computers what to do.
This isn't much of an issue though because once we tell a computer how to do
something, we never need to tell it again.

Let's talk through an example. Your boss says, "I have a file with everyone's
birthday in it. I want you to make a program that sends everyone in the company
an email on their birthday."

By the end of the day you have the program ready for your boss and it looks
something like this:

``` ruby
def send_birthday_greetings(filename)
  parse_file(filename).each do |person|
    if person[:birthday] == Date.today()
      send_email(person[:email], "Happy Birthday!")
    end
  end
end
```

This code loads a file, looks at each person in the file, and, if their birthday
is today, sends them a message that says "Happy Birthday!". We don't need to
write that program every day, we can configure it to run daily.

The next day your boss comes to you and says she wants a program that will send
a greeting on everyone's hire anniversary. At this point, there is no need to
write another program, you can modify the previous one.

``` ruby
def send_anniversery_greetings(filename, attribute, message)
  parse_file(filename).each do |person|
    if person[attribute] == Date.today()
      send_email(person[:email], message)
    end
  end
end
```

This time the code takes a file, the name of an attribute in that file that
represents an anniversary, and the message to send to users.

We just made our program more abstract and flexible. We can even create a
user interface for our boss so that if she wants to send out a message on
everyone's pet's birthday she can do that without even talking to us.

If our job was to send birthday greetings, we're unemployed. If our job was to
write programs that send messages on certain days, we've programmed our job out
of existence.

As programmers we avoid writing repetitive code or solving the same problem
twice.  We take repetitive tasks and make it possible to do them over and over
at nearly zero cost.

Beyond eliminating repetition, programmers work to eliminate "incidental
complexity" or the work that is not essential to the problem at hand.
Committees of programmers create standards and enhance platforms so that
we can stop dealing with trivial technical problems and focus on the
fundamental complexity of what we're working on.

For instance, front-end developers used to spend hours creating images to
produce rounded corners on the web and now this can be accomplished in with one
line of CSS code. When programmers recognize problems that are solved over and
over, they create reusable libraries and frameworks that can radically improve
the speed and reliability of software construction. Programmers are so fanatical
about improving the state of their work that they often join world-wide
communities of volunteers to build and maintain software that eliminates
programming work.

If programming keeps getting "easier" then why isn't there less demand for
software developers? Because as creating software gets more efficient, we find
new uses for it and tackle new problems that were too hard and too expensive to
do before.  This is why we can have autonomous vehicles and too many online
social networks.

If Computers Could Program
--------------------------

So the very nature of programming is that it eliminates repetitive work. Let's
focus back on the question: Can we teach a computer to program? Can a computer
take some very basic input from a human and create a new program as a human
programmer would?

Here's our thought experiment: Tom wants to start a business selling books, so
he goes to a computer and says, "build a program to sell books." The computer
recognizes this command and, with the precision of a team of master programmers,
creates and deploys a new book-selling application in one second.  Tom logs into
his new online store.  There he needs to fill out some information.  He needs
his business's bank information. He needs some way to tell the application what
books is he selling and whether they are physical or can be downloaded.  What
shipping providers is he going to use?

This is getting a little involved, but this isn't programming, right? This is
just using and configuring a program.

My point is this: Even if we could teach a computer to write a program like this
it wouldn't be useful. If someone came to me today and said they want to sell
books on the web (and that is all the information they gave me) I would go
online, find an existing program, and use that. Again, this is
the nature of programming -- after we write a program we can run it and copy it as
many times as we want at very little cost.

The only point where a "programming computer" becomes useful is when it requires
very little human thought and input to work. In other words, the computer would
need to use Artificial Intelligence fill in all the logical gaps to create the
program. Tom says, "make me money" and the computer decides that a book store
would be profitable.  It gathers information on book buying habits, creates a
store, sets up a bank account. The computer writes 1,000 teen vampire books and
decides to sell them exclusively on the Kindle. It brokers a deal with Amazon
for this exclusivity and gets a bigger share of the profits.  The store nets
over $1,000,000 in the first six months of operation.

At this point we're talking about sentient and omniscient computers that can
create new solutions to problems that humans have not thought of. If we ever get
here, a decline in programming jobs will be the last thing on our minds.

Programming is Not a Good Candidate for Cheap Outsourcing
---------------------------------------------------------

So if humans are going to continue programming for now, why can't we ship this
work to low-wage workers overseas as my teachers suggested in 1998? There are
two aspects of programming that make this strategy difficult. First, adding
programmers to a software project has rapidly diminishing returns. Second, for a
software project to succeed, the ideas that motivate it and the ideas in the
program must be closely aligned.

For most software projects I would not expect, for instance, a team of 1000
programmers to produce software any better or faster than a team of 5
programmers. Having more developers means that companies can work on more
projects, but it does not allow them to finish any one project faster.
Furthermore, having 1000 less experienced and less skilled programmers on a
project will never compare to 5 experienced and skilled programmers.  The ideal
programming team is small, motivated, and competent.

> The bottom line is, no large organization is ever going to achieve the ideal
> of a handful of talented, like-minded people communicating well. (53:03)
>
> Small groups of focused, really smart people can do great things in short
> periods of time -- things that larger groups of less motivated, less
> experienced, less talented people can never do.
>
> It's not like, ok it takes these 5 people in room a year to do this but if I
> have 300 people and 3 years, I could equal them. No, will never equal them
> with 300 people if you don't have the right five... (58:00)
>
> -- John Siracusa on [Accidental Tech Podcast #55](http://atp.fm/episodes/55-dave-who-stinks)

My experience may be skewed since, as a consultant working in the US, if I'm
working with client that tried to build something cheaply with an offshore team
they probably have been burned. Still I'll present some anecdotal evidence to
give an idea of how bad things can get for these companies.

In one case, a friend of mine was working with an offshore team based in India
to create a software product that gave paying users access to content. After
spending a year and more than $300,000 on the product, they finally had to
abandon it. This was a devastating blow to this small company and they had to
give up on their growth strategy and change their business model to survive.  My
friend later described the product to me and I told him that all of the
functionality could be found in a free product called Wordpress.  He installed
it in a weekend, picked out a theme, and added the company logo.  They
successfully use that product today, but the company is still recovering from
the financial impact of the failed project.

I had another friend whose company invested about $80,000 in a product that was
lead by a US-based consultancy and outsourced to cheap overseas workers. The
product failed to work consistently (mainly due to some bizarre architecture
choices and unmaintainable programming practices). The problem itself was trivial and
the incumbent team created a reliable solution in a couple days of work.

Why We Still Have Local Developers
----------------------------------

Over and over I've seen large teams of cheap programmers fail to deliver what
small, motivated, and competent teams of three to five programmers are capable
of. There are teams that can produce software in a matter of weeks that another
team could never create.

This phenomenon can't be explained by looking at the intelligence, skill, or
intrinsic motivations of the programmers involved. I've been on projects where I
was floored by what the team delivered and I've been on projects where the
programmers had no way to help a project succeed.

When companies work with cheap, offshore labor they create a situation where it
is impossible for programmers to succeed. These companies expect to give very
little input about the problem they are solving and believe that the sheer
number of programmers they have at their disposal will produce results. The
relationship between the client and the company doing the programming work
becomes adversarial as they begin to argue about exactly what the original
requirements said and whether those requirements were technically delivered.

More programmers is rarely better because as team sizes increase the number of
possible communication connections increases exponentially. A 3 person team has
3 connections, a 5 person team has 10 connections, and a 15 person team has 105
connections. Think 15 person software teams are rare? Keep in mind that I'm not
talking about the programmers on a team, I'm talking about everyone that
participates in the process of building software: Executives, Project Managers,
Programmers, Designers, Marketers, Copywriters and Quality Assurance engineers.

Given that programming is the practise of synthesizing ideas into solution that
can be reperformed by a computer, any distance, noise, and "game of telephone"
effects between the problem and the solution are devastating. When a competent
programmer knows the full scope of a problem, nothing can rival their
productivity. When the knowledge of a problem is overseas and several employment
hierarchies away from a programmer, a solution may be unattainable.

I've leaned on the word "cheap" to describe the type of outsourcing that was
foretold in 1998, but low-cost is not what makes these teams ineffective. The
fundamental problem lies in the relationship that hiring companies create with
these providers: a relationship where the company's employees dream of a product
and then expect programmers to translate these dreams into "code". In this
relationship, the ideas are valuable and "coding" is a comoditized service.

I've worked with extremely effective offshore teams, but these teams were
selected for their expertise, not just for their price. These programmers were
integrated into the development process as much as any employee and were
involved in designing the product, not just coding it into existence based on a
set of requirements.

Successful products are built in partnership with developers with the
understanding that ideas are cheap and execution is critical.
