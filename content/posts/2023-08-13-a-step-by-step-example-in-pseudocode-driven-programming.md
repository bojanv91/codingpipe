---
title: "How to Use Pseudocode to Design Better Software - Working Example"
date: 2023-08-13
dateUpdated: Last Modified
permalink: /posts/a-step-by-step-example-in-pseudocode-driven-programming/
tags:
  - Software Design
  - Code
layout: layouts/post.njk
---

Pseudocode is a plain language text that describes the steps of a computer program.

Let's look at the following example. We have a program that takes user's input, parses it, then generates a QR code image from the parsed input, and finally it saves the image to the filesystem.

Here is the first iteration of the pseudocode for it:

```csharp
// read user's input value
// parse the input
// generate QR code image from the parsed input
// save the image to filesystem
// print the image filepath so the user can know where to look for the image
```

Here is the final iteration of the pseudocode:

```csharp
// input = read user's input value
// parsedValue = parseInput(input)
// image = generateQrCodeImage(parsedValue)
// saveImage(image, filepath)
// print(filepath) 
```

Most of the time this is the very first step I take before I start writing computer code. I breakdown the problem into smaller chunks to get better sense of it. Then I write the solution down using pseudocode.
Note that the above pseudocode was written and rewritten a couple of times until I was satisfied with the program's workflow and the naming.

From here I start writing actual code like this:
```csharp
// input = read user's input value
Console.WriteLine("Please enter an input value:");
var input = Console.ReadLine();

// parsedValue = parseInput(input)
var parsedInput = ParseInput(input);

// image = generateQrCodeImage(parsedValue)
byte[] image = GenerateQrCodeImage(parsedInput);

// saveImage(image, filepath)
var imageFilepath = SaveImage(image, "C:\\qrcode-images");

// print(filepath) 
Console.WriteLine(imageFilepath)
```

Then, I can remove the comments:
```csharp
Console.WriteLine("Please enter an input value:");
var input = Console.ReadLine();

var parsedInput = ParseInput(input);
byte[] image = GenerateQrCodeImage(parsedInput);

var imageFilepath = SaveImage(image, "C:\\qrcode-images");
Console.WriteLine(imageFilepath)
```

Also, I can write some tests for the parser like this:
```csharp
Assert.Equal("mode=a;value=787878;", ParseInput("a.2222787878"));
Assert.Equal("mode=d;value=112233;", ParseInput("d asdf112233"));
Assert.ThrowsException(() => ParseInput("qwex"));
```

Finally, I can implement the remaining methods using the same approach. Sometimes the implementation is straightforward, sometimes is not. The important thing is to unblock yourself, don't rush to write actual "final" code, make a good design/flow/structure in pseudocode, then implement stuff. 

Rules of thumb:
- First breakdown solutions in pseudocode, then in computer code
- Pseudocode can be written closer to a plain language or closer to a computer code
- Keep the pseudocode high-level, but not too generic
- Solve problems using an "outside-in" way - start from the main program flow (outside), and then focus on the details (inside)
