# Project: RIZ (Radical Insight Zone)

## Overview

A full-stack monorepo featuring a NestJS backend configured for AWS Lambda, a React admin dashboard, and a React Native mobile application.

## Tech Stack

- **Language**: TypeScript (Node.js runtime, using bun and pnpm)
- **Framework**: NestJS (`riz-be`), React + Vite (`riz-admin-fe`), React Native + Expo (`riz-app-v2`)
- **Database**: PostgreSQL with Prisma ORM (`riz-be/apps/nest`)
- **Lint**: ESLint + Prettier
- **Build**: Turborepo (`riz-be`), Vite (`riz-admin-fe`), Expo EAS (`riz-app-v2`)
- **Test**: Jest (`riz-be`), Vitest (`riz-admin-fe`)

## Commands

- **Lint**: `pnpm lint` (backend), `bun lint` (frontend/mobile)
- **Type check**: `npx tsc --noEmit` / `bun check`
- **Test**: `pnpm test` (backend), `bun test` (frontend)
- **E2E**: `pnpm test:e2e` (backend)
- **Deploy**: AWS CDK (`riz-be/apps/cdk`), EAS Build (`riz-app-v2`)
