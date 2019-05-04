---
layout: post
title:  "What does the AWS security certification mean?"
date:   2019-05-04
categories: aws
---

I recently passed the strangely titled "AWS Certified Security - Specialty" exam, woo 🎉. I guess that makes me "AWS Security - Specialty" certified? Quite a mouthful.

This is what AWS [say this means][cert] (numbering + split are my additions):

> Abilities Validated by the Certification:
>
>  1. An understanding of specialized data classifications and AWS data protection mechanisms
>  2. An understanding of data encryption methods and AWS mechanisms to implement them
>  3. An understanding of secure Internet protocols and AWS mechanisms to implement them
>  4. A working knowledge of AWS security services and features of services to provide a secure production environment
>
>      ----
>  5. Competency gained from two or more years of production deployment experience using AWS security services and features
>  6. Ability to make tradeoff decisions with regard to cost, security, and deployment complexity given a set of application requirements
>  7. An understanding of security operations and risk

First, note that there _isn't_ an entry along these lines: "Ability to make effective use of AWS security features to secure a production deployment". I think that is what many people would expect this certification to validate. Number 6 comes close, but isn't quite as general.

OK, so the certification is perhaps not quite targeted to what would make it most useful, but how well does it validate the abilities it does claim?

I've split the list into two halves -- the top set deal primarily with knowledge of AWS services, the bottom set are more about experience and capabilities.

# Knowledge of AWS security features

I think the certification does a good job at 1-4. It's a multiple choice exam that covers a fairly broad swathe of AWS products. Most of the questions in my exam had at least one distracter that was sufficiently plausible to put you on at best 50:50 odds for the question if you didn't know your AWS security features pretty well (usually the distracters either referenced a real AWS feature or product that doesn't _quite_ meet the requirements, or a perfect sounding feature that doesn't actually exist).

This comes with an interesting caveat due to the nature of multiple choice questions: the right answer is always staring you in the face. The exam doesn't test the ability of candidates to come up with possible solutions to a given problem, only to select a good solution from a set of options. Does this matter in practice? I suspect it does -- the majority of your AWS security challenges are not going to be of the form "which of these should I do?", they're either going to be "how do I do X", or even more subtle: knowing that there is something you should be doing in a particular area in the first place.

# Applied security engineering in an AWS environment
The next set of claimed abilities are a lot harder to validate, and I don't think the certification does a particularly good job here.

Number 5: 2+ years of production deployment experience. Firstly this is an extremely hard thing to define what it even means in terms of abilities, and secondly from my sample size of one it's demonstrably false. I have at most 0.5 years production deployment experience with AWS security services -- I started working seriously with AWS about 6 months prior to taking the exam. Perhaps my background in security and fairly intense learning over the past few months is roughly equal to a 'typical' 2 years of production experience? If you could reliably deduce this kind of information from a multiple-choice quiz, the technical part of hiring would be a solved problem.

Number 6: ability to trade off cost/security/complexity. This is _really_ hard to test in a multiple choice exam, and certainly the questions I got in my exam (they're drawn from a big pool of questions) didn't assess this in any meaningful way. As soon as you're trading off various valid answers that only vary in domains like cost/complexity/security, either (a) the question has to be quite contrived and very leading, or (b) you need _way_ more context than they include about a scenario to be able to determine the 'right' answer, and even then you might have in mind some reasonable assumptions that make an alternative answer valid. Multiple choice questions need a clear best answer, and don't allow for "[it depends][tay]".

Number 7: security operations and risk. I'm least confident about my assessment of this one, as it's the area I'm most comfortable with -- I don't remember much about the questions I got in this domain. Again though, I think most of the questions were pretty basic or fairly leading.


# In brief
To sum up, I think the cert does a good job of testing your knowledge of AWS security services and features, and not such a good job of testing that you can use that knowledge effectively.


----

<br/>
OK that's it! If you too have less than 2 years production experience of AWS security but want to pass the exam, check out the [notes] I made as I was studying. It's a big list of all the AWS-specific security topics that cover points 1-4:

<blockquote class="twitter-tweet" data-cards="hidden" data-lang="en"><p lang="en" dir="ltr">Just passed the AWS security specialty certification exam 🎉<br><br>Here are the notes I made as I went along, briefly covering the security aspects of 60ish AWS products.<a href="https://t.co/73XBs0vaXU">https://t.co/73XBs0vaXU</a> </p>&mdash; Michael Macnair (@michael_macnair) <a href="https://twitter.com/michael_macnair/status/1115358910796443650?ref_src=twsrc%5Etfw">April 8, 2019</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

[cert]:https://aws.amazon.com/certification/certified-security-specialty/
[notes]:https://github.com/mykter/aws-security-cert-service-notes
[tay]:https://twitter.com/SwiftOnSecurity/status/1002403110151745536
