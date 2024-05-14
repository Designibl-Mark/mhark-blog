# Mhark blog

[mhark.co.uk](https://mhark.co.uk)

It's built with [Astro](https://astro.build), and uses [TinaCMS](https://tina.io) to edit the content!

## Run it yourself

All commands are run from the root of the project, from a terminal:

| Command                          | Action                                                        |
| :------------------------------- | :------------------------------------------------------------ |
| `bun install`                    | Installs dependencies                                         |
| `bun run dev`                    | Starts local dev server at `localhost:4321`                   |
| `bunx tinacms dev -c 'astro dev'` | Manually run local server if the regular command doesn't work |
| `bun run build`                  | Build your production site to `./dist/`                       |
| `bun run preview`                | Preview your build locally, before deploying                  |

You go to `localhost:4321/admin/index.html` to see the CMS and use it. If you want to clone this for yourself, you'll need a `.env.development` file so clone and edit `.env.example`

## Credits

- [Cassidy](https://x.com/cassidoo/status/1788810892152340671) - creator of the [Blhag template](https://github.com/cassidoo/blahg) originally used.
