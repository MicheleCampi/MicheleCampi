Hi, I'm Michele 👋
Operations Intelligence Engineer
Manufacturing Optimization • OR-Tools • MCP Servers
I build computational decision systems that optimize production, quantify risk, and prescribe actions for European mid-market manufacturers.

🧠 Featured work
OptimEngine — 11-tool mathematical optimization service
A production solver for FJSP scheduling, CVRPTW routing, bin packing, Pareto multi-objective analysis, Monte Carlo risk simulation with CVaR metrics, parametric sensitivity, and prescriptive intelligence.
Built on Google OR-Tools CP-SAT v9.0.0, exposed via REST API and dual-stack MCP (open SSE at /mcp, OAuth 2.1-gated Streamable HTTP at /mcp/v2). Live and verified on automotive manufacturing scenarios — optimal schedules in 0.04–2 seconds.
Stack: Python 3.12 · FastAPI · OR-Tools · FastMCP · ScaleKit OAuth · Pydantic · Railway
optim-arc-v3 — x402/Nanopayments gateway on Arc
A Next.js seller-side implementation of Circle Nanopayments for OptimEngine, exposing 10 paid optimization endpoints for autonomous AI agents. Forked from circlefin/arc-nanopayments (Apache-2.0).
Live on Vercel: optim-arc-v3.vercel.app. Each endpoint returns HTTP 402 with valid x402 v2 + Circle GatewayWalletBatched payment requirements.

📰 Recent articles
What an OR-Tools solver finds in a week of contract packaging — and what the planner usually misses (April 2026)
A synthetic but realistic case study on a mid-market contract packager: eight customer orders, six production lines, sequence-dependent setup times. The expert manual schedule lands around 190 quarter-hours of makespan. OptimEngine returns the proven optimum in ten milliseconds: 161 quarters. The interesting finding isn't the 15% gain — it's what the solver shows about hidden capacity that the manual planner can't see.
Three production scheduling failures I've seen, and the math that would have caught them (April 2026)
Three chronic operational failures that repeat across European mid-market manufacturers, regardless of sector. Each had a visible symptom in the dashboards, a hidden cause nobody was tracking, a management reaction that addressed the symptom, and a quantitative method that would have caught the cause.
Why your AI assistant can't actually plan your factory (April 2026)
A real test on a synthetic SME manufacturer — 15 CNC machines, 6 automotive orders. Compares OptimEngine's production solver to a generic AI assistant running CP-SAT in its sandbox. The gap is sharper than you'd expect.
Exposing a math solver as Circle Nanopayments (April 2026)
Forking Circle's arc-nanopayments sample to expose 10 optimization endpoints as gasless USDC paid resources on Arc testnet. Pattern, code, deploy, and the lessons learned along the way.
How I exposed OR-Tools as a production MCP server (April 2026)
Building a Model Context Protocol server that wraps Google's OR-Tools constraint solver. Why MCP fits decision systems, what it took to make it production-grade, and what changes when AI agents can call your solver in natural language.

🏭 Background
7+ years in operations controlling within manufacturing, with a Finance and Risk Management degree (2013). I bridge deep domain expertise in production operations with self-taught technical capabilities in optimization, graph intelligence, and systems building.
The path is uncommon — I built OptimEngine because I knew exactly what production controllers need and what manufacturing software typically gets wrong.

⛓️ Active in agent economy
ERC-8004 Agent #22518 on Base L2 — permanent on-chain identity for autonomous agent operations. OptimEngine endpoints are accessible via x402-native paid APIs on Arc testnet (Circle Nanopayments) and Base mainnet, designed as primitives for the emerging "markets of calculations and decisions" between AI agents.

📫 Open to
I help European mid-market companies — particularly manufacturers and SMEs in the €5-50M revenue range — make sense of operations digitalization. The specific shapes this takes:

Operational audit + quantitative analysis — focused assessment of a scheduling, planning, or capacity utilization problem, ending in a written report with findings, modeled alternatives, and ROI estimates
Working prototypes and proofs of concept — building a functioning prototype that demonstrates whether an idea is viable, before committing to a full project
Technical roadmap and architecture review — written documents that help business teams evaluate projects, with proposed architecture, step-by-step roadmap, cost and effort estimates
Bridge work between business and technical teams — ongoing presence in projects where the business knowledge sits with operations or controlling but the technical execution sits with internal IT or external consultants

European mid-market focus (Italy, Germany, DACH primarily) — open to remote engagements globally.
If your operations plan weekly production with spreadsheets and intuition, or you're integrating optimization decisions into AI agent pipelines, reach out.

Iterating OptimEngine in public. Follow this profile for new articles, integrations, and case studies.
