# BPM Counter Copilot Instructions

## Project

- This is a Vue 3 application written in TypeScript and built with Vite.
- Use Vue single-file components with `<script setup lang="ts">`.
- `src/App.vue` owns the page shell; reusable interaction logic belongs in `src/components/`.
- `src/components/Tapper.vue` owns keyboard tap handling, BPM calculation, hit counting, and pulse animation.
- RxJS is the existing tool for event streams and timing calculations. Preserve its use for tap-timing behavior unless there is a clear reason to change the design.
- `vite-plugin-pwa` is configured for the app. Treat `dev-dist/` and other generated build output as generated files; do not edit them by hand.

## Code Style

- Follow the repository Prettier configuration: two spaces, semicolons, single quotes, trailing commas where configured, and an 80-character print width.
- Keep TypeScript strict and avoid `any`, unused locals, and unused parameters.
- Prefer small, readable Vue components and existing dependencies over introducing new abstractions or packages.
- Preserve the public behavior of the tapper: keyboard events produce BPM and hit-count updates, and a long pause resets the timing accumulation.
- Keep UI changes responsive and accessible. Preserve keyboard usability and provide visible focus states for interactive controls.
- Avoid editing unrelated files or reformatting code outside the requested change.

## Validation

- Install dependencies with `npm install` when needed.
- Run `npm run build` after code changes. This runs `vue-tsc -b` and the Vite production build.
- There is currently no test script or test suite. When changing timing or interaction behavior, manually verify keyboard tapping, BPM updates, the two-second reset behavior, and the pulse animation in the development app with `npm run dev`.
- Do not add generated PWA output to source changes unless explicitly requested.
