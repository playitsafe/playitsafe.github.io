---
title: A Tech spike on Next.js
date: 2024-08-04 21:29:50
tags:
  - React
  - Web
---

This tech spike is conducted to explore the frameworks that we could potentially use for our current React project. I've listed some key points as metrics that could help us to evaluate and compare among other frameworks. Thought it would be good to put here as well as a reference.

---

Next.js has emerged as a leading framework for building modern web applications with JavaScript and React. It combines the power of React for building interactive user interfaces with the robust capabilities of server-side rendering (SSR), static site generation (SSG), and dynamic routing, making it a great choice for scalable and performant web projects.

This tech spike aims to explore and evaluate Next.js as a framework for our upcoming NexGen project. Throughout this tech spike, we will conduct performance benchmarks, analyze deployment strategies, and compare Next.js with other frameworks to determine its suitability for our specific project requirements.

# 🚀 Deployment Metrics

Next.js is keen to improve building and compiling performances by applying modern bundling tools and compilers.

## Build Time

**Compiler**:
From version 12 onwards, Next.js replaces babel with a Rust-based compiler [SWC](<(https://swc.rs/)>) and it’s up to “20x faster than Babel on a single thread and 70x faster on four cores”.

From version 13 onwards, SWC compiler is used for minification by default which is 7x faster than Terser.

**Bundler**:
Option to use Turbopack can be used in Next.js in both the pages and app directories for faster local development(currently only supports `next dev` and does not support `next build`, it will be available for prod build in the future).

It will:

- leverage the [SWC](https://nextjs.org/docs/architecture/nextjs-compiler#why-swc) compiler for all transpilation and optimizations
- use a single compiler that can target multiple environments.
- Optimize dev server’s startup time([Why Turbopack?](https://turbo.build/pack/docs/why-turbopack))

## CI / CD Integration

**CI Cache:**

To improve build performance, Next.js saves a cache to .next/cache that is shared between builds(Our CI workflow will need to be configured to correctly persist the cache between builds.)

https://nextjs.org/docs/pages/building-your-application/deploying/ci-build-caching

**CD:**

`next build` emits a hybrid application which supports deployment to Node.js server and Docker containers in any cloud providers.

# ⚡Performance Metrics

## Load time

**Pre-rendering**

Next.js has two forms of pre-rendering: **Static Generation** and **Server-side Rendering**.

It allows developers to choose which pre-rendering form to use for each page. It’s possible to create a "hybrid" Next.js app by using both Static Generation and Server-side Rendering.

It improves web vital metrics through efficient code splitting, pre-rendering strategies (SSG and SSR), optimized image handling, automatic performance optimizations, built-in performance monitoring, and innovative features like Incremental Static Regeneration (ISR). These improvements help developers build fast and responsive web applications that enhance user experience and meet modern performance standards.

It achieves Incremental Static Regeneration (ISR) by combining its Static Site Generation (SSG) capabilities with a “stale-while-revalidate” strategy. This approach allows for faster page loads by serving static pages immediately while asynchronously updating them with the latest data in the background. ISR is particularly useful for dynamic content that needs to be updated frequently but can benefit from the performance benefits of static pre-rendering.

**SSR**

Caching headers (`Cache-Control`) inside `getServerSideProps` can be used to cache dynamic responses.

```ts
// This value is considered fresh for ten seconds (s-maxage=10).
// If a request is repeated within the next 10 seconds, the previously
// cached value will still be fresh. If the request is repeated before 59 seconds,
// the cached value will be stale but still render (stale-while-revalidate=59).
//
// In the background, a revalidation request will be made to populate the cache
// with a fresh value. If you refresh the page, you will see the new value.
export async function getServerSideProps({ req, res }) {
  res.setHeader(
    "Cache-Control",
    "public, s-maxage=10, stale-while-revalidate=59"
  );

  return {
    props: {},
  };
}
```

**ISR**

Next.js is able to create or update static pages after the site is built. Incremental Static Regeneration (ISR) enables it to use static-generation on a per-page basis, **without needing to rebuild the entire site**.

With ISR, Next.js enables the pre-rendering of pages containing dynamic data during the build time. Subsequently, these pages are incrementally re-rendered in the background when needed, providing an efficient and dynamic user experience.

```ts
// This function gets called at build time on server-side.
// It may be called again, on a serverless function, if
// revalidation is enabled and a new request comes in
export async function getStaticProps() {
  const res = await fetch("https://.../posts");
  const posts = await res.json();

  return {
    props: {
      posts,
    },
    // Next.js will attempt to re-generate the page:
    // - When a request comes in
    // - At most once every 10 seconds
    revalidate: 10, // In seconds
  };
}
```

When a request is made to a page that was pre-rendered at build time, it will initially show the cached page.

- Any requests to the page after the initial request and before 10 seconds are also cached and instantaneous.
- After the 10-second window, the next request will still show the cached (stale) page
- Next.js triggers a regeneration of the page in the background.
- Once the page generates successfully, Next.js will invalidate the cache and show the updated page. If the background regeneration fails, the old page would still be unaltered.

**Partial Pre-render(PPR)**([experimental](https://wanews.atlassian.net/wiki/pages/resumedraft.action?draftId=361070674))

Partial Prerendering is an experimental feature that allows static portions of a route to be prerendered and served from the cache with dynamic holes streamed in, all in a single HTTP request.

Partial pre-rendering(uses the React Suspense API) allows developers to control which part of a page is pre-rendered or rendered first.

PPR leverages a combination of static processing, specifically incremental static regeneration (ISR) and full server-side processing (SSR). With ISR, Next.js enables the pre-rendering of pages containing dynamic data during the build time. Subsequently, these pages are incrementally re-rendered in the background when needed, providing an efficient and dynamic user experience.

**ServerComponents**

It allows the server to render and cache the react component. The rendering work is further split by route segments to enable streaming and partial rendering.

Next.js App Router is built with streaming by default. This helps improve both the initial page loading performance, as well as UI that depends on slower data fetches that would block rendering the whole route.

Streaming helps to render UI from the server progressively. Work is split into chunks and streamed to the client as it becomes ready. This allows the user to see parts of the page immediately, before the entire content has finished rendering.

**Caching mechanism**

| Mechanism                                                                                                | What                       | Where  | Purpose                                         | Duration                        |
| -------------------------------------------------------------------------------------------------------- | -------------------------- | ------ | ----------------------------------------------- | ------------------------------- |
| [Request Memoization](https://nextjs.org/docs/app/building-your-application/caching#request-memoization) | Return values of functions | Server | Re-use data in a React Component tree           | Per-request lifecycle           |
| [Data Cache](https://nextjs.org/docs/app/building-your-application/caching#data-cache)                   | Data                       | Server | Store data across user requests and deployments | Persistent (can be revalidated) |
| [Full Route Cache](https://nextjs.org/docs/app/building-your-application/caching#full-route-cache)       | HTML and RSC payload       | Server | Reduce rendering cost and improve performance   | Persistent (can be revalidated) |
| [Router Cache](https://nextjs.org/docs/app/building-your-application/caching#router-cache)               | RSC Payload                | Client | Reduce server requests on navigation            | User session or time-based      |

**Built-in components**

Next.js has its built-in components that helps to improve the rendering performance and UX.

- `<Script>`: To load a third-party script , Next.js will ensure the script will only load once, even if a user navigates between multiple pages.

- `<Images>`: Built on the native `<img>` element. The Image Component optimizes images for performance by lazy loading and automatically resizing images based on device size.

- `<Link>`: Built on the native `<a>` tags. The Link Component prefetches pages in the background, for faster and smoother page transitions.(prefetch pages when a user hovers over a link, improving the perceived performance by loading resources in advance.)

## Bundling

- Splitting by routes

code splitting is done automatically based on routes. Each page component in the `pages` directory becomes a separate chunk of JavaScript. When a user navigates to a specific route, only the JavaScript necessary for that route is loaded.

- Dynamic import

Next.js leverages dynamic imports using `import()` to split code into smaller bundles. Instead of loading all JavaScript files upfront, dynamic imports allow components or modules to be loaded asynchronously when they are needed.

```ts
import dynamic from "next/dynamic";

const DynamicComponent = dynamic(() => import("./Component"), {
  loading: () => <p>Loading...</p>,
});

function HomePage() {
  return (
    <div>
      <DynamicComponent />
    </div>
  );
}
```

# 🎚️Scalability and Extensibility

## State management

Local state management: `useState`

Global State Management options: redux, React Context API, MobX

- Redux:

  Redux is well-suited for large-scale applications with complex state management needs. Its centralized store and consistent handling of state make it easier to manage a large and growing codebase.

  Redux can lead to better performance in large applications due to its optimized handling of state changes and efficient update mechanisms.

- ContextAPI:

  Context API is useful for smaller applications or specific parts of larger apps.

- MobX:

  It leverages proxy to create mutable data sources which can be directly written to or reactively read from. Race conditions in asynchronous updates can be an issue for large applications.

## SEO

Next.js is an excellent choice for building SEO-friendly applications due to its server-side rendering (SSR) capabilities, which provide search engines with fully-rendered HTML content. Additionally, Next.js comes with built-in optimizations, such as automatic code splitting and prefetching, that contribute to better SEO performance

Meta Tags and Title Optimization(Dynamic Meta Tags and Titles):

By leveraging the `next/head` component, you can customize these tags based on the page’s content dynamically.

Next.js is able to dynamically generate meta tags and titles based on the content of each page. Server-side data fetching can be used to pass the required information to the `next/head` component.

## A11y

**Route Announcements**

Next.js includes a route announcer by default. When transitioning between pages rendered on the server (e.g. using the `<a href>` tag) screen readers and other assistive technology announce the page title when the page loads so that users understand that the page has changed.

**Linting**

Next.js provides an [integrated ESLint experience](https://nextjs.org/docs/pages/building-your-application/configuring/eslint) out of the box, including custom rules for Next.js.

## Security

By implementing SSR and SSG, pages are rendered on the server before being sent to the client. This approach helps mitigate client-side vulnerabilities such as XSS (Cross-Site Scripting) attacks because content is sanitized and controlled on the server. SSG also reduces exposure to runtime attacks since content is pre-rendered and does not execute on the server or client.

Next.js includes built-in support for setting security headers like X-Frame-Options, X-XSS-Protection, Content-Security-Policy, and others. These headers help protect against common web vulnerabilities such as click jacking, XSS, and data injection attacks.

# 🛠️ Developer experience

## Community and support

With over 6 million weekly downloads(123k stars on Github), Next.js has a large and active community of developers across the world and a pretty mature ecosystem.

Github repo: https://github.com/vercel/next.js

Github discussion board: https://github.com/vercel/next.js/discussions

Discord: https://discord.com/invite/bUG2bvbtHy

Reddit: https://www.reddit.com/r/nextjs

StackOverflow: https://stackoverflow.com/questions/tagged/next.js

## 3rd-party libraries

Alongside Next.js, numerous third-party libraries and tools have been developed by the community to complement the framework and provide solutions for various aspects of web development including:

- nextjs-auth0
- swr
- TanStack libraries
- Storybook

`@next/third-parties` is a library that provides a collection of components and utilities that improve the performance and developer experience of loading popular third-party libraries in your Next.js application.

## Learning curve

Low for developers who have React experience as most of it is React and there’s a possibility we can reuse the components we created for current projects.

# 🗝️ Key Features

**File-based Routing**

Next.js uses a file-based routing system, where each React component in the pages directory corresponds to a route, simplifying navigation and code organization.

**API Routes**

It allows developers to build API endpoints directly within their Next.js application using API routes, making it easy to handle server-side logic and data fetching.

**Image Optimization**

Next.js provides built-in support for optimizing images, including automatic resizing, lazy loading, and responsive image handling, improving performance and user experience.

# Other

## Other companies using Next.js

- The Washington Post
- Tencent News
- OpenAI
- Netflix Jobs
- …
