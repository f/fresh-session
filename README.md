# Fresh Session 🍋

Dead simple cookie-based session for [Deno Fresh](https://fresh.deno.dev).

## Get started

Fresh Session comes with a simple middleware to add at the root of your project,
which will create or resolve a session from the request cookie.

### Install / Import

You can import Fresh Session like so:

```ts
import {
  cookieSession,
  createCookieSessionStorage,
  CookieSessionStorage
  WithSession,
  Session,
} from "https://deno.land/x/fresh_session@0.1.1";
```

### Setup secret key

Fresh Session currently use [JSON Web Token](https://jwt.io/) under the hood to
create and manage session in the cookies.

JWT requires to have a secret key to encrypt new token. Fresh Session use the
session from your [environment variable](https://deno.land/std/dotenv/load.ts)
`APP_KEY`.

If you don't know how to setup environment variable locally or on Deno Deploy, a
blogpost is on the way on [my blog](https://xstevenyung.com)

### Create a root middleware (`./routes/_middleware.ts`)

```ts
import { MiddlewareHandlerContext } from "$fresh/server.ts";
import { cookieSession WithSession } from "https://deno.land/x/fresh_session@0.1.1";

export type State = WithSession;

export function handler(
  req: Request,
  ctx: MiddlewareHandlerContext<State>,
) {
  return cookieSession(req, ctx);
}
```

Learn more about
[Fresh route middleware](https://fresh.deno.dev/docs/concepts/middleware).

### Interact with the session in your routes

Now that the middleware is setup, it's going to handle creating/resolving
session based on the request cookie. So all that you need to worry about is
interacting with your session.

```tsx
// ./routes/dashboard.tsx
/** @jsx h */
import { h } from "preact";
import { Handlers, PageProps } from "$fresh/server.ts";
import { WithSession } from "https://deno.land/x/fresh_session@0.1.1";

export type Data = { session: Record<string, string> };

export const handler: Handlers<
  Data,
  WithSession // indicate with Typescript that the session is in the `ctx.state`
> = {
  GET(_req, ctx) {
    // The session is accessible via the `ctx.state`
    const { session } = ctx.state;

    // Access data stored in the session
    session.get("email");
    // Set new value in the session
    session.set("email", "hello@deno.dev");
    // returns `true` if the session has a value with a specific key, else `false`
    session.has("email");
    // clear all the session data
    session.clear();
    // Access all session data value as an object
    session.data;

    // You can pass the session data to the page
    return ctx.render({ session: session.data });
  },
};

export default function Dashboard({ data }: PageProps<Data>) {
  return <div>You are logged in as {data.session.email}</div>;
}
```
