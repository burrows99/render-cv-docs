
Go to file
t
Name		
ml-aixolotl
ml-aixolotl
Merge pull request #121 from aixolotl/feature/105-display-issues-in-d…
e56a1d1
 · 
3 days ago
.cursor/rules
Restore Ultracite + fix sidebar (#1233)
4 months ago
.github
Update Github issue templates
2 weeks ago
.vscode
Update agent instuctions for both Copilot and Windsurf
2 months ago
.windsurf/rules
Consistent instuction filename pattern
2 months ago
app
105 - Updated comment
3 days ago
artifacts
Merge branch 'enhancement/ensure-consistent-typing-and-canvas-variabl…
last month
components
105 - Lint issues sorted out
3 days ago
hooks
feat(mcp): enable GitHub authentication via MCP panel and streamline …
last month
lib
Merge pull request #112 from aixolotl/task/hide-shadcn-tool
last month
public/images
Initial vercel template commit
6 months ago
tests
Merge branch 'dev'
2 months ago
.env.example
Update env example for new param
last month
.gitignore
Remove build artifacts and update .gitignore
last month
LICENSE
Clean up deps
2 years ago
README.md
105 - Fixed display issues in the dark mode for business components r…
4 days ago
biome.jsonc
Apply biome lintin
2 months ago
components.json
Upgrade to Tailwind CSS v4 (#1173)
4 months ago
docker-compose.yml
Added optional docker "host" profile for locally running
last month
drizzle.config.ts
Apply biome lintin
2 months ago
instrumentation.ts
Apply biome lintin
2 months ago
next-env.d.ts
Squashed commit of the following:
last month
next.config.ts
Squashed commit of the following:
last month
package.json
feat(mcp): implement GitHub MCP authentication via MCP panel and refa…
last month
plan.md
Fix UT and update plan
2 months ago
playwright.config.ts
Apply biome lintin
2 months ago
pnpm-lock.yaml
feat(mcp): implement GitHub MCP authentication via MCP panel and refa…
last month
postcss.config.mjs
Restore Ultracite + fix sidebar (#1233)
4 months ago
proxy.ts
Squashed commit of the following:
last month
setup-tests.ts
test: add comprehensive test suite for business components
2 months ago
tailwind.config.ts
refactor(ui): simplify button variants and dark mode config
2 months ago
tsconfig.json
Enhance type safety and consistency across canvas components and work…
last month
vercel-template.json
Add vercel-template.json (#1193)
4 months ago
vitest.config.ts
style: format vitest config file for consistency
2 months ago
Repository files navigation
README
License
Next.js 14 and App Router-ready AI chatbot.
Chat SDK
Chat SDK is a free, open-source template built with Next.js and the AI SDK that helps you quickly build powerful chatbot applications.

Read Docs · Features · Model Providers · Deploy Your Own · Running locally


Features
Next.js App Router
Advanced routing for seamless navigation and performance
React Server Components (RSCs) and Server Actions for server-side rendering and increased performance
AI SDK
Unified API for generating text, structured objects, and tool calls with LLMs
Hooks for building dynamic chat and generative user interfaces
Supports xAI (default), OpenAI, Fireworks, and other model providers
shadcn/ui
Styling with Tailwind CSS
Component primitives from Radix UI for accessibility and flexibility
Data Persistence
Neon Serverless Postgres for saving chat history and user data
Vercel Blob for efficient file storage
Auth.js
Simple and secure authentication
Model Providers
This template uses the Vercel AI Gateway to access multiple AI models through a unified interface. The default configuration includes xAI models (grok-2-vision-1212, grok-3-mini) routed through the gateway.

AI Gateway Authentication
For Vercel deployments: Authentication is handled automatically via OIDC tokens.

For non-Vercel deployments: You need to provide an AI Gateway API key by setting the AI_GATEWAY_API_KEY environment variable in your .env.local file.

With the AI SDK, you can also switch to direct LLM providers like OpenAI, Anthropic, Cohere, and many more with just a few lines of code.

Deploy Your Own
You can deploy your own version of the Next.js AI Chatbot to Vercel with one click:

Deploy with Vercel

Running locally
Prerequisites
Node.js 24.12.0 (use your preferred version manager or download from nodejs.org).
pnpm (needed for dependency management).
Docker Desktop running locally for the database and services defined in docker-compose.yml.
Ollama installed and activated (requires creating a free account and signing in once).
You will also need the environment variables defined in .env.example. Create a .env.local and .env (or use Vercel Environment Variables) and never commit it to git. The AI_PROVIDER variable supports ollama and azure; switch between them in your .env to pick the provider you want at runtime.

Setup steps
# 1. Install dependencies
pnpm install

# 2. Start local infra (database, etc.)
docker-compose up -d

# 3. Apply database migrations
pnpm db:migrate

# 4. Make sure Ollama is running locally and you are signed in

# 5. Start the app
pnpm dev
Visit localhost:3000 once pnpm dev reports that the server is ready.

About
AIxPM Chatbot interface

aix-chat-wine.vercel.app
Resources
 Readme
License
 View license
 Activity
 Custom properties
Stars
 1 star
Watchers
 0 watching
Forks
 1 fork
Releases
No releases published
Create a new release
Packages
No packages published
Publish your first package
Contributors
3
@burrows99
burrows99 Raunak Burrows
@ml-aixolotl
ml-aixolotl Martin
@mislavKucanda
mislavKucanda
Deployments
31
 Preview 2 days ago
 Production 3 days ago
+ 29 deployments
Languages
TypeScript
98.0%
 
JavaScript
1.2%
 
CSS
0.8%
Footer
© 2026 GitHub, Inc.
Footer navigation
Terms
Privacy
Security
Status
Community
Docs
Contact
Manage cookies