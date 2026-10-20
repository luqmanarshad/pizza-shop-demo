# Pizza Shop Ordering System (Demo)

A front-end demo of a pizza shop ordering and kitchen system built with **Vue 3**, **Vite** and **Tailwind CSS**.

**Live demo:** https://luqmanarshad.github.io/demos/pizza/

## Features

- **Menu:** pizzas, sides and drinks with category filters.
- **Customization:** size options and extra toppings, with live price updates.
- **Cart and checkout:** quantity handling, delivery or pickup, delivery charge in the total.
- **Kitchen view:** orders move through New, Preparing, Ready and Delivered, with a running order count and sales total.

## Notes

- This is a demo with sample data. There is no backend or database; orders are saved in the browser's `localStorage`.
- Production versions of systems like this are built with Laravel (REST API, MySQL, queues for notifications) and Vue.js.

## Run locally

```bash
npm install
npm run dev
```

Build for production with `npm run build`.

## Author

Luqman Arshad, Full Stack Engineer (Laravel, Vue.js, Angular).
Portfolio: https://luqmanarshad.github.io/
