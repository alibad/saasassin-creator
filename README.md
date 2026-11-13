# SaaSassin Creator

**Free, open-source Kajabi alternative.** Run a paid creator business on an open stack that moves with you.

Creator business OS for independent educators, coaches, and membership businesses.

- **Try it:** [creator.saasassin.net](https://creator.saasassin.net)
- **Research brief and product decision:** [saasassin.net/targets/kajabi](https://saasassin.net/targets/kajabi)

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-creator&project-name=saasassin-creator&repository-name=saasassin-creator&env=NEXT_PUBLIC_FIREBASE_API_KEY%2CNEXT_PUBLIC_FIREBASE_AUTH_DOMAIN%2CNEXT_PUBLIC_FIREBASE_PROJECT_ID%2CNEXT_PUBLIC_FIREBASE_APP_ID%2CFIREBASE_PROJECT_ID%2CFIREBASE_CLIENT_EMAIL%2CFIREBASE_PRIVATE_KEY&envDescription=Firebase+web+app+config+and+service-account+credentials+%28see+the+README%29&envLink=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-creator%231-create-a-firebase-project)

Kajabi is a trademark of its owner and is named here only to describe what this product
replaces. SaaSassin Creator is an independent, clean-room product and is not affiliated
with or endorsed by Kajabi.

## What you can do today

| Who | Can now |
|-----|---------|
| Creator | Create a school (tenant), invite teammates by email as admin / member / viewer |
| Creator | Create a course, add modules and lessons, reorder both, write lesson text (markdown-ish), attach a YouTube / Vimeo / Loom video, set a drip delay per lesson, attach a scored quiz, preview any lesson as a learner |
| Creator | Move a course draft → published → draft, or archive / restore it; every transition is checked server-side, confirmed in a dialog and written to the audit log |
| Creator | **Migrate from Kajabi**: paste or upload a Kajabi-style CSV (Product / Category / Post columns) or JSON export, review a preview with column mapping, warnings and errors, then import every course as a draft in one audited batch |
| Creator | **Duplicate** any course as a new draft |
| Creator | Set free enrollment, or store a paid price that is shown honestly as **Payments not connected** (nobody can be charged; paid enrollment is refused with 402) |
| Visitor | Browse a school's **public catalog** at `/apps/creator/s/{tenantId}` and each course landing page at `/apps/creator/c/{tenantId}/{courseSlug}` (published courses only; drafts 404); join the course email list with explicit consent |
| Learner (any Creator account) | Sign in from the landing page, enroll for free, open lessons, watch videos, mark lessons complete / incomplete, take quizzes, see progress, **discuss each lesson** with other learners, and open a printable certificate once every lesson is done |
| Learner | See every course they joined under **My learning** on the Creator home |
| Creator | See learners with per-learner progress, quiz results, completion and access status; the email list; team; audit log |
| Creator | **Course analytics** from real records: enrollments over time, completion rate, per-lesson completion funnel, drop-off lesson, quiz pass rates — each definition shown on screen |
| Creator | **Moderate discussions**: see every lesson thread, hide or restore comments (audited) |
| Owner / admin | **Bulk-enroll** existing accounts into a published course by pasting emails; get back who was enrolled, who already was, who has no account, and which entries were invalid |
| Owner / admin | Revoke a learner's access (progress kept, re-enrollment blocked) and restore it later |
| Owner / admin | Remove a lead when someone withdraws consent (the audit log keeps the fact, not the address) |
| Owner / admin | Download the whole school as one versioned JSON export |

Full status, verification and known gaps: [docs/STATUS.md](docs/STATUS.md).

## Self-host

SaaSassin Creator is a Next.js app backed by Firebase Authentication and Cloud Firestore.
You need Node.js 20.9 or newer, [pnpm](https://pnpm.io/installation) and a Firebase project of
your own.

### 1. Create a Firebase project

1. Create a project in the [Firebase console](https://console.firebase.google.com/).
2. **Firestore:** Build → Firestore Database → Create database.
3. **Authentication:** Build → Authentication → Get started. Creator signs people in with
   **Google** and **Email/Password**; step 2 below enables both, or enable them yourself under
   Authentication → Sign-in method.
4. **Web app config:** Project settings → General → Your apps → add a Web app. Copy its config
   into the `NEXT_PUBLIC_FIREBASE_*` variables.
5. **Service account:** Project settings → Service accounts → Generate new private key. From the
   downloaded JSON, set `FIREBASE_PROJECT_ID` (`project_id`), `FIREBASE_CLIENT_EMAIL`
   (`client_email`) and `FIREBASE_PRIVATE_KEY` (`private_key`). Never commit the key, and never
   give these variables a `NEXT_PUBLIC_` prefix.
6. Once you deploy, add your domain under Authentication → Settings → Authorized domains
   (`localhost` is already allowed).

### 2. Deploy the rules, indexes and sign-in providers

```bash
pnpm install
pnpm exec firebase login
pnpm exec firebase deploy --only firestore:rules,firestore:indexes,auth --project <your-project-id>
```

`firestore.rules` denies all browser access to product data: every read and write goes through the server routes, which check the session, tenant membership and role with the Firebase Admin SDK. `auth` enables the Google and email/password providers listed in `firebase.json`.

### 3. Run it

```bash
cp .env.example .env.local   # fill in the values from step 1
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000). This repository serves SaaSassin
Creator at the root of every host: `next.config.ts` sets `STANDALONE_PRODUCT=creator`,
and `src/proxy.ts` maps `/` to the product (`/apps/creator`) and `/sign-in` to its sign-in page.

### 4. Deploy to Vercel

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-creator&project-name=saasassin-creator&repository-name=saasassin-creator&env=NEXT_PUBLIC_FIREBASE_API_KEY%2CNEXT_PUBLIC_FIREBASE_AUTH_DOMAIN%2CNEXT_PUBLIC_FIREBASE_PROJECT_ID%2CNEXT_PUBLIC_FIREBASE_APP_ID%2CFIREBASE_PROJECT_ID%2CFIREBASE_CLIENT_EMAIL%2CFIREBASE_PRIVATE_KEY&envDescription=Firebase+web+app+config+and+service-account+credentials+%28see+the+README%29&envLink=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-creator%231-create-a-firebase-project)

The button copies this repository into your GitHub account, creates a Vercel project and asks for
the Firebase variables. To deploy an existing fork instead, import it in Vercel and add the
variables from `.env.example` under Project Settings → Environment Variables. Either way, add the
Vercel domain to your Firebase authorized domains afterwards.

## Tests

```bash
pnpm lint
pnpm typecheck
pnpm test        # unit tests
pnpm build
```

The Firestore suites run against the local Firestore emulator, which needs a Java 21+ JDK first on
your `PATH`:

```bash
export JAVA_HOME=/path/to/jdk-21   # macOS example: /Library/Java/JavaVirtualMachines/temurin-21.jdk/Contents/Home
export PATH="$JAVA_HOME/bin:$PATH"
pnpm test:rules      # Firestore security rules
pnpm test:emulator   # the golden workflow against a real Firestore emulator
```

## Contributing

Development happens in the SaaSassin monorepo,
[github.com/alibad/saasassin](https://github.com/alibad/saasassin), the single source of truth for
all ten SaaSassin products. This repository is generated from it by the monorepo's
`scripts/export-product.mjs`, so accepted changes land in the monorepo first and are then synced
here. Open issues and pull requests there or here.

Report vulnerabilities privately as described in [SECURITY.md](SECURITY.md).

## Talk to us

Moving off Kajabi, running Creator for your team, or want to help build it?
[Tell us on saasassin.net](https://saasassin.net/targets/kajabi#interest).

## License

[GNU Affero General Public License v3.0](LICENSE) (AGPL-3.0-only). Use it, self-host it and modify
it; if you run a modified version as a network service, publish your changes under the same
license.
