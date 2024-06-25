---
title: Introduction
sidebar_position: 1
slug: /
---

This is a [Next.js](https://nextjs.org/) project bootstrapped
with [`create-next-app`](https://github.com/vercel/next.js/tree/canary/packages/create-next-app). It uses the new Next.js App Router in addition to RSC & Server Action. [Click Here](https://nextjs.org/docs/app/building-your-application/routing) to learn more about the new Next.js App Router.

## Getting Started

First, install the dependencies using your package manager of choice (preferably `pnpm`):

```bash
pnpm install
```

Then, run the following command to generate the necessary types for the project:
(Please note to update the .env file with the desired values)

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
# or
bun dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

## Tech Stack

- Next.js (App Router)
- TypeScript
- TailwindCSS
- shadcn/ui
- zod
- Storybook
- Husky
- Commitlint
- react-hook-form
- ESLint
- Prettier
- Jest

## Folder Structure

This app uses next app router to manage the routes. You can read more about
it [here](https://nextjs.org/docs/app/building-your-application/routing)

### Highlighted Folders

- `src/components`: Contains all the components used in the app.
- `src/components/ui`: Contains all shadcn/ui components used in the app.
- `src/app`: Contains all routes and layouts.
- `src/lib`: Contains all the utility functions and hooks.
- `src/styles`: Contains all the global styles used in the app.
- `src/types`: Contains custom types used in the app including backend DTO types.

### Highlighted Files

- `.env`: Contains environment variables used in the app. you can override it by creating a `.env.local` file.
- `tailwind.config.js`: Contains the tailwindcss configuration and any overrides.
- `commitlint.config.js`: Contains the git commitlint configuration rules.
