---
authors: ["emulator000"]
date: "2023-09-05"
share: true
tags: [ai, chatbot, business, rust, openai, chatgpt]
title: "AI Chat: An upcoming new project"
layout: post
languageCode: "en-US"
toc: true
---

I've been keeping a secret, and it's time to spill the beans. I'm excited to introduce the world to AI Chat, a project
in collaboration that's been "cooking" in my lab for a while now.

## What is AI Chat?

AI Chat is all about bringing the power of AI to businesses, helping them enhance customer interactions, streamline
processes, and level up their engagement game with a fully customized chat bot.

This isn't just another chat bot; it's an AI assistant that's tuned to understand every business's unique needs, and
communicate with clients in a natural and effective way.

![ai-chat-logo](/assets/img/ai-chat/logo.png)

## Creating AI assistants in a click

One of the coolest things about AI Chat is its ability to create AI assistants that are truly in tune and shapes with
your company's data. You can feed and fine-tune it with various types of information – from PDFs and Excel sheets to
website content (thanks to the brand new Rust's crate [crawly](https://github.com/CrystalSoft/crawly)) and more – and watch it transform into a knowledgeable
assistant that speaks your company language. And here's the best part: you're in control. You can update its knowledge
whenever you want or even wipe the slate clean.

![ai-chat-screen1](/assets/img/ai-chat/screen1-en.png)

## Simplify customer interactions

With AI Chat, you can effortlessly craft conversations that feel human. Tweak responses on the go, with your provided
context and bot personaly, ensuring that your customers have a seamless experience.

Whether it's answering queries, providing support, or guiding users through processes, your AI assistant has got your back.

![ai-chat-video](/assets/img/ai-chat/video-en.gif)

## Team collaboration

For businesses, collaboration is key. AI Chat takes teamwork seriously by allowing you to collaborate with your team members
on bot management. You can also customize the role and permission of every single invited team member.

![ai-chat-screen2](/assets/img/ai-chat/screen2-en.png)

## The tech behind

Curious about the tech that powers AI Chat? Here's a an high-level summarization:

- **Frontend**: The frontend is built using React, Next.js, and Material-UI, providing a modern and intuitive user interface.
- **Backend**: The backend is made in Rust, using Tonic, Serde, Diesel, and Tokio and many other libraries.
- **Database**: PostgreSQL handles the heavy lifting, managing bot configurations and user information.
- **Internal services communication**: Redpanda enables smooth event handling and communication between services.

## Stay tuned

AI Chat is on the horizon, and I can't wait to share more details with you.

Keep an eye on the official [LinkedIn](https://www.linkedin.com/company/99948946/admin/feed/posts/) and
[GitHub](https://github.com/aichat-bot) profile for updates. Feel free to reach me out and share your thoughts!
