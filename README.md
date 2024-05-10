# Mhark blog

[mhark.co.uk](https://mhark.co.uk)

It's built with [Astro](https://astro.build), and uses [TinaCMS](https://tina.io) to edit the content!

## Todo

- [ ] Add your TinaCMS keys (see below)
- [ ] Edit the images in `public/` (optional)

After this, you can add your content to `src/posts` with Markdown files, or with TinaCMS by going to `yoururl.com/admin`!

## Run it yourself

All commands are run from the root of the project, from a terminal:

| Command                          | Action                                                        |
| :------------------------------- | :------------------------------------------------------------ |
| `bun install`                    | Installs dependencies                                         |
| `bun run dev`                    | Starts local dev server at `localhost:4321`                   |
| `bunx tinacms dev -c 'astro dev'` | Manually run local server if the regular command doesn't work |
| `bun run build`                  | Build your production site to `./dist/`                       |
| `bun run preview`                | Preview your build locally, before deploying                  |

You go to `localhost:4321/admin/index.html` to see the CMS and use it. If you want to clone this for yourself, you'll need a `.env.development` file that has the following in it:

```dotenv
TINACLIENTID=<from tina.io>
TINATOKEN=<from tina.io>
TINASEARCH=<from tina.io>
```

## Credits

- [Cassidy](https://x.com/cassidoo/status/1788810892152340671) - creator of the [Blhag template](https://github.com/cassidoo/blahg) originally used.
