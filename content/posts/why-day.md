---
title: "_Why Day, and a really old MS-DOS project"
date: 2022-08-19T05:59:05-05:00
draft: true
tags: ["2022"]
---

Happy [\_why day!](https://www.whyday.org/) I learned about *Why the Lucky Stiff* when I was looking for resources to learn Ruby for my upcoming internship. Though I don't really understand much of what made \_why popular, I do understand his love for writing code for the sake of it - something I don't feel like I've been doing enough of. So, I thought today might be the day I dust off a really old DOS project I worked on way back in 2018, and see if I can still run it. This was one of my first programming projects, and it's been a while since I've messed around with it.

 What makes it a strange project is that it is written for MS-DOS using Turbo C++. This wasn't out of interest or nostalgia, but because my high school curriculum had literally not been updated since the 1990s, which meant that we learned how to program on Turbo C++. (For any future employers reading this, I can assure you that I now do have experience with modern IDEs.)
 
 To get the project running on my laptop, I had to first figure out how to get [DOSBOX and Turbo C++ running on Linux](https://www.geeksforgeeks.org/how-to-install-turbo-c-on-linux/), which was surprisingly easy. (It was definitely easier than getting it running on the Mac I used to have back when I made this project). The harder part was getting used to the Turbo C++ GUI. Even though it supports mouse input, it isn't intuitive and most operations still require keyboard shortcuts.

![BankProject.png](../why-day/BankProject.png#center)

(and yes, I apparently never fixed that warning. 😎)

Running the project was simple enough since my entire project was written as part of **one** giant  `.cpp` file. The only thing I did have issues with was the graphics library that I used. Due to an issue with how Turbo C++ managed working directories, I had to copy over [all of the contents of the BGI folder into the BIN folder.](https://stackoverflow.com/questions/7605942/bgi-error-how-to-resolve-it)  After a couple of minutes of tinkering with my code, I finally managed to get this ancient project running, and... it was just as I thought it would be:

![BankProject2.png](../why-day/BankProject2.png#center)

The project doesn't do much, it's a simple "Banking Database" that can do transactions and give you reward points. It uses binary data files to store account information and supports generating transaction receipts in `.txt` format. It's... not the kind of project I'd brag about in an interview now, but it was cool to work on at the time. Everything happened through keyboard input, and there was no CSS padding or race conditions to worry about. 

![33b81e97c2d6f9a99e39c99355be4022.png](../why-day/33b81e97c2d6f9a99e39c99355be4022.png#center)

I'm not sure how much longer I'd be able to run this project on a modern machine. Sure, DOSBOX is a very well-supported software for now, but who knows how well Turbo C++ or this particular graphics library would be supported for. But I still have fond memories of working on this scrappy little project, and I think a small part of that programmer still lives on in me today.

Source Code: https://github.com/Aryaman73/BankingSystemCPP