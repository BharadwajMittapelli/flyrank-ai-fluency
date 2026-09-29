# Portfolio framed cases
**Bharadwaj Mittapelli** · CSE, Institute of Aeronautical Engineering  
**Audience:** hiring engineers / eng managers who need to see how I think about backends, money, and agent boundaries.  
**One action:** email me and book a 15-minute system review.

Sources used for this draft: public GitHub (`BharadwajMittapelli`), resume, and the live sitemap at [portfolio-sitemap](https://bharadwajmittapelli.github.io/portfolio-sitemap/). I used those as the “interview” because I cannot sit you down live. Read this out loud. Cut anything you would not say.

---

## Voice card (5–7 words)

**plain, specific, first-person, no buzzwords**

### Standing instruction for Claude Project (paste this)

Voice: plain, specific, first-person, no buzzwords.  
Write like Bharadwaj talking to another engineer. Use his words and his actual decisions. Do not invent metrics, logos, or “results-driven” filler. Every case is three beats: the problem, what I did and decided, what came of it. Speak to hiring engineers. End by asking them to email for a 15-minute system review. If a fact is not in the repo, resume, or sitemap, say so or leave it out.

---

## Interview notes (messy, from the work)

**Internship / Flyrank — BlackSwan AI**  
Generic “paste a contract, get an AI summary” wraps hallucinate. I did not want another wrapper. Split Next.js UI from FastAPI. Custom MCP + AST parser so the model has to talk from code structure, not vibes. DexScreener for tickers. Fallback UI when APIs rate-limit. Live: https://blackswan-ai-45.vercel.app · https://github.com/BharadwajMittapelli/blackswan-ai

**CrednPay**  
People want card benefits they do not have. I built P2P escrow, not “just send money.” Parallel Razorpay/UPI webhooks can double-apply a balance. App-level locks die across pods. Decision: `SELECT … FOR UPDATE` on the account row, replay check on txn id, 2s DB timeout so a lock cannot sit forever. https://github.com/BharadwajMittapelli/crednpay

**CodeCheck**  
Running stranger code in Docker still shares a kernel. Sitemap piece: Go jailer + Firecracker microVM, 256MB cap, read-only rootfs, no guest net/IPC. This is the isolation design on `/systems`, not a separate public product repo. Be honest about that.

**E-commerce / Instagram Graph**  
Promo posts dump mentions. If you call Graph on every webhook you eat 429s and get banned. Ack 200, dump to Redis, token-bucket outbound (~200/hour in the mock). IARE lab: token-bucket prototype for flaky webhook connections.

---

## Case 0 — BlackSwan AI (Flyrank internship)

*Smart-contract risk engine. Not a sitemap `/systems` tile, but it is the internship work the brief said to start with.*

**The problem**  
Most “AI auditors” are a chat box on top of source. That looks smart until the model invents a vulnerability or misses a real one. If someone is about to buy a token, a pretty paragraph is worse than silence.

**What I did / decided**  
I split the product: Next.js dashboard at the edge, FastAPI for the heavy path. I did not prompt the model on raw Solidity and hope. I built an MCP loop with tools that fetch on-chain data, parse the contract as an AST, and score liquidity/risk, then I make Gemini write from that payload. Tickers like `pepe` resolve through DexScreener instead of a hardcoded map. Known demo addresses (reentrancy vault, ungated mint) go to a static path so a live demo does not depend on a flaky RPC. When APIs throttle, the UI shows a placeholder instead of dying.

**What came of it**  
A working diligence board: team / tech / tokenomics / roadmap, holder clustering, syntax explorer next to the snippet. Deployed: [blackswan-ai-45.vercel.app](https://blackswan-ai-45.vercel.app). Repo: [blackswan-ai](https://github.com/BharadwajMittapelli/blackswan-ai). The point is not “we use agents.” The point is the model is not allowed to talk until the parser and the chain tools have spoken.

---

## Case 1 — CrednPay (sitemap: SYS_A / async ledger)

**The problem**  
CrednPay is a credit-card sharing / escrow flow. Borrower wants someone else’s card benefits; money has to sit until the lifecycle says release. Payment gateways fire webhooks more than once, and two webhooks for the same wallet can run at the same time. Two workers read ₹100. One adds ₹10, one subtracts ₹50. Last write wins. Ledger is now wrong. If I lock in Node, that lock is per process. Scale to two pods and it is gone.

**What I did / decided**  
Backend: Node/Express, PostgreSQL, JWT roles, a real transaction table, Razorpay + UPI with webhooks. Isolation lives in Postgres, not in memory: `SELECT balance, status FROM accounts WHERE id = %s FOR UPDATE`, then refuse a txn id already `PROCESSED`, then update balance and insert. Commit drops the lock. Connections time out at 2s so a dropped client cannot pin a row. Frontend is React + Tailwind: dashboard, status, disputes. The sitemap simulator is there so a stranger can watch Tx1 block Tx2 instead of reading a paragraph about ACID.

**What came of it**  
A ledger path that cannot silently overwrite a parallel webhook. Replay does not double-pay. The UI is not the product; the row lock and the txn table are. Code and story: [crednpay](https://github.com/BharadwajMittapelli/crednpay), sitemap `/systems` → CrednPay.

---

## Case 2 — CodeCheck (sitemap: SYS_B / isolation VMM)

**The problem**  
If you compile or run user code, Docker is not a hard wall. Same kernel as the host. A kernel bug is your bug. You also need the jail up in milliseconds or nobody will use it for CI-style checks.

**What I did / decided**  
Two layers. Outside: small Go daemon, rootless container. Inside: Firecracker, read-only rootfs, skinny guest kernel, networking and IPC off. Memory hard-capped at 256MB, CPU throttled, so a fork bomb or leak dies in the guest. I would rather a job OOM than escape. The sitemap jailer (`codecheck/sandbox.go`) is the contract I am willing to defend: socket, kernel image, drive, limits.

**What came of it**  
An isolation design you can click through on `/systems` (boot story &lt;5ms, 256MB tracker). I do not have a separate public “CodeCheck” product repo next to crednpay. If I put this on the site, I say: this is the sandbox architecture, demonstrated, not a shipped SaaS with customer logs.

---

## Case 3 — E-commerce webhooks (sitemap: SYS_C / Instagram Graph)

**The problem**  
A shop posts. Mentions and comments spike. Instagram Graph has a hard ceiling. If the webhook handler calls Graph inline, you 429, you retry, you get worse. Drop the webhook and you lose orders. Block the handler and Instagram retries you to death.

**What I did / decided**  
Split ingest from work. Webhook returns 200 and the payload hits Redis. A worker pulls the stream and spends a token before any Graph fetch. Mock limiter: 200 tokens an hour. State machine after that, not during HTTP. Same idea as the IARE lab note: token bucket on flaky external APIs so reliability is a queue problem, not a hope problem.

**What came of it**  
Ingest stays up during a spike; outbound stays inside the budget; events wait instead of vanishing. Simulator on the sitemap fires a burst so you can see ack-now, call-later. Academic write-up sits under Runtime as the 2025 webhook ingestion project.

---

## Bio

I am Bharadwaj Mittapelli, a CSE student at IARE. I care about what happens when two requests hit the same row, when a model has no AST, and when an API says 429. Java, Python, TypeScript. Postgres, Redis. FastAPI and Node. I have a Salesforce Agentforce Specialist cert; I treat agents like they will do the wrong thing unless you bound them.

I do not want a generic internship blurb. I want 15 minutes on a pipeline, a sandbox, or an agent boundary.

---

## Contact / CTA

**Do this:** mail me and ask for a 15-minute system review. Pick CrednPay, CodeCheck, Instagram ingest, or BlackSwan.

- Email: [bharadwajmittapelly@gmail.com](mailto:bharadwajmittapelly@gmail.com)  
- Phone: +91 63024 12421  
- GitHub: [github.com/BharadwajMittapelli](https://github.com/BharadwajMittapelli)  
- Sitemap: [bharadwajmittapelli.github.io/portfolio-sitemap](https://bharadwajmittapelli.github.io/portfolio-sitemap/)

---

## Before / after (generic AI vs edited)

**Before (generic AI)**  
“I am a results-driven full-stack developer passionate about leveraging cutting-edge technologies to deliver scalable, impactful solutions that drive business value and exceed stakeholder expectations.”

**After (mine)**  
“If two payment webhooks hit the same wallet, I lock the Postgres row. If a model is going to audit Solidity, it gets an AST first. That is the work. Email me if you want 15 minutes on it.”

---

## Sitemap coverage checklist

| Sitemap piece | Framed case |
|---|---|
| `/` landing + scheduler CTA | Bio + contact. Action = 15-min review. |
| `/systems` CrednPay | Case 1 |
| `/systems` CodeCheck | Case 2 |
| `/systems` E-commerce / Instagram | Case 3 |
| Internship (Flyrank / BlackSwan) | Case 0 |
| `/runtime` Agentforce + IARE | Bio + Case 3 academic beat |

Repo index (not extra sitemap tiles; do not pretend they are): PAD-ai, NutriTracker, shopping-assistant-agent, ambient-expense-agent, Face-Recognition, event-talks-app, google-cloud-serverless-app.
