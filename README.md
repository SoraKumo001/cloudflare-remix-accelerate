# Remix + Cloudflare Pages + Prisma Accelerate Self Hosting

## Create Self Hosted Prisma Accelerate

- https://github.com/SoraKumo001/prisma-accelerate-deno
- https://github.com/SoraKumo001/prisma-accelerate-workers

## Create API Key

```Sample of Supabase
npx prisma-accelerate-local -s secret -m postgres://postgres.xxxxxxxx:xxxxxxx@aws-0-ap-northeast-1.pooler.supabase.com:6543/postgres
```

## Environment Variables

- prisma.schema

```prisma
generator client {
  provider        = "prisma-client-js"
  previewFeatures = ["driverAdapters"]
}

datasource db {
  provider = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_DATABASE_URL")
}

model Test {
  id String @id @default(uuid())
}
```

- .env

```env
DIRECT_DATABASE_URL=postgres://postgres.xxxx:xxxxx@aws-0-ap-northeast-1.pooler.supabase.com:5432/postgres?schema=public
```

- .dev.var

```env
DATABASE_URL=prisma://xxxx.xxxx.workers.dev?api_key=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

- commands

```sh
npm run prisma migrate dev
npm run prisma generate --no-engine
```

## Sample of Remix

- app/routes/\_index.tsx

```tsx
import { PrismaClient } from "@prisma/client/edge";
import { LoaderFunctionArgs } from "@remix-run/cloudflare";
import { useLoaderData } from "@remix-run/react";

export default function Index() {
  const values = useLoaderData<string[]>();
  return (
    <div>
      {values.map((v) => (
        <div key={v}>{v}</div>
      ))}
    </div>
  );
}

export async function loader({
  context,
}: LoaderFunctionArgs): Promise<string[]> {
  const prisma = new PrismaClient({
    datasourceUrl: context.cloudflare.env.DATABASE_URL,
  });
  await prisma.test.create({ data: {} });
  return prisma.test.findMany({ where: {} }).then((r) => r.map(({ id }) => id));
}
```

## Deploy

Set the `DATABASE_URL` in the dashboard or `wrangler.toml`.

## Demo

https://cloudflare-remix-accelerate.pages.dev/
