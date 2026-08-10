# Overview

- Next.js is a full-stack React framework. React provides components and state management, while Next.js provides the structure around React needed to build and deploy a complete web application

- The framework handles routing, server rendering, data fetching, caching, API endpoints, image and font optimization, metadata, bundling, and production builds

- Next.js does not replace React. Every Next.js interface is still made from React components. Next.js decides where and when those components execute, how users reach them, and how the result is delivered

- The modern routing system is the App Router, held in the `app` directory. The older Pages Router, held in the `pages` directory, is still supported but should mainly be learned for maintaining older projects

- Next.js 16 is the baseline for this file. Its main defaults are the App Router, TypeScript-ready tooling, React Server Components, and Turbopack

- A useful mental model is:

    React = How the interface is built

    Next.js = How the interface becomes a complete web application

    Node.js = One possible runtime executing the server-side code

    Browser = The runtime executing Client Components

- Next.js applications may contain three kinds of work at once:

    Static work: Completed during the build and reused for every visitor

    Cached work: Completed occasionally and reused until it is revalidated

    Dynamic work: Completed for a specific request or user

- The main benefit of Next.js is choosing the correct kind of work at the component level instead of forcing the entire application to be purely client-rendered or purely server-rendered

# Installation and Commands

- Next.js 16 requires Node.js 20.9 or newer

- The standard creation command is:

    ```bash
    npx create-next-app@latest my-app
    cd my-app
    npm run dev
    ```

- The recommended defaults create a TypeScript application with ESLint, Tailwind CSS, the App Router, Turbopack, and the `@/*` import alias

- The core packages are `next`, `react`, and `react-dom`

- Standard `package.json` scripts:

    ```JSON
    {
        "scripts": {
            "dev": "next dev",
            "build": "next build",
            "start": "next start",
            "lint": "eslint",
            "lint:fix": "eslint --fix"
        }
    }
    ```

- `next dev`: Starts the development server with Fast Refresh. Turbopack is the default bundler

- `next build`: Creates an optimized production build and reports which routes are static or dynamic

- `next start`: Runs a completed production build. `next build` must run first

- `next info`: Prints system and dependency information useful for bug reports

- `next telemetry disable`: Disables anonymous Next.js telemetry

- `create-next-app`: Scaffolds a project and may also be used through package managers other than npm, including pnpm, Yarn, and Bun

- `@next/codemod`: Performs official mechanical migrations for renamed or changed APIs. Review its diff and run tests instead of assuming every migration decision can be automated

- A typical major-version upgrade is started with `npx @next/codemod@canary upgrade latest`, followed by reading the matching upgrade guide

- Add `--webpack` to `next dev` or `next build` when a project specifically requires Webpack

- Development runs at `http://localhost:3000` by default. Use `next dev -p 4000` for another port

# Project Structure

- Next.js uses file-system routing. Folder and file names inside `app` determine the URLs and behavior of the application

- A common structure is:

    ```text
    my-app/
    |-- app/
    |   |-- api/
    |   |-- dashboard/
    |   |-- globals.css
    |   |-- layout.tsx
    |   |-- loading.tsx
    |   |-- not-found.tsx
    |   `-- page.tsx
    |-- components/
    |-- lib/
    |-- public/
    |-- next.config.ts
    |-- next-env.d.ts
    |-- package.json
    |-- proxy.ts
    `-- tsconfig.json
    ```

- `app`: App Router routes, layouts, loading states, errors, and route handlers

- `components`: Reusable React components. This name is a convention, not a special Next.js folder

- `lib`: Database clients, validation, authentication, data access, and utility functions. This is also a normal convention

- `public`: Static files available from the root URL. `public/logo.png` is requested as `/logo.png`

- `next.config.ts`: Framework configuration

- `next-env.d.ts`: Generated TypeScript declarations. Do not manually edit it

- `proxy.ts`: Request-time logic that runs before matching routes

- `.env`, `.env.local`, `.env.development`, `.env.production`: Environment variable files

- The entire source tree may be placed inside an optional `src` folder. Configuration files and `public` remain at the project root

- Files may be safely colocated inside `app`. A file only becomes public when a `page.tsx` or `route.ts` exposes its route segment

- Prefixing a folder with `_`, such as `_components`, makes it a private folder which is excluded from routing

- The `@/*` alias normally maps to the project root or `src`, allowing `import Button from '@/components/Button'` instead of long relative imports

# Routing Fundamentals

- Each folder inside `app` is a route segment. A `page.tsx` file makes the segment publicly accessible

    ```text
    app/page.tsx                  -> /
    app/about/page.tsx            -> /about
    app/blog/page.tsx             -> /blog
    app/blog/create/page.tsx      -> /blog/create
    ```

- A page is the leaf UI for a URL. It must default-export a React component

    ```tsx
    export default function AboutPage() {
        return <h1>About</h1>
    }
    ```

- A layout wraps pages and nested layouts. Layouts persist between navigations, preserve state, and do not rerender from the beginning on every child navigation

- The root `app/layout.tsx` is required and must contain `<html>` and `<body>`

    ```tsx
    export default function RootLayout({
        children,
    }: Readonly<{ children: React.ReactNode }>) {
        return (
            <html lang="en">
                <body>{children}</body>
            </html>
        )
    }
    ```

- A nested `app/dashboard/layout.tsx` only wraps routes inside `/dashboard`

- A `template.tsx` also wraps children, but it receives a new identity during navigation. Its state resets and its effects rerun. Use layouts for persistent UI and templates when the wrapper must remount

- Route groups use parentheses and organize routes without changing the URL:

    ```text
    app/(marketing)/about/page.tsx -> /about
    app/(shop)/cart/page.tsx       -> /cart
    ```

- Route groups can provide different layouts to unrelated areas. Two groups cannot create the same final URL

- Multiple root layouts are possible by placing root layouts in different route groups. Navigating between different root layouts causes a full page load

# Dynamic Routes

- Square brackets create dynamic segments:

    ```text
    app/blog/[slug]/page.tsx           -> /blog/hello-world
    app/shop/[category]/[item]/page.tsx -> /shop/shoes/123
    ```

- In current App Router pages, `params` and `searchParams` are Promises and should be awaited

    ```tsx
    type Props = {
        params: Promise<{ slug: string }>
        searchParams: Promise<{
            page?: string | string[]
            sort?: string | string[]
        }>
    }

    export default async function BlogPost({ params, searchParams }: Props) {
        const { slug } = await params
        const { page: rawPage = '1' } = await searchParams
        const page = Array.isArray(rawPage) ? rawPage[0] : rawPage

        return <p>{slug}, page {page}</p>
    }
    ```

- `[...slug]` is a catch-all segment. `/docs/a/b` produces `{ slug: ['a', 'b'] }`

- `[[...slug]]` is optional catch-all. It also matches the parent route where `slug` is missing

- `generateStaticParams` returns known dynamic parameters to prerender during the build

    ```tsx
    export async function generateStaticParams() {
        const posts = await getPosts()
        return posts.map((post) => ({ slug: post.slug }))
    }
    ```

- `dynamicParams = false` makes unknown parameters return `404` instead of being generated when first requested

- Page props can be typed manually or with generated helpers such as `PageProps<'/blog/[slug]'>` after Next.js has generated route types

# Navigation

- `Link` from `next/link` is the standard navigation component. It performs client-side navigation, preserves shared layouts, and can prefetch routes

    ```tsx
    import Link from 'next/link'

    export default function Nav() {
        return <Link href="/dashboard">Dashboard</Link>
    }
    ```

- Use a normal `<a>` for external URLs and downloads. Use `Link` for internal routes

- Static routes are normally prefetched when a visible `Link` approaches the viewport. Dynamic routes may be partially prefetched when they have a loading boundary

- Set `prefetch={false}` when prefetching is too costly. Setting `prefetch={true}` forces a full prefetch where supported

- `replace` replaces the current browser history entry. `scroll={false}` prevents the default scroll behavior

- `useRouter` performs navigation from a Client Component:

    ```tsx
    'use client'

    import { useRouter } from 'next/navigation'

    export default function SaveButton() {
        const router = useRouter()

        return <button onClick={() => router.push('/done')}>Finish</button>
    }
    ```

- `router.push(url)`: Navigate and add a history entry

- `router.replace(url)`: Navigate without adding a history entry

- `router.back()` / `router.forward()`: Move through browser history

- `router.refresh()`: Request a new Server Component payload and merge it into the current UI without losing unaffected browser state

- `router.prefetch(url)`: Manually prefetch a route

- Never pass untrusted input into `router.push` or `router.replace`. A malicious `javascript:` URL can execute code

- `redirect('/login')` redirects from a Server Component, Route Handler, or Server Action by throwing a framework-controlled redirect response

- `permanentRedirect('/new-url')` uses a permanent redirect. Configuration-level redirects are better for large static redirect lists

- Client navigation can display a pending indicator through the `useLinkStatus` hook inside a custom child of `Link`

# Server and Client Components

- Components in the App Router are Server Components by default

- A Server Component executes on the server, can directly access databases and private environment variables, sends less JavaScript to the browser, and cannot use browser APIs or interactive React hooks

    ```tsx
    export default async function UsersPage() {
        const users = await db.user.findMany()

        return users.map((user) => <p key={user.id}>{user.name}</p>)
    }
    ```

- A Client Component starts with the exact `'use client'` directive before imports

    ```tsx
    'use client'

    import { useState } from 'react'

    export default function Counter() {
        const [count, setCount] = useState(0)
        return <button onClick={() => setCount(count + 1)}>{count}</button>
    }
    ```

- Use Client Components for state, event handlers, effects, browser APIs, custom hooks using these features, and context providers

- `'use client'` creates a client boundary. The component and modules it imports become part of the client bundle. Do not add it to every file

- Props passed from a Server Component into a Client Component must be serializable. Strings, numbers, booleans, arrays, plain objects, and supported server references work; normal functions, class instances, and database connections do not

- Server Components may render Client Components. A useful pattern is a mostly server-rendered page containing small interactive islands

- A Client Component cannot directly import a Server Component. It can receive already-rendered Server Component content through `children` or another prop

    ```tsx
    // Server Component
    <Modal><ServerCart /></Modal>
    ```

- Mark server-only modules with `import 'server-only'`. This produces a build error if client code imports them

- Mark browser-only modules with `import 'client-only'`

- Place providers as deep as practical so Next.js can optimize the static server-rendered area above them

# Rendering, Hydration, and Streaming

- Server rendering creates an initial representation on the server. Hydration attaches Client Component JavaScript to the existing HTML so it becomes interactive

- Server Components are sent as a React Server Component payload. Next.js uses this payload together with Client Component JavaScript to render the browser interface

- Static rendering happens before a user request, normally during the build or cache generation. It is best for content shared by every user

- Dynamic rendering happens at request time. It is required when output depends on cookies, headers, search parameters, uncached request data, or other request-specific information

- `connection()` from `next/server` waits for a real request and opts everything below it into request-time rendering

- Streaming sends completed parts of a route before slower parts finish. Users see useful UI without waiting for the entire page

- `loading.tsx` creates an automatic Suspense boundary around the route segment and provides immediate fallback UI during navigation

    ```tsx
    export default function Loading() {
        return <p>Loading dashboard...</p>
    }
    ```

- `<Suspense fallback={...}>` creates more precise streaming boundaries around individual slow components

    ```tsx
    import { Suspense } from 'react'

    export default function Page() {
        return (
            <>
                <ProductDetails />
                <Suspense fallback={<p>Loading reviews...</p>}>
                    <Reviews />
                </Suspense>
            </>
        )
    }
    ```

- Suspense only provides a useful stream when the suspended work happens inside the boundary, not before the component is returned

- Partial Prerendering is implemented through Cache Components. It combines a prerendered shell with streamed dynamic sections in the same route

# Data Fetching

- Fetch data in Server Components whenever possible. This keeps credentials and database code off the browser and prevents an unnecessary request from the browser to your own server

- Next.js extends server-side `fetch` with framework caching and revalidation options

    ```tsx
    export default async function Page() {
        const response = await fetch('https://api.example.com/products')

        if (!response.ok) {
            throw new Error('Failed to load products')
        }

        const products = await response.json()
        return <ProductList products={products} />
    }
    ```

- Always check `response.ok`. `fetch` does not throw automatically for HTTP errors such as `404` or `500`

- Call an ORM or database client directly from a Server Component or server-only data access function. Creating a Route Handler merely to call it from the same Server Component adds an unnecessary network step

- Independent requests should start in parallel:

    ```tsx
    const userPromise = getUser()
    const postsPromise = getPosts()
    const [user, posts] = await Promise.all([userPromise, postsPromise])
    ```

- Sequential requests are correct when one result is needed to start the next request

- React `cache()` memoizes a server function for the lifetime of a render, preventing identical calls in one render tree from repeating. It is request memoization, not a persistent Next.js data cache

- Use `preload` functions to begin predictable work early without passing promises through many components

- Client-side fetching is appropriate for frequently changing browser-only data. Common tools include SWR and TanStack Query, or a normal effect for simple cases

- Avoid request waterfalls. Fetch at the server boundary, parallelize unrelated work, and stream slow sections

# Caching and Revalidation

- Caching is one of the most version-sensitive parts of Next.js. Do not assume an old tutorial's default behavior still applies

- The main cache layers are:

    Request memoization: Deduplicates identical `GET` fetch calls during one server render

    Data/Components cache: Reuses explicitly cached data or component output across requests

    Full route output: Reuses prerendered HTML and Server Component payloads

    Client router cache: Reuses visited or prefetched route segments in the browser

- In the modern model, dynamic data should be assumed fresh unless it is deliberately cached

- `fetch(url, { cache: 'no-store' })` requests fresh data

- `fetch(url, { cache: 'force-cache' })` opts the response into persistent caching where supported

- `fetch(url, { next: { revalidate: 3600, tags: ['products'] } })` associates time-based revalidation and cache tags with the response

- Cache Components is opt-in and enabled in `next.config.ts`:

    ```tsx
    import type { NextConfig } from 'next'

    const nextConfig: NextConfig = {
        cacheComponents: true,
    }

    export default nextConfig
    ```

- `'use cache'` caches the output of an async function, component, or file. Its cache key includes the build, function identity, arguments, and referenced serializable values

    ```tsx
    import { cacheLife, cacheTag } from 'next/cache'

    export async function getProducts() {
        'use cache'
        cacheLife('hours')
        cacheTag('products')

        return db.product.findMany()
    }
    ```

- `cacheLife` sets a cache profile. Built-in profiles include `seconds`, `minutes`, `hours`, `days`, `weeks`, and `max`

- A profile controls:

    `stale`: How long the client may reuse content without checking the server

    `revalidate`: When a request may trigger background regeneration

    `expire`: When stale output can no longer be used and fresh output must be generated

- `cacheTag('products')` gives cached output a name that mutations can invalidate

- `'use cache: private'` allows private runtime request data in a short-lived private cache. It requires a configured private cache profile and should only be used when its privacy behavior is understood

- `'use cache: remote'` uses a configured remote cache handler for persistent shared caching across server instances

- `revalidatePath('/products')` marks a page or layout path for revalidation

- `revalidateTag('products', 'max')` uses stale-while-revalidate behavior: cached data is served while fresh data is generated in the background

- `updateTag('products')` expires a tag immediately and is intended for Server Actions when the same user must read their own update immediately

- `refresh()` from `next/cache` refreshes the client router from a Server Action. It does not invalidate server caches by itself

- `unstable_cache` is the older API for caching database or non-`fetch` functions. Prefer Cache Components and `'use cache'` in new Next.js 16 code

- `unstable_noStore` is also legacy. Prefer fresh uncached work or `connection()` depending on the goal

- Never cache personalized data into a globally shared entry. User identity must either be outside the cached scope, part of its key, or handled by an appropriate private cache

# Server Actions, Mutations, and Forms

- A Server Action is an async server function callable through a form or Client Component. It is used for mutations, not general data fetching

- Add `'use server'` inside a function or at the top of a dedicated actions file

    ```tsx
    // app/actions.ts
    'use server'

    import { revalidatePath } from 'next/cache'
    import { redirect } from 'next/navigation'

    export async function createPost(formData: FormData) {
        const title = formData.get('title')

        if (typeof title !== 'string' || title.trim() === '') {
            throw new Error('A title is required')
        }

        await db.post.create({ data: { title } })
        revalidatePath('/posts')
        redirect('/posts')
    }
    ```

    ```tsx
    import { createPost } from '@/app/actions'

    export default function Form() {
        return (
            <form action={createPost}>
                <input name="title" />
                <button type="submit">Create</button>
            </form>
        )
    }
    ```

- Forms calling Server Actions work through progressive enhancement. A server-rendered form can submit before the client JavaScript finishes loading

- Treat every Server Action as a public endpoint. Validate input and perform authentication and authorization inside the action

- Use a schema library such as Zod when validation is complex. TypeScript types disappear at runtime and cannot validate hostile input

- `useActionState` returns the action result and pending state, useful for validation messages

    ```tsx
    'use client'

    import { useActionState } from 'react'
    import { createUser } from './actions'

    export default function Signup() {
        const [state, action, pending] = useActionState(createUser, null)

        return (
            <form action={action}>
                <input name="email" type="email" />
                {state?.error && <p>{state.error}</p>}
                <button disabled={pending}>Sign up</button>
            </form>
        )
    }
    ```

- `useFormStatus` reads the status of the nearest parent form and is useful for reusable submit buttons

- `useOptimistic` displays the expected result immediately while the real mutation is still running. The server result remains the source of truth

- `.bind()` can attach additional trusted identifiers to an action, but they are still client-provided request values and must be authorized on the server

- Server Actions use `POST`. Their arguments and return values must be serializable

- Use `after()` for work that should run after the response is finished, such as logging or analytics. Do not move required business logic into `after()` because the user may already have received success

# Route Handlers

- A `route.ts` file creates an HTTP endpoint using the Web Request and Response APIs

    ```tsx
    // app/api/users/route.ts
    import { NextResponse } from 'next/server'

    export async function GET() {
        const users = await db.user.findMany()
        return NextResponse.json(users)
    }

    export async function POST(request: Request) {
        const body = await request.json()
        const user = await db.user.create({ data: body })
        return NextResponse.json(user, { status: 201 })
    }
    ```

- Supported method exports are `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, and `OPTIONS`. An unsupported method returns `405 Method Not Allowed`

- A route cannot contain both `page.tsx` and `route.ts` at the same segment because both would own the same URL

- Dynamic handler parameters are awaited:

    ```tsx
    export async function GET(
        request: Request,
        { params }: { params: Promise<{ id: string }> },
    ) {
        const { id } = await params
        return Response.json({ id })
    }
    ```

- Use `request.json()`, `request.formData()`, `request.text()`, `request.arrayBuffer()`, and `new URL(request.url).searchParams` to read input

- `NextRequest` adds convenient `cookies`, `nextUrl`, and user-agent-related behavior to the standard Request API

- `NextResponse` adds `json`, `redirect`, `rewrite`, `next`, and cookie helpers to the standard Response API

- Route Handlers can return streams, files, XML, webhooks, RSS, and non-UI responses

- `GET` handlers are not automatically a permanent global cache. Explicitly choose cache behavior

- CORS headers must be added to the response or configured through `next.config.ts`. Browser same-origin rules still apply

- Use Route Handlers when external clients need HTTP access. Use Server Actions when a mutation only belongs to the Next.js interface

# Proxy

- `proxy.ts` runs before a request reaches the route. It replaces the former `middleware.ts` convention in Next.js 16 and uses the Node.js runtime

- Proxy is useful for redirects, rewrites, locale detection, experiments, header changes, and fast optimistic session checks

    ```tsx
    import { NextRequest, NextResponse } from 'next/server'

    export function proxy(request: NextRequest) {
        const session = request.cookies.get('session')

        if (!session && request.nextUrl.pathname.startsWith('/dashboard')) {
            return NextResponse.redirect(new URL('/login', request.url))
        }

        return NextResponse.next()
    }

    export const config = {
        matcher: ['/dashboard/:path*'],
    }
    ```

- Proxy runs frequently, including around prefetched requests. Avoid slow database queries and heavy business logic

- Proxy is not the final security boundary. Repeat secure authorization near the database or mutation

- `NextResponse.rewrite(url)` serves another destination while keeping the visible URL unchanged

- `NextResponse.redirect(url)` changes the browser destination

- Matchers statically select paths. Exclude static assets, image optimization, APIs, or metadata when they do not need proxy work

# Errors and HTTP Outcomes

- Throw a normal error for unexpected failures. The nearest `error.tsx` boundary handles errors in its route segment

- `error.tsx` must be a Client Component because it receives the error and a reset function

    ```tsx
    'use client'

    export default function ErrorPage({
        error,
        reset,
    }: {
        error: Error & { digest?: string }
        reset: () => void
    }) {
        return (
            <main>
                <h2>Something went wrong</h2>
                <button onClick={reset}>Try again</button>
            </main>
        )
    }
    ```

- Production Server Component error messages are hidden from the browser to avoid leaking sensitive details. The digest can help match the displayed failure to server logs

- `global-error.tsx` handles failures in the root layout or root template and must provide its own `<html>` and `<body>` tags

- `notFound()` stops rendering the segment and displays the nearest `not-found.tsx`

    ```tsx
    const post = await getPost(slug)
    if (!post) notFound()
    ```

- A root `not-found.tsx` also handles unmatched application URLs

- `forbidden()` with `forbidden.tsx` produces a `403` UI, while `unauthorized()` with `unauthorized.tsx` produces a `401` UI. These conventions are experimental in the current baseline and should not be relied on without checking the installed version

- Expected validation errors should be returned as action state, not thrown as unexpected exceptions

- Do not catch `redirect()`, `notFound()`, or similar control-flow APIs unless the error is rethrown. They work by throwing special internal errors

- `unstable_rethrow(error)` rethrows Next.js internal control-flow errors from a broad catch block

# CSS and Styling

- Global CSS is normally imported from the root layout. It applies to the entire application

    ```tsx
    import './globals.css'
    ```

- CSS Modules use `*.module.css` and locally scope class names

    ```css
    /* button.module.css */
    .primary {
        background: navy;
        color: white;
    }
    ```

    ```tsx
    import styles from './button.module.css'

    export default function Button() {
        return <button className={styles.primary}>Save</button>
    }
    ```

- Tailwind CSS generates utility styles from classes and is offered by `create-next-app`

- Sass works after installing `sass`. Next.js supports `.scss`, `.sass`, and Sass Modules

- CSS-in-JS libraries differ in Server Component support. Libraries requiring runtime style injection usually need a Client Component registry

- Next.js orders styles according to JavaScript import order. Keep imports predictable and avoid accidentally depending on development-only ordering

# Built-in Components

- `next/link`: Optimized internal navigation and prefetching

- `next/image`: Image sizing, lazy loading, responsive loading, and format/quality optimization

    ```tsx
    import Image from 'next/image'
    import profile from '@/public/profile.jpg'

    export default function Avatar() {
        return <Image src={profile} alt="Jane Doe" placeholder="blur" />
    }
    ```

- Local static image imports automatically know width, height, and blur information. Remote images need width/height or `fill`, and their host/path must be allowed in `images.remotePatterns`

- Always provide meaningful `alt` text, or `alt=""` for a purely decorative image

- `fill` makes the image fill its positioned parent. Supply a correct `sizes` string or the browser may download an unnecessarily large image

- `priority` or the newer preload behavior should be limited to important above-the-fold images such as the Largest Contentful Paint image

- `next/font`: Self-hosts and optimizes Google or local fonts, removes external font requests, and reduces layout shift

    ```tsx
    import { Inter } from 'next/font/google'

    const inter = Inter({ subsets: ['latin'] })

    export default function Layout({ children }: { children: React.ReactNode }) {
        return <body className={inter.className}>{children}</body>
    }
    ```

- `next/script`: Controls third-party script loading

- Script strategies:

    `beforeInteractive`: Critical site-wide scripts needed before hydration

    `afterInteractive`: Load soon after hydration; this is the common default

    `lazyOnload`: Load during browser idle time

    `worker`: Experimental off-main-thread loading with limitations

- `next/form`: A form component optimized for navigation-style `GET` forms. It prefetches the destination and performs client navigation while preserving form semantics

- Normal HTML `<form>` is still used for Server Actions and ordinary submissions

# Metadata, SEO, and Social Sharing

- Export a static `metadata` object from a layout or page when values are known beforehand

    ```tsx
    import type { Metadata } from 'next'

    export const metadata: Metadata = {
        title: {
            default: 'My Store',
            template: '%s | My Store',
        },
        description: 'A store for useful products',
    }
    ```

- Export `generateMetadata` when metadata depends on parameters or fetched data

    ```tsx
    export async function generateMetadata({ params }: Props): Promise<Metadata> {
        const { slug } = await params
        const post = await getPost(slug)

        return {
            title: post.title,
            description: post.summary,
        }
    }
    ```

- Metadata exports only work in Server Components. Metadata is merged through the route hierarchy

- Set `metadataBase` when relative Open Graph, canonical, or social image URLs need an absolute base

- Metadata file conventions include:

    `favicon.ico`, `icon.*`, `apple-icon.*`: Browser and device icons

    `opengraph-image.*`, `twitter-image.*`: Social share images

    `robots.txt` or `robots.ts`: Crawler rules

    `sitemap.xml` or `sitemap.ts`: Search-engine URL index

    `manifest.json` or `manifest.ts`: Web application manifest

- `ImageResponse` generates dynamic social images with JSX

- `generateImageMetadata` creates multiple dynamic images for one route

- `generateSitemaps` splits very large sites into multiple sitemaps

- `generateViewport` dynamically creates viewport and theme-color information. Normal SEO fields belong in metadata, not viewport

- Add JSON-LD using a `<script type="application/ld+json">`. Serialize a trusted object and replace `<` to reduce injection risk

- Good SEO also requires semantic HTML, useful page content, accessible links, fast loading, canonical URLs, and correct status codes. Metadata alone cannot compensate for a poor page

# Static Files, Images, and MDX

- Files in `public` are served from `/`. Their names should be treated as public and stable

- Public files are not automatically imported or processed. Use static imports when build-time image metadata is desired

- MDX allows Markdown containing JSX. Install `@next/mdx`, configure page extensions, and create `mdx-components.tsx` at the project root

- MDX is useful for documentation and content sites. Remote MDX must be trusted or securely compiled because JSX can execute server code

- Dynamic imports reduce initial JavaScript:

    ```tsx
    import dynamic from 'next/dynamic'

    const Chart = dynamic(() => import('./Chart'), {
        loading: () => <p>Loading chart...</p>,
        ssr: false,
    })
    ```

- Use `ssr: false` only for truly browser-only Client Components. It cannot be used to disable server rendering from a Server Component

# Cookies, Headers, and Draft Mode

- `cookies()` reads incoming cookies in Server Components and can write cookies in Server Actions or Route Handlers

    ```tsx
    import { cookies } from 'next/headers'

    const cookieStore = await cookies()
    const theme = cookieStore.get('theme')?.value
    ```

- Cookie writes should set secure attributes:

    ```tsx
    cookieStore.set('session', token, {
        httpOnly: true,
        secure: process.env.NODE_ENV === 'production',
        sameSite: 'lax',
        path: '/',
        maxAge: 60 * 60 * 24 * 7,
    })
    ```

- `headers()` reads request headers. Both `cookies()` and `headers()` use request-time data and affect rendering behavior

- Headers and cookies are untrusted client input. They must not be treated as proof of identity without cryptographic or server-side verification

- `draftMode()` uses a cookie to bypass normal published-content behavior so a CMS can preview drafts. Enable and disable it through secure Route Handlers with a secret token

- `userAgent(request)` parses browser, device, OS, engine, CPU, and bot details from a request. User-agent strings are hints and can be spoofed

# Authentication, Authorization, and Data Security

- Authentication verifies identity. Session management remembers the identity. Authorization checks what that identity may do

- A common flow is:

    Submit credentials -> Validate input -> Verify identity -> Create encrypted session -> Set secure cookie -> Check authorization near protected data

- Prefer a maintained authentication library for OAuth, password resets, CSRF protection, provider behavior, and session rotation

- Use a Data Access Layer for secure queries. Each exported function should verify the current session and return only the fields the caller needs

- Use Data Transfer Objects to avoid sending complete database rows, password hashes, private emails, or internal permissions into the UI

- Hiding a button is not authorization. Server Actions, Route Handlers, and database access must independently verify permission

- Layout-only authorization is unsafe because layouts do not necessarily rerender on every client navigation. Check close to the page and data operation

- Proxy may quickly redirect obviously unauthenticated users, but the secure check belongs near the protected resource

- React taint APIs can prevent sensitive objects or values from crossing the Server-to-Client boundary when the relevant Next.js configuration is enabled

- Prevent SQL injection with parameterized queries or a safe ORM. Prevent XSS by letting React escape text and avoiding untrusted `dangerouslySetInnerHTML`

- Validate file type, content, size, and destination for uploads. A browser-provided MIME type and file name are not proof of safety

- Add a Content Security Policy for strong XSS defense. Also consider HSTS, `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`, and frame restrictions

- Keep dependencies updated and audit production packages. Secrets should be rotated after accidental exposure, not merely removed from Git history

# Environment Variables

- Next.js automatically loads `.env*` files into `process.env`

    ```bash
    DATABASE_URL="postgresql://..."
    SESSION_SECRET="..."
    NEXT_PUBLIC_ANALYTICS_ID="..."
    ```

- Unprefixed variables are server-only. Variables beginning with `NEXT_PUBLIC_` are inserted into browser JavaScript during the build and are permanently public

- Never put database credentials, private API keys, or session secrets in `NEXT_PUBLIC_` variables

- Public variables are frozen at build time. Promoting one completed build between environments does not change their bundled values

- `.env.local` normally contains machine-specific secrets and should be ignored by Git. `.env.example` should document required names without real values

- The `@next/env` package can load Next.js environment behavior in external configuration or test code

- Validate required environment variables at startup so missing configuration fails clearly

# Advanced Routing

- Parallel Routes use named slots beginning with `@` to render independently navigable sections inside one layout

    ```text
    app/dashboard/@analytics/page.tsx
    app/dashboard/@team/page.tsx
    app/dashboard/layout.tsx
    ```

- Slots are passed to the parent layout as props. They do not appear in the URL

- `default.tsx` provides a fallback for a parallel slot when Next.js cannot recover that slot's active state during a full page load

- Intercepting Routes display another route inside the current layout while preserving context. They are commonly combined with Parallel Routes for shareable modals

- Interception notation is based on route segments, not filesystem levels:

    `(.)photo`: Same level

    `(..)photo`: One segment above

    `(..)(..)photo`: Two segments above

    `(...)photo`: From the application root

- Soft navigation to an intercepted photo can show a modal, while directly opening or refreshing its URL shows the full photo page

- `useSelectedLayoutSegment()` returns the active child segment. `useSelectedLayoutSegments()` returns all active segments below a layout

- `usePathname()` returns the current pathname in a Client Component

- `useSearchParams()` returns read-only `URLSearchParams` in a Client Component. Use page `searchParams` for server-side data fetching

- `useParams()` reads dynamic route parameters in a Client Component

- Rewrites map one incoming URL to another destination without changing the visible URL. Redirects change the visible URL and status response

- Internationalized routing is commonly implemented with a top-level `[lang]` segment, locale detection in Proxy, localized dictionaries, and `lang` on the root HTML

- Multi-zones combine multiple independently deployed Next.js applications under one domain. Navigating between zones is a hard navigation

# Runtime API Reference

- Main directives:

    `'use client'`: Creates a Client Component boundary

    `'use server'`: Exposes server functions as callable Server Actions

    `'use cache'`: Caches function or component output

    `'use cache: private'`: Uses a private runtime-aware cache

    `'use cache: remote'`: Uses a remote cache handler

- Main server functions:

    `after`: Schedule work after the response

    `cacheLife`: Choose cached output lifetime

    `cacheTag`: Tag cached output

    `connection`: Wait for a request and force request-time rendering below the call

    `cookies`: Read or mutate cookies in allowed server contexts

    `headers`: Read request headers

    `draftMode`: Read, enable, or disable draft mode

    `redirect` / `permanentRedirect`: End the current render with a redirect

    `notFound`: End the current segment with a `404`

    `forbidden` / `unauthorized`: Experimental `403` and `401` outcomes

    `revalidatePath`: Invalidate by route path

    `revalidateTag`: Stale-while-revalidate by cache tag

    `updateTag`: Immediately expire a cache tag in a Server Action

    `refresh`: Refresh the client router from a Server Action

    `userAgent`: Parse request user-agent information

- Main generation functions:

    `generateStaticParams`: List dynamic paths to prerender

    `generateMetadata`: Create route metadata from data

    `generateViewport`: Create dynamic viewport information

    `generateImageMetadata`: Generate variants for image metadata files

    `generateSitemaps`: Generate multiple sitemap files

    `ImageResponse`: Generate an image response with JSX

- Main Client hooks from `next/navigation`:

    `useRouter`, `usePathname`, `useSearchParams`, `useParams`, `useSelectedLayoutSegment`, `useSelectedLayoutSegments`

- `useReportWebVitals` receives measurements such as LCP, CLS, INP, FCP, and TTFB for analytics

- `NextRequest` and `NextResponse` extend standard Web Request and Response objects. Prefer the standard objects when their features are sufficient

# File Convention Reference

- `page.tsx`: Public route UI

- `layout.tsx`: Persistent shared UI

- `template.tsx`: Shared UI that remounts on navigation

- `loading.tsx`: Route Suspense fallback

- `error.tsx`: Segment error boundary

- `global-error.tsx`: Root layout error boundary

- `not-found.tsx`: Not-found UI

- `forbidden.tsx`: Experimental forbidden UI

- `unauthorized.tsx`: Experimental unauthorized UI

- `route.ts`: HTTP Route Handler

- `default.tsx`: Parallel route fallback

- `proxy.ts`: Logic before route handling

- `instrumentation.ts`: Server instrumentation registration and request error hook

- `instrumentation-client.ts`: Early client-side instrumentation

- `mdx-components.tsx`: Required MDX component mapping

- `[id]`: Dynamic segment

- `[...slug]`: Catch-all segment

- `[[...slug]]`: Optional catch-all segment

- `(group)`: Organizational route group excluded from the URL

- `@slot`: Parallel route slot excluded from the URL

- `_folder`: Private non-route folder

- `(.)`, `(..)`, `(...)`: Intercepting route notation

- Metadata files: `favicon`, `icon`, `apple-icon`, `opengraph-image`, `twitter-image`, `robots`, `sitemap`, and `manifest`

# Route Segment Configuration

- Segment options are exported from a page, layout, or Route Handler

    ```tsx
    export const dynamic = 'force-dynamic'
    export const revalidate = 3600
    export const runtime = 'nodejs'
    export const preferredRegion = 'home'
    ```

- `dynamic`: `'auto'`, `'force-dynamic'`, `'error'`, or `'force-static'`

- `dynamicParams`: Whether dynamic paths not returned by `generateStaticParams` may be generated

- `revalidate`: Default route revalidation interval where the installed caching model supports it

- `fetchCache`: Advanced override for default server `fetch` caching behavior

- `runtime`: Usually `'nodejs'`; `'edge'` provides a restricted web-standard runtime where supported

- `preferredRegion`: Deployment hint for where server work should run

- `maxDuration`: Deployment hint for maximum execution time

- With Cache Components enabled, several older segment-level caching controls are disabled or replaced by component-level cache choices. Prefer local, explicit caching over forcing an entire route

# next.config.ts Reference

- A typed configuration begins as:

    ```tsx
    import type { NextConfig } from 'next'

    const nextConfig: NextConfig = {
        reactStrictMode: true,
        typedRoutes: true,
    }

    export default nextConfig
    ```

- Common routing options:

    `basePath`: Serve the application below a path such as `/docs`

    `trailingSlash`: Consistently add trailing slashes

    `redirects()`: Configure request redirects

    `rewrites()`: Configure internal or external URL rewrites

    `headers()`: Add response headers by path pattern

- Common build/output options:

    `output: 'standalone'`: Produce a minimal deployable Node.js server bundle

    `output: 'export'`: Produce a static export with limited server features

    `distDir`: Change the `.next` build directory

    `compress`: Control built-in gzip compression for `next start`

    `productionBrowserSourceMaps`: Generate browser source maps in production

    `generateBuildId`: Control the deployment build identifier

- Common compiler/bundler options:

    `reactCompiler`: Enable the React Compiler

    `turbopack`: Configure Turbopack aliases, extensions, and compatible loaders

    `webpack`: Customize Webpack when explicitly using Webpack

    `transpilePackages`: Transpile local monorepo or external packages

    `serverExternalPackages`: Leave selected server dependencies external to bundling

    `optimizePackageImports`: Optimize imports from large libraries

    `experimental.optimizeCss` or `useLightningcss`: Version-dependent CSS optimizations

- Common framework options:

    `cacheComponents`: Enable Cache Components and PPR behavior

    `cacheLife`: Define custom cache profiles

    `cacheHandlers`: Configure remote/private cache handlers

    `images`: Configure remote patterns, formats, sizes, qualities, loaders, and optimization behavior

    `typedRoutes`: Type-check statically known `Link` and router destinations

    `serverActions`: Configure action body limits and allowed origins

    `staleTimes`: Tune client router cache timing

    `reactStrictMode`: Enable React development checks; the App Router already uses strict behavior by default

    `typescript`: Configure TypeScript build checking. Ignoring build errors is dangerous

    `logging`: Tune development logging

    `poweredByHeader`: Remove the `X-Powered-By` header when false

    `crossOrigin`: Add a cross-origin attribute to generated script tags

    `assetPrefix`: Serve framework assets from another location or CDN

    `allowedDevOrigins`: Permit additional origins to access the development server

    `devIndicators`: Configure development indicators

    `onDemandEntries`: Tune development page retention

    `isolatedDevBuild`: Keep development output separate from production build output

    `proxyClientMaxBodySize`: Limit request bodies forwarded through Proxy

    `sassOptions`: Pass configuration to Sass

    `webVitalsAttribution`: Add detail to reported Web Vitals

    `viewTransition`: Enable experimental React View Transition integration

- The complete config surface changes frequently. Only add options needed by the project and verify experimental options against the installed Next.js version

# TypeScript, ESLint, and Code Quality

- Next.js has first-class TypeScript support. Renaming a file to `.ts` or `.tsx` and running the dev server creates required configuration

- Run `next typegen` when generated route, page, layout, and environment types are needed without a full build

- Typed routes catch many invalid literal links at compile time when `typedRoutes` is enabled

- Use `import type` for types and type all page parameters, action state, API responses, and reusable component props

- Do not silence TypeScript build errors globally. Fix the type or isolate a precisely explained exception

- Next.js provides `@next/eslint-plugin-next` rules for common framework mistakes. Next.js 16 no longer includes the `next lint` command and `next build` no longer performs linting automatically

- Run ESLint or Biome directly in development and CI

- Formatters improve consistency but do not replace linting, type checking, or tests

# Testing

- Unit tests verify isolated functions and simple components

- Component tests verify rendered component behavior

- Integration tests verify cooperating modules

- End-to-End tests verify complete user behavior in a browser

- Common tools:

    Jest: Unit and snapshot testing

    Vitest: Fast unit and component testing with modern ESM support

    React Testing Library: Test component behavior through accessible user-facing queries

    Playwright: End-to-End browser testing

    Cypress: End-to-End and component testing

- Prefer End-to-End tests for async Server Components when the unit-test environment cannot correctly reproduce the React server runtime

- High-value flows include authentication, authorization, forms, payments, redirects, error states, and cache invalidation

- Test production behavior with `next build` and `next start`. Development mode has different caching, rendering, and error behavior

- CI should normally run formatting checks, linting, type checking, unit tests, a production build, and important End-to-End tests

# Debugging and Instrumentation

- Use the development error overlay for render, type, and module failures

- Server Component logs appear in the server terminal. Client Component logs appear in browser DevTools

- React DevTools inspects the component tree, props, state, and rendering behavior

- Next.js development tools expose route, build, cache, and runtime diagnostics through the development interface. These are debugging tools and are not part of the production application

- Next.js 16 also includes an MCP development server for compatible coding agents to inspect current application state and errors. Keep agent instructions and framework documentation current, and never expose development control surfaces as a public production endpoint

- `instrumentation.ts` exports `register()` to initialize server monitoring once per server instance

- It may also export `onRequestError` to report server request failures

- `instrumentation-client.ts` runs before the application becomes interactive and can initialize browser monitoring or observe navigation

- OpenTelemetry provides vendor-neutral traces, metrics, and context. Next.js includes built-in instrumentation hooks and can use `@vercel/otel` for simpler setup

- `useReportWebVitals` sends browser performance measurements to an analytics endpoint

- Useful measurements:

    LCP: Time until the largest visible content element renders

    INP: Responsiveness to user interactions

    CLS: Unexpected visual movement

    FCP: Time until the first content appears

    TTFB: Time until the first response byte arrives

- Use bundle analysis to find oversized Client Component dependency trees. A single unnecessary `'use client'` boundary can move many imports into the browser bundle

# Performance

- Keep components server-side unless interactivity requires the client

- Stream slow sections through Suspense instead of blocking the whole page

- Parallelize independent data access and remove request waterfalls

- Cache shared expensive work with an explicit freshness policy. Do not cache user-specific data globally

- Use `Image`, `Font`, and `Script` correctly. Reserve image space, preload only critical resources, and delay low-priority third-party scripts

- Use dynamic imports for large interactive components not needed immediately

- Limit context providers and large state libraries to the part of the tree that uses them

- Avoid barrel files that accidentally import huge packages into client code. Import narrow entry points where possible

- Measure before optimizing. Lighthouse, browser Performance tools, React DevTools, bundle analysis, server traces, and real-user Web Vitals find different problems

- Cache build output such as `.next/cache` in CI to speed repeated builds, while using a safe key based on lockfiles and source inputs

- Use `server-only` to protect server modules and make accidental client bundle expansion fail early

# Deployment

- Main deployment forms:

    Node.js server: Supports every Next.js feature through `next build` and `next start`

    Docker container: Supports every feature when running the Node.js output correctly

    Static export: Produces only static files and cannot support request-time server features

    Platform adapter: Translates build output for a hosting platform; exact feature support depends on the adapter

- Vercel provides the closest managed integration, but Next.js can run anywhere providing a suitable Node.js server

- Static export configuration:

    ```tsx
    const nextConfig: NextConfig = {
        output: 'export',
        images: { unoptimized: true },
    }
    ```

- Static export cannot use Server Actions, Proxy, request-dependent rendering, ISR requiring a server, or dynamic Route Handlers

- Standalone output is useful for Docker:

    ```tsx
    const nextConfig: NextConfig = {
        output: 'standalone',
    }
    ```

- Self-hosted production should normally place a reverse proxy such as nginx in front of Next.js for TLS, rate limiting, malformed requests, slow connections, and payload limits

- Multiple server instances need coordinated cache storage and tag invalidation to avoid serving different stale versions

- Streaming infrastructure must not buffer the entire response if Server Component and PPR streaming benefits are expected

- Use deployment health checks, structured logs, error monitoring, database connection limits, graceful shutdown, and rollback-ready releases

- Build once and promote the same artifact when possible. Remember that `NEXT_PUBLIC_` values are embedded during that build

# Production Checklist

- Run linting, type checking, tests, and a clean production build

- Verify every environment variable in the production environment

- Confirm authentication and authorization inside every protected mutation and query

- Validate Server Action and Route Handler input

- Check cache behavior using multiple users and after mutations

- Add `loading`, `error`, empty, and not-found states

- Verify metadata, canonical URLs, sitemap, robots rules, icons, and social images

- Test keyboard navigation, labels, focus behavior, contrast, reduced motion, and screen-reader structure

- Review images, fonts, scripts, Client Component boundaries, and bundle size

- Add CSP and appropriate security headers

- Configure monitoring, Web Vitals, request logs, and source maps with controlled access

- Test a production build instead of assuming development behavior is identical

- Verify redirects, rewrites, Proxy matchers, cookies, webhooks, uploads, and rate limits

- Document deployment, migrations, backup recovery, and rollback steps

# Pages Router (Legacy Reference)

- The Pages Router uses the `pages` folder and predates Server Components. Learn it for existing applications, not as the default starting point for new ones

- `pages/index.tsx` maps to `/`

- `pages/about.tsx` maps to `/about`

- `pages/posts/[id].tsx` creates a dynamic route

- `pages/_app.tsx` wraps all pages and is used for global styles and providers

- `pages/_document.tsx` customizes the outer HTML document rendered on the server

- `pages/404.tsx` and `pages/500.tsx` customize error pages

- `pages/api/users.ts` creates a Node-style API Route using `NextApiRequest` and `NextApiResponse`

- `getStaticProps`: Fetch data during the build for a static page

- `getStaticPaths`: List dynamic paths generated with `getStaticProps`

- `getServerSideProps`: Fetch data for every request

- `getInitialProps`: Older universal data API which disables several automatic static optimizations. Avoid in new code

- Returning `revalidate` from `getStaticProps` enables Incremental Static Regeneration

- Pages Router navigation uses `next/router`, not `next/navigation`

- App Router equivalents:

    `getStaticProps` -> Async Server Component plus caching

    `getStaticPaths` -> `generateStaticParams`

    `getServerSideProps` -> Async Server Component using request-time data or `connection()`

    API Routes -> Route Handlers

    `_app` / `_document` -> Root layout

    `next/head` -> Metadata API

    Middleware -> Proxy in Next.js 16

- App Router and Pages Router can coexist during migration, but they cannot own the same URL

# Common Mistakes

- Adding `'use client'` everywhere and turning the application into a large client bundle

- Importing secrets or database code into a Client Component

- Fetching from your own Route Handler inside a Server Component instead of calling the data source directly

- Treating TypeScript as runtime validation

- Trusting IDs, roles, cookies, hidden fields, or bound action values received from the browser

- Assuming a hidden page or button is authorized

- Using Proxy as the only authorization check

- Sequentially awaiting unrelated requests

- Forgetting `response.ok` after `fetch`

- Caching personalized data globally

- Mutating data without revalidating the affected cache or UI

- Catching `redirect()` or `notFound()` and accidentally blocking it

- Using `router.push` with an untrusted URL

- Using `searchParams` for state that should remain local, or local state for information that should be shareable in the URL

- Omitting `sizes` from a responsive `Image` using `fill`

- Loading every third-party script immediately

- Expecting static export to support server-only features

- Testing only development mode

- Copying Pages Router APIs into an App Router project

- Following old caching tutorials without checking the installed Next.js major version

# Mastery Path

1. Build static pages, nested layouts, links, and dynamic segments

2. Understand Server Components before adding Client Components

3. Fetch directly in async Server Components and stream slow sections

4. Build validated forms with Server Actions and visible pending/error states

5. Build Route Handlers for external API access and webhooks

6. Learn the cache layers, then practice `use cache`, tags, and mutation revalidation

7. Add authentication with secure authorization in a Data Access Layer

8. Add metadata, images, fonts, loading states, errors, and accessibility

9. Test with unit tests and complete Playwright user flows

10. Inspect bundles, Web Vitals, server logs, and production caching

11. Deploy both a Node.js build and a restricted static export to understand the difference

12. Study Parallel and Intercepting Routes only after normal routing is automatic

# Quick Decision Reference

- Need reusable non-interactive UI? Server Component

- Need state, effects, event handlers, or browser APIs? Client Component

- Need shared persistent UI? `layout.tsx`

- Need shared UI to reset on navigation? `template.tsx`

- Need a public page? `page.tsx`

- Need an HTTP endpoint for external clients? `route.ts`

- Need an internal UI mutation? Server Action

- Need fresh request-specific work? Uncached data and request-time rendering

- Need shared expensive work reused? `'use cache'` with an explicit `cacheLife`

- Need to invalidate after editing? `updateTag`, `revalidateTag`, or `revalidatePath` based on the desired freshness behavior

- Need immediate fallback UI? `loading.tsx` or `<Suspense>`

- Need expected form errors? Return action state

- Need unexpected error recovery? `error.tsx`

- Need missing-resource UI? `notFound()` and `not-found.tsx`

- Need pre-route redirects or rewrites? `proxy.ts`

- Need SEO values? Static `metadata` or `generateMetadata`

- Need internal navigation? `Link`

- Need optimized media or fonts? `Image` and `next/font`

- Need complete server capability? Node.js or Docker deployment

- Need only static files? `output: 'export'`, after confirming no server features are required

# Official Reference

- Main App Router Documentation: `https://nextjs.org/docs/app`

- App Router API Reference: `https://nextjs.org/docs/app/api-reference`

- Upgrade Guides: `https://nextjs.org/docs/app/guides/upgrading`

- Next.js Repository: `https://github.com/vercel/next.js`

- React Documentation: `https://react.dev`

- Check the installed version with `npm list next` before relying on experimental or version-sensitive behavior
