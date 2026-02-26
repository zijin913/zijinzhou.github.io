---
title: Nutrica – AI-Powered Nutrition Tracking
summary: An AI-powered full-stack web app that turns meal photos and text descriptions into structured nutrition logs.
tags:
  - AI
  - Web
  - Full Stack
  - React
  - Node.js
  - OpenAI
date: 2025-10-01

links:
- name: Live Demo
  url: https://nutrica.app/
- type: code
  url: https://github.com/zijin913/Nutrica
---

## Demo

<video width="100%" controls>
  <source src="nutrica.mp4" type="video/mp4">
</video>

## Project Overview

Nutrica is an AI-powered nutrition tracking application that turns messy, real-world meal inputs into structured nutrition logs. Users can upload photos of food labels and meals or simply describe what they ate in natural language. The system automatically extracts calories, carbohydrates, fats, and protein and logs them into a daily dashboard.


I built Nutrica as a modern, production-style full‑stack project, focusing on real user experience rather than just a toy demo.

## Key Features

- **AI Food Analysis**
  - Image-based nutrition extraction from food labels and meal photos.
  - Natural language meal descriptions parsed into structured nutrition facts.
  - Automatic emoji generation to make each food entry more visual and memorable.

- **Nutrition Tracking & Gamification**
  - Daily tracking of calories, carbs, fats, and protein with a calendar-based history view.
  - Personalized nutrition goals and visual progress indicators.
  - Puzzle-style collection system that rewards consistent logging and promotes engagement.

- **User Experience**
  - Mobile‑first, responsive UI with smooth transitions.
  - Shareable nutrition reports generated directly from the app.
  - Secure authentication and private user data.

## My Role & Responsibilities

- Designed and implemented the **React frontend** (routing, state management, and UI components).
- Built the **Node.js + Express backend API** for AI parsing, food logging, and user collections.
- Integrated **OpenAI GPT‑4** for image and text-based food analysis.
- Used **Supabase** for authentication and database, and deployed the stack with **Vercel** (frontend) and **Railway** (backend).
- Implemented logging, error handling, and basic performance monitoring in production.

## Tech Stack

- **Frontend**: React, Vite, React Router, modern CSS, html2canvas
- **Backend**: Node.js, Express, OpenAI API, Supabase, Multer, Winston
- **Infrastructure**: Vercel (frontend), Railway (backend), Supabase (DB & auth)

