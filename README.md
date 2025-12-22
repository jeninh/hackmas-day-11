# Let's make a blog with Astro and Markdown!

This workshop will _hopefully_ tech you how to make a simple blog using [Astro](https://astro.build) in about 2 to 3 hours :3

The example project for this workshop can be found here: [https://haxmas-day-12.shymike.dev](https://haxmas-day-12.shymike.dev)
If you have any questions, feel free to ask DM me ([@miggy](https://hackclub.enterprise.slack.com/team/U07VC9705D4)) on Slack!

Have fun!

| Prizes         |
|----------------|
|10$ domain grant|
|1 snowflake     |

![workshop banner](./assets/workshop-banner.png)

---

## Prerequisites

Before starting this workshop, make sure you have the following installed on your machine:

- [Bun](https://bun.sh) ([npm](https://www.npmjs.com/package/npm) also works but I will be using Bun)
- A [GitHub](https://github.com) account
- A [Cloudflare](https://cloudflare.com) account
- An IDE (I will be using [VSCode](https://code.visualstudio.com))

## 1) Create a new Astro project

To start, set up a new Astro project by running the following command in your terminal:

```bash
bun create astro@latest -- --template minimal
```

Give your project a name (e.g., `haxmas-day-12`) and navigate into the project directory:

```bash
cd haxmas-day-12
```

Woah! That was easy!

Now to start the development server, run:

```bash
bun dev
```

You should now be able to head over to `http://localhost:4321` and see your new Astro project running!

![astro dev server](./assets/setup.png)

## 2) Installing dependencies

No extra dependencies are required but if you are familiar with [tailwindcss](https://tailwindcss.com), you can install it by running:

```bash
bun astro add tailwind
```

## 3) Making a blog structure

To create a blog structure, we will need to set up a few directories and files.

Your `src` directory should look like this:

```src
├── assets
├── components
│   └── FormattedDate.astro
├── content
│   └── posts
├── layouts
│   └── BlogLayout.astro
├── pages
│   ├── posts
│   │   ├── [...slug].astro
│   │   └── index.astro
│   └── index.astro
└── content.config.ts
```

Don't worry about the contents of these files yet, we will get to that later.

## 4) Configuring content

Astro will need to read and parse our markdown files, for that we will need to configure `content.config.ts`.

We will be using the `./src/content/posts` directory to store our blog posts.

```ts
import { defineCollection, z } from 'astro:content';
import { glob } from 'astro/loaders';

const blog = defineCollection({
    // Tell Astro where to look for blog posts
    loader: glob({ base: './src/content/posts', pattern: '**/*.md' }),
    // Schema of the data in each markdown file (using Zod)
    schema: ({ image }) =>
        z.object({
            title: z.string(),
            description: z.string(),
            pubDate: z.coerce.date(),
            updatedDate: z.coerce.date().optional(),
            heroImage: image().optional(),
        }),
});

// Export the collection
export const collections = { blog };
```

Astro can now read and parse our markdown files! Now let's create some blog posts :3

## 5) Creating blog posts

To create a blog post, create a new markdown file in the `src/content/posts` directory.

I will be using the following markdown file as an example: `hello-world.md`

```md
---
title: Hello, World!
description: This is a very cool workshop
pubDate: Aug 08 2025
heroImage: ../../assets/blog/crazy.jpg
---

This is a very cool blog post that was made using markdown :3

You can make stuff __bold__, *italic*, or even ~~strikethrough~~ like in regular markdown!
```

## 6) Displaying blog posts

To display a blog post, we will need to create a layout and a page to render the blog posts.

That's exactly what `BlogLayout.astro` and `posts/[...slug].astro` are for!

The layout file will be used to style and structure the blog post page, while the slug file will be used to match the route to the correct blog post.

- `src/layouts/BlogLayout.astro`:

```astro
---
import { Image } from 'astro:assets';
import type { CollectionEntry } from 'astro:content';

// Import the FormattedDate component
import FormattedDate from '../components/FormattedDate.astro';

// Get the blog post data from props
type Props = CollectionEntry<'blog'>['data'];
const { title, description, pubDate, updatedDate, heroImage } = Astro.props;
---
<div class="post-page">
    <article class="post">
        <!-- Header with prost data -->
        <header>
            <h1>{title}</h1>
            <p class="description">{description}</p>
            {heroImage && (
                <Image src={heroImage} format="webp" alt={title} class="hero" />
            )}
            <p class="date">
                <FormattedDate date={pubDate} />
            </p>
            {updatedDate && (
                <p class="updated">
                    Last updated on <FormattedDate date={updatedDate} />
                </p>
            )}
        </header>
        <!-- The post's content will be rendered in the slot tag -->
        <div class="body">
            <slot />
        </div>
    </article>
</div>

<!-- Example CSS styling -->
<style>
    body {
        margin: 0;
        font-family: Arial, sans-serif;
        background: #f4f0ff 0%;
        color: #2a2a2a;
    }

    header {
        padding-bottom: 16px;
        border-bottom: 3px solid #6a6a7a;
        text-align: center;
    }

    .post-page {
        min-height: 100vh;
        display: flex;
        justify-content: center;
        padding: 56px 16px 72px;
    }

    .post {
        max-width: 920px;
        width: 100%;
        padding: 32px 24px 40px;
        color: inherit;
    }

    h1 {
        margin: 0 0 12px;
        font-size: 2.4rem;
        color: #322a53;
        letter-spacing: -0.02em;
    }

    .description {
        margin: 0 0 20px;
        font-size: 1.1rem;
        color: #4a4761;
    }

    .hero {
        width: 100%;
        max-height: 460px;
        object-fit: cover;
        margin: 20px 0 10px;
        border-radius: 14px;
    }

    .date,
    .updated {
        margin: 6px 0;
        color: #6a6a7a;
        font-weight: 600;
        letter-spacing: 0.01em;
    }

    .body {
        line-height: 1.7;
        color: #2f2f39;
        padding: 12px 32px;
    }
</style>
```

- `src/pages/posts/[...slug].astro`:

```astro
---
import { type CollectionEntry, getCollection, render } from 'astro:content';
import BlogPost from '../../layouts/BlogPost.astro';

// This is used to generate static paths for all blog posts
export async function getStaticPaths() {
    const posts = await getCollection('blog');
    return posts.map((post) => ({
        params: { slug: post.id },
        props: post,
    }));
}

// Get the blog post data from props
type Props = CollectionEntry<'blog'>;
const post = Astro.props;

// Render the markdown content to HTML
const { Content } = await render(post);
---

<BlogPost {...post.data}>
    <Content />
</BlogPost>
```

Oh no! We're missing a `FormattedDate` component!

_Wait, what's even a component? It's reusable piece of code that can be used in many different places on your website._

Let's add it in `src/components/FormattedDate.astro`:

```astro
---
interface Props {
    date: Date;
}

const { date } = Astro.props;
---

<time datetime={date.toISOString()}>
    {
        date.toLocaleDateString('en-us', {
            year: 'numeric',
            month: 'short',
            day: 'numeric',
        })
    }
</time>
```

## 7) It exists?

You can now head over to `http://localhost:4321/posts/hello-world` (or whatever you named your markdown file) to see your blog post live!

_(if you see an error about missing content or incorrect types, stop the dev server and start it again with `bun dev`)_

![blog post](./assets/post.png)

**The styling is currently very basic and you should tweak it and make it your own before submitting!**

You can see that it's missing a page to list all blog posts. Let's add that next!

## 8) Listing blog posts

The page that will be used to list all blog posts will be `/posts` and the file for that is `src/pages/posts/index.astro`.

It will fetch all blog posts from the content collection and display them in a list.

```astro
---
import { getCollection } from 'astro:content';
import FormattedDate from '../../components/FormattedDate.astro';

// Sort posts by publication date, most recent first
const posts = (await getCollection('blog')).sort(
    (a, b) => b.data.pubDate.valueOf() - a.data.pubDate.valueOf()
);
---

<div class="posts-page">
    <header class="page-header">
        <h1>My very cool blog</h1>
        <p class="tagline">Make this unique!</p>
    </header>

    {posts.length === 0 && <p class="empty">No posts yet.</p>}

    <div class="posts">
        {posts.map((post) => (
            <article class="post-card">
                <a href={`/posts/${post.id}`} class="title-link">
                    {post.data.title}
                </a>
                <p class="description">{post.data.description}</p>
                <p class="meta">
                    Published on <FormattedDate date={post.data.pubDate} />
                    {post.data.updatedDate && (
                        <>
                            {' '}
                            | Updated on{' '}
                            <FormattedDate date={post.data.updatedDate} />
                        </>
                    )}
                </p>
            </article>
        ))}
    </div>
</div>

<style>
    body {
        margin: 0;
        font-family: Arial, sans-serif;
        background: #f4f0ff 0%;
        color: #2a2a2a;
    }

    .posts-page {
        min-height: 100vh;
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    header {
        text-align: center;
        padding: 32px 24px 40px;
    }

    h1 {
        margin: 0 0 10px;
        font-size: 2.4rem;
        color: #322a53;
        letter-spacing: -0.02em;
    }

    .tagline {
        margin: 0;
        font-size: 1.1rem;
        color: #4a4761;
    }

    .empty {
        margin: 0;
        color: #6a6a7a;
        font-weight: 600;
    }

    .posts {
        width: 100%;
        max-width: 920px;
        display: grid;
        gap: 18px;
    }

    .post-card {
        background: #fff;
        padding: 20px 18px 22px;
        border: 1px solid #d7d3e5;
        box-shadow: 0 12px 32px rgba(31, 20, 71, 0.08);
        color: inherit;
        transition: transform 180ms ease, box-shadow 180ms ease;
    }

    .title-link {
        display: inline-block;
        margin-bottom: 10px;
        font-size: 1.4rem;
        font-weight: 700;
        color: #322a53;
        text-decoration: none;
    }

    .title-link:hover {
        color: #5145b7;
        text-decoration: underline;
    }

    .description {
        margin: 0 0 12px;
        color: #2f2f39;
        line-height: 1.6;
    }

    .meta {
        margin: 0;
        color: #6a6a7a;
        font-weight: 600;
        letter-spacing: 0.01em;
        font-size: 0.95rem;
    }

    @media (max-width: 600px) {
        .posts-page {
            padding: 32px 14px 48px;
        }

        .post-card {
            padding: 18px 16px 20px;
        }
    }
</style>
```

You can now head over to `http://localhost:4321/posts` to see all your blog posts!

![blog posts list](./assets/posts.png)

## 9) Customizing

You may have seen that the home page (`http://localhost:4321`) is still just the default Astro page and that the theme is very basic.

**Before submitting, make sure to customize the home page and the blog's styles to make it your own!**

Need help with Astro? Check out the [Astro documentation](https://docs.astro.build)!

Want to implement more features into your blog? Check out [Astro's official blog guide](https://docs.astro.build/en/tutorial/0-introduction)!

## 10) Deploying to Cloudflare Pages

To deploy your very amazing blog to Cloudflare Pages:

- Go to the [Cloudflare Pages](https://pages.cloudflare.com) website and log in to your Cloudflare account.

- Click on `Create Application`, then click the small text on the bottom saying `Looking to deploy Pages? Get started`.

![cloudflare pages create](./assets/cf-pages.png)

- After that, select `Import an existing Git repository` and add your GitHub account.

- Find your project's repository, select it and hit `Begin setup`.

- Edit the following settings:
  - **Project name**: Your project name
  - **Framework preset**: `Astro`
  - **Build command**: `bun build`

![cloudflare pages deployment](./assets/create-deployment.png)

- Click `Save and Deploy`.

You're done! After a few seconds, your blog should be live on Cloudflare Pages!

![cloudflare pages deployed](./assets/success.png)

## 11) Submitting

### [https://forms.hackclub.com/haxmas-day-12](https://forms.hackclub.com/haxmas-day-12)
