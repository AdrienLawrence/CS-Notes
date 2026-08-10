# Pathlib
- Pathlib is a module part of the standard library which contains a Path class for turning file paths into objects 

- It is imported with “From pathlib import Path”
With this Path class, file paths are objects of the respective class, as opposed to strings with the previous os.path

- This allows for class methods to be called in place on the directory Path object, shortening code as opposed to using more verbose string operations on a string path

- Use “ / “ to join paths ( “( path / ‘subdir’ / ‘file.txt’)” )

- New inspection methods are available to use on a Path object such as path.exists()

- Parsing methods are also simpler and more concise 

Inspection Methods: 

    .name (The final component of the path, “file.txt”)

	.stem  (The name without its suffix, “file”)

	.suffix (The suffix of the file name, “.txt”)

	.suffixes (List of multiple extensions, archive.tar.gz = “[.tar, .gz]”)

	.parent (The path’s parent directory as a new Path object)

	.parts (A tuple giving access to the path’s components)

	.anchor (The root of the path, “/” or “C:\”)

Action Methods:
 
    .exists() (Boolean)

	.is_file() (Boolean)

	.is_dir() (Boolean)

	.is_absolute() (Boolean)

	.stat() (Returns info)

    .read_text(encoding=None, ‘utf-8’) (Opens text file, returning it as string and closing it)

    .write_text(data, encoding=None, ‘utf-8’) (Opens text file, writes string into it and closes it)

    .read_bytes() (Opens file in bytes mode, returns contents, and closes)

    .write_bytes(data) (Opens file in bytes mode and writes bytes data into it)

    .mkdir(mode=0o777, parents=False, exist_ok=True) (Create Directory)

    .touch(mode=0o666, exist_ok=True) (Create empty file at this path)

    .rename(target) (rename path to given path)
    etc.

# React

- React is a JavaScript library for building interfaces from reusable components

- A component is a function returning JSX

    ```tsx
    type GreetingProps = { name: string }

    export function Greeting({ name }: GreetingProps) {
        return <h1>Hello {name}</h1>
    }
    ```

- JSX resembles HTML but produces JavaScript element descriptions. Use `className`, camel-cased event names, and JavaScript expressions inside `{}`

- Props are values passed from a parent. State is component-owned memory which triggers rendering when updated

    ```tsx
    import { useState } from 'react'

    export function Counter() {
        const [count, setCount] = useState(0)
        return <button onClick={() => setCount((value) => value + 1)}>{count}</button>
    }
    ```

- State updates describe the next render. Do not mutate state objects or arrays directly

- Use functional state updates when the next value depends on the previous value

- Rendering should remain pure. The same props/state should produce the same JSX without external side effects

- Effects synchronize with systems outside React such as subscriptions, browser APIs, timers, and non-React widgets

    ```tsx
    useEffect(() => {
        const controller = new AbortController()
        loadData(controller.signal)
        return () => controller.abort()
    }, [resourceId])
    ```

- An effect cleanup reverses the previous synchronization before rerunning or unmounting

- Do not use an effect merely to calculate a value from props/state. Calculate during rendering, and use `useMemo` only for measured expensive work or required stable identity

- `useRef` stores a mutable value without triggering a render and can reference DOM elements

- Context shares a value through a subtree. It is useful for themes, authentication context, and stable services, but frequently changing global state may cause broad rerenders

- Controlled inputs receive their value from state and update it through event handlers

- Keys identify list items across renders. Use stable data IDs, not array positions when items reorder, insert, or delete

- Custom hooks package reusable stateful behavior and must follow Hook rules

- Hooks are called at the top level of React components or custom hooks, not conditionally or inside normal loops

- Composition through children and focused components is usually simpler than large inheritance hierarchies

- Error boundaries catch rendering errors below them. Suspense coordinates fallback UI for supported suspended work

- React transitions mark non-urgent updates. Optimistic state shows expected mutation results before the server finishes

- In Next.js App Router, Server Components are the default and browser React hooks require a Client Component boundary

# Prisma ORM

- Prisma ORM provides type-safe database access for Node.js/TypeScript applications

- Main tools:

    Prisma Schema: Declarative data model

    Prisma Client: Generated typed query API

    Prisma Migrate: SQL migration history for supported relational databases

    Prisma Studio: Visual data editor

    Prisma CLI: Initialization, generation, introspection, formatting, validation, and migrations

- A schema defines models and relations

    ```prisma
    datasource db {
      provider = "postgresql"
    }

    generator client {
      provider = "prisma-client"
      output   = "../generated/prisma"
    }

    model User {
      id        String     @id @default(uuid()) @db.Uuid
      email     String     @unique
      notebooks Notebook[]
      createdAt DateTime   @default(now())
    }

    model Notebook {
      id        String   @id @default(uuid()) @db.Uuid
      title     String
      ownerId   String   @db.Uuid
      owner     User     @relation(fields: [ownerId], references: [id], onDelete: Cascade)
      createdAt DateTime @default(now())

      @@index([ownerId, createdAt])
    }
    ```

- `@id`, `@unique`, `@default`, `@relation`, `@map`, and native `@db.*` attributes describe fields

- `@@id`, `@@unique`, `@@index`, and `@@map` describe model-level behavior

- Prisma 7 uses explicit generated client output and database driver adapters in its standard setup. Check the installed major version before copying initialization code

- Common CLI commands:

    ```bash
    npx prisma init
    npx prisma format
    npx prisma validate
    npx prisma generate
    npx prisma migrate dev --name add_notebooks
    npx prisma migrate deploy
    npx prisma db pull
    npx prisma db push
    npx prisma studio
    ```

- `migrate dev` creates/applies migrations for development and may use a shadow database

- `migrate deploy` applies committed migrations in production without generating new ones

- `db pull` introspects an existing database into the Prisma schema

- `db push` updates a schema without migration history and is best for prototypes or databases where Migrate does not apply

- Review generated SQL before committing a migration, especially for type changes, required fields, renames, large indexes, or dropped data

- CRUD examples:

    ```ts
    const user = await prisma.user.create({
        data: { email: 'alice@example.com' },
    })

    const notebook = await prisma.notebook.findUnique({
        where: { id: notebookId },
        select: { id: true, title: true, ownerId: true },
    })

    await prisma.notebook.update({
        where: { id: notebookId },
        data: { title: newTitle },
    })

    await prisma.notebook.delete({ where: { id: notebookId } })
    ```

- Query methods include `findUnique`, `findFirst`, `findMany`, `create`, `createMany`, `update`, `updateMany`, `upsert`, `delete`, `deleteMany`, `count`, `aggregate`, and `groupBy`

- `select` returns named fields. `include` loads relations. Returning only needed fields reduces data exposure and work

- Nested writes create, connect, disconnect, update, or delete related records inside one supported transaction

- Filters use typed operators such as `equals`, `in`, `contains`, `startsWith`, `gt`, `lt`, `some`, `every`, and `none`

- Use `$transaction([query1, query2])` for independent operations or an interactive callback when later queries depend on earlier results

    ```ts
    await prisma.$transaction(async (tx) => {
        const notebook = await tx.notebook.findFirst({
            where: { id: notebookId, ownerId: authenticatedUserId },
        })

        if (!notebook) throw new Error('Notebook not found')

        await tx.page.updateMany({
            where: { notebookId },
            data: { archived: true },
        })
    })
    ```

- Keep interactive transactions short. Do not call slow external APIs while holding the transaction

- Use parameterized tagged-template raw SQL APIs for features not modeled by Prisma. Avoid unsafe string-interpolated raw SQL

- pgvector usually requires custom SQL migrations and raw/type extension patterns because specialized PostgreSQL features may not have complete ORM-native support

- Prisma does not replace database constraints, indexes, query planning, transaction knowledge, or authorization

- Always include ownership/tenant scope inside the query instead of loading by ID and trusting client state

- In Next.js development, reuse one Prisma Client instance to avoid creating a new pool after each hot reload

- Serverless deployments need a compatible driver/pooler and bounded connections. Migrations commonly require a direct connection

- Prisma Studio is an administrative tool. Do not expose it publicly or treat manual edits as a migration process

# Streamlit

- Streamlit builds Python data and ML interfaces from a script which reruns top-to-bottom when user input changes

    ```python
    import streamlit as st

    st.title("Tiny Model Demo")
    prompt = st.text_area("Prompt")

    if st.button("Generate") and prompt:
        with st.spinner("Generating..."):
            st.write(generate(prompt))
    ```

- Widgets return values directly. Use `st.session_state` for values which must persist across reruns

- `st.cache_data` caches serializable computed data. `st.cache_resource` caches shared resources such as loaded models or database clients

- Cache keys depend on function inputs and code. Do not globally cache user-private results without identity and authorization context

- Useful display tools include `st.dataframe`, `st.table`, `st.metric`, `st.line_chart`, `st.image`, `st.audio`, `st.code`, and layout columns/tabs

- Uploads from `st.file_uploader` are untrusted and require validation

- Keep model loading outside repeated uncached work and use inference mode during PyTorch generation

- Streamlit is useful for demos, internal tools, experiments, model inspection, and lightweight applications. Complex custom client behavior may require a conventional frontend framework


