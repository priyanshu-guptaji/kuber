Tech & Libraries

React (functional components + hooks)

Tailwind CSS (clean pastel palette + navy/charcoal text)

React Router

Recharts (or Chart.js) for visualizations

Framer Motion (optional but preferred)

date-fns or dayjs

papaparse or minimal CSV util

localStorage as persistence (seeded)

Optional: a small confetti library for celebratory moments

UX Goals

Pastel + professional aesthetic, light rounded corners, soft shadows, glass highlights

Responsive (desktop / tablet / mobile)

Friendly microcopy + occasional emoji

Accessibility: semantic HTML, labels, keyboard nav

Pages & Behaviors (must be functional)
Auth (Welcome / Login / Signup)

Simulated auth: lightweight forms. After successful "login", redirect to /dashboard. Use placeholder user Abhishek from mock data. No real server.

Dashboard /dashboard

Greeting (e.g., “Good evening, Abhishek — You’re doing great 💚”)

Stat cards: Total Income, Total Expense, Net Balance, Monthly Budget Remaining (calculated).

Pie chart: category-wise expenses

3-month trend chart

Upcoming Bills panel (cards sorted by nearest due date)

Quick action buttons: Add Expense, Add Goal, Add Bill (open modals)

Expense Tracker /expenses

Full CRUD for expenses (Add/Edit/Delete)

Fields: date, amount, category, payment mode, note

Table for desktop + compact card list for mobile

Monthly Budget progress bar that updates live when expenses change

Savings Goals /goals

CRUD

"Add to goal" modal increments saved. When reaching target: confetti + badge award

Badges stored in data

Subscriptions & Bills (/subscriptions, /bills)

Add entries with name, amount, cycle, nextDueDate

Days-left counter, sorted by nearest due

Mark paid flow: option to (A) advance nextDueDate automatically for recurring and/or (B) add a linked expense automatically. Both options must be toggleable.

Debt & Split Tracker /debts

Tabs: I Owe / Owed to Me

CRUD + mark as paid (creates expense or updates balances)

Insights /insights

6-month Income vs Expense chart (line or bar)

Calendar heatmap or daily activity view (highlight high-spend days)

Animated counters: This Month Spent / Saved / Remaining

Budget Challenge /challenge

Create challenge (name, monthly limit)

Track progress automatically from expenses

Progress ring + status, award badges on success

Settings /settings

Theme selector: Light / Dark / Dreamy

Export CSV buttons for expenses, goals, bills

Reset to mock data (with confirm modal)

Key Requirements

Seed on first run. Example:

if (!localStorage.getItem('pfs_data')) {
  localStorage.setItem('pfs_data', JSON.stringify(MOCK_DATA));
}
const db = JSON.parse(localStorage.getItem('pfs_data'));


Central useData() React context/hook that reads/writes to localStorage and triggers re-renders.

All CRUD ops update localStorage immediately and update charts / dashboard.

Charts should re-render on data change.

Recurring bill stamp-forward logic + add-expense option.

CSV export triggers immediate download.

Demo Steps widget (5 clickable steps) visible to judges.

Provide README with run instructions and demo steps.

MOCK_DATA (seed EXACTLY)
{
  "user": { "id": "u1", "name": "Abhishek", "email": "abhishek@example.com" },
  "settings": { "theme": "dreamy", "currency": "INR", "monthlyBudget": 25000 },
  "income": [
    { "id":"inc1", "source":"Pocket Money","amount":20000,"date":"2025-09-01" },
    { "id":"inc2", "source":"Freelance App","amount":8000,"date":"2025-09-15" }
  ],
  "expenses": [
    { "id":"e1","date":"2025-10-02","amount":240,"category":"Food","mode":"UPI","note":"Lunch Zomato" },
    { "id":"e2","date":"2025-10-04","amount":1200,"category":"Transport","mode":"Card","note":"Monthly pass" },
    { "id":"e3","date":"2025-10-07","amount":599,"category":"Subscriptions","mode":"Card","note":"Netflix" },
    { "id":"e4","date":"2025-10-10","amount":3500,"category":"Shopping","mode":"Card","note":"Shoes" }
  ],
  "goals": [
    { "id":"g1","name":"Trip to Goa","target":10000,"saved":7500 },
    { "id":"g2","name":"New Laptop","target":60000,"saved":15000 }
  ],
  "bills": [
    { "id":"b1","name":"Electricity","amount":1800,"dueDate":"2025-10-20","recurring":"monthly" },
    { "id":"b2","name":"WiFi","amount":699,"dueDate":"2025-10-18","recurring":"monthly" }
  ],
  "subscriptions": [
    { "id":"s1","name":"Spotify","amount":99,"nextDue":"2025-11-05","cycle":"monthly" },
    { "id":"s2","name":"Netflix","amount":599,"nextDue":"2025-11-25","cycle":"monthly" }
  ],
  "debts": [
    { "id":"d1","name":"Rohit","amount":1200,"type":"owed_to_me","due":"2025-11-01","note":"Dinner" },
    { "id":"d2","name":"Sara","amount":500,"type":"i_owe","due":"2025-10-30","note":"Movie" }
  ],
  "challenges": [
    { "id":"c1","name":"Under ₹5000 Oct", "limit":5000, "month":"2025-10", "progress": 3800 }
  ],
  "badges": ["first-saver"]
}

Extra feature (AI Assistant)

Add an on-page AI Assistant chatbot that summarizes the entire dataset (income, expenses, bills, goals, badges, challenges, trends) and answers quick questions (e.g., “How much did I spend on subscriptions this month?”, “Which bills are due in next 7 days?”, “Summarize my goals and progress”). See AI Assistant spec below for details.

Deliverable: A GitHub-style repo with README that documents run steps, where mock data is seeded, and the Demo Steps visible in the app.

2 — Implementation guide (what to add in hackathon, step-by-step)

Below are implementation details, code patterns, and sample utility functions to include in your repo. Use these in your implementation; judges will use them to verify flows.

Architecture & State

src/context/DataContext.jsx or src/hooks/useData.js: central context exposing db and methods:

get(), set(), addExpense(), editExpense(), deleteExpense(), addGoal(), addToGoal(), awardBadge(), addBill(), markBillPaid(options), resetToMock(), exportCSV(type), etc.

Always persist to localStorage.setItem('pfs_data', JSON.stringify(db)) after mutations.

Use useReducer inside the context to ensure predictable updates.

Seed logic (exact)
// src/data/mock.js
export const MOCK_DATA = { ... } // exact JSON from prompt

// src/context/DataProvider.jsx (simplified)
useEffect(() => {
  if (!localStorage.getItem('pfs_data')) {
    localStorage.setItem('pfs_data', JSON.stringify(MOCK_DATA));
  }
  setDb(JSON.parse(localStorage.getItem('pfs_data')));
}, []);

Currency + Date helpers
// src/utils/format.js
import { format, parseISO } from 'date-fns';
export const formatCurrency = (v, currency='INR') => {
  return new Intl.NumberFormat('en-IN', { style: 'currency', currency }).format(v);
};
export const fmtDate = (iso) => format(parseISO(iso), 'yyyy-MM-dd');

Example: useData API (excerpts)
const addExpense = (expense) => {
  const newExp = { id: 'e' + Date.now(), ...expense };
  db.expenses.push(newExp);
  saveDb();
};

// mark recurring bill paid:
const markBillPaid = ({ billId, advanceNext = false, createExpense = false }) => {
  const bill = db.bills.find(b => b.id === billId);
  if (!bill) return;
  if (createExpense) {
    db.expenses.push({
      id: 'e' + Date.now(),
      date: new Date().toISOString().slice(0,10),
      amount: bill.amount,
      category: 'Bills',
      mode: 'Card',
      note: `Paid ${bill.name}`
    });
  }
  if (advanceNext && bill.recurring) {
    const next = addPeriodToDate(bill.dueDate || bill.nextDue, bill.recurring);
    bill.dueDate = next;
  }
  saveDb();
};


addPeriodToDate() should add 1 month / 3 months / 12 months depending on recurring.

CSV Export helper
export function exportCSV(filename, rows, headers) {
  const csv = [headers.join(','), ...rows.map(r => headers.map(h => JSON.stringify(r[h] ?? '')).join(','))].join('\n');
  const blob = new Blob([csv], { type: 'text/csv' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url; a.download = filename;
  a.click();
  URL.revokeObjectURL(url);
}


Call with mapped rows (apply formatCurrency).

Charts

Lazy load chart components (React.lazy) for performance.

Recharts examples: Pie for categories, Line for income vs expense, Bar for monthly totals.

Important: derive chart data via memoized selectors that recompute when db changes.

Recurring bill date advance

Use date-fns:

import { addMonths } from 'date-fns';
function addPeriodToDate(dateStr, cycle) {
  const d = parseISO(dateStr);
  if (cycle === 'monthly') return format(addMonths(d, 1), 'yyyy-MM-dd');
  if (cycle === 'quarterly') return format(addMonths(d, 3), 'yyyy-MM-dd');
  if (cycle === 'yearly') return format(addMonths(d, 12), 'yyyy-MM-dd');
  return dateStr;
}

Goal completion

When goal.saved >= goal.target:

award badge (db.badges.push('goal-completed-' + goal.id) or a friendly name)

show confetti / Framer Motion celebration.

Demo Steps widget

Visible on Dashboard; 5 clickable step buttons that programmatically navigate and execute small actions (e.g., mark a specific bill paid using markBillPaid, or open add goal modal pre-filled).

Accessibility & keyboard

Ensure modals trap focus, all buttons are button elements, inputs have labels and aria-* attributes.

3 — AI Assistant Chatbot feature (FULL spec + implementation options)

You asked for: “add a ai assistant chatbot that summarise the whole data from the data here — write and explain all the every details in it i have to add in hackathon” — below is a complete spec you can implement without a backend (local summarizer) plus an optional LLM integration that uses an external API (OpenAI or similar) if you want richer natural language capabilities.

Goals of the AI Assistant

Provide immediate, accurate summaries of the seeded pfs_data.

Answer quick queries (predefined Qs) like:

“Show me upcoming bills due in 7 days”

“How much was spent on Subscriptions this month?”

“Summarize my goals and progress”

“Give a monthly snapshot for October 2025”

Provide a one-click “Export summary” (printable report page).

Optional: wire to LLM for natural language flexibility.

UI

Floating bottom-right chat widget (small bubble). Click opens chat drawer.

Top: “AI Assistant — Abhishek’s Finance” with small profile image.

Input: prompt box + quick suggested prompts chips (prepopulated selectable)

Responses: show cards with sections, charts, or text. Include a “Copy summary” and “Print” button.

Accessibility: keyboard focus, aria-live region for responses.

Implementation Option A — Local summarizer (no external calls) — recommended for hackathon

This computes and summarizes everything from db synchronously and is fully offline.

Key functions

summarizeAll(db) → returns structured summary:

{
  user: { name, email },
  totals: { income, expense, net, budgetRemaining },
  topCategories: [{category, amount}],
  upcomingBills: [{name, amount, dueInDays}],
  goals: [{name, target, saved, pct}],
  debts: { iOwe: total, owedToMe: total },
  challenges: [{name, limit, progress, pct}],
  badges: [...],
  insights: { highSpendDays: [...], monthTrend: [{month, income, expense}] }
}


answerQuery(db, query) — basic NLP by keyword rules:

If query contains subscriptions or netflix: filter expenses/subscriptions by category/name and compute totals.

If contains due or next 7 days: compute bills with dueDate within 7 days.

If contains export or print: prepare printable report.

Fallback: show full summary.

Example summarizeAll output (using your mock data):

Totals: Income = 28000, Expenses = 5139, Net = 22861, Monthly budget remaining = 25000 - (expenses this month) => calculate per current month (Oct 2025).

Upcoming bills: WiFi — ₹699 due 2025-10-18 (X days), Electricity — ₹1800 due 2025-10-20.

Goals: Trip to Goa — 75% saved (₹7,500/₹10,000), New Laptop — 25% saved.

Badges: first-saver

Challenges: “Under ₹5000 Oct” — limit ₹5000, progress ₹3,800 (76% used)

(You should compute exact day counts with date-fns.)

UI Behavior

When user types “Summarize my finances for Oct 2025”, answerQuery calls summarizeAll(db, { month: '2025-10'}) and returns a nicely formatted response with bullet points and small charts (mini-sparkline).

Why local summarizer?

Instant, deterministic, works offline — perfect for hackathon judges evaluating functionality. No API keys needed.

Implementation Option B — Optional: LLM-powered responses (richer but external)

If you want natural language answers, provide metadata to the model rather than raw DB to avoid token bloat.

Compose a structured prompt that includes:

Short system instruction: “You are an assistant summarizing financial data for a user called Abhishek. Use INR currency. Use friendly tone.”

JSON payload of computed aggregates from summarizeAll(db) (not raw full DB).

Example Q&A pairs to show desired style.

Make a client-side POST to an LLM API (must keep API key safe — for hackathon you can use proxy or serverless function). If using only client, warn about exposing keys — judges may accept a readme note.

Sample prompt template for LLM:

System: You are a friendly financial assistant. Use INR notation and short bullet lists.
User: Here is the user's finance summary JSON: <INSERT_AGGREGATE_JSON>
User: Answer: "Summarize overall finances and list next 3 upcoming bills."


Safety/notes: For hackathon deliverable, include both local summarizer and optional LLM code paths; default to local summarizer.

Chatbot Component Outline (React)

<AiAssistant> — main widget: handles open/close, input, quick chips

useAiAssistant(db) hook — exposes getSummary(), handleQuery(q), useStreamedResponse if using LLM streaming

If using local summarizer, handleQuery calls local functions and returns markdown/text

If LLM path chosen, handleQuery sends aggregated JSON to backend / API and returns model text

Example local summarizer code (simplified)
function summarizeAll(db) {
  const totalIncome = db.income.reduce((s,i)=>s+i.amount,0);
  const totalExpense = db.expenses.reduce((s,e)=>s+e.amount,0);
  const net = totalIncome - totalExpense;
  const categories = db.expenses.reduce((acc,e)=>{
    acc[e.category] = (acc[e.category]||0) + e.amount;
    return acc;
  }, {});
  const topCategories = Object.entries(categories).map(([k,v])=>({k,v})).sort((a,b)=>b.v-a.v).slice(0,5);
  const upcomingBills = db.bills.map(b=>({name:b.name, amount:b.amount, dueDate:b.dueDate, daysLeft: daysBetween(new Date().toISOString().slice(0,10), b.dueDate)})).sort((a,b)=>a.daysLeft-b.daysLeft);
  return { totalIncome, totalExpense, net, topCategories, upcomingBills, goals: db.goals.map(g=>({...g, pct: Math.round(g.saved/g.target*100)})), badges: db.badges || [] };
}

Example canned quick prompts (chips)

“Show me this month’s snapshot”

“Upcoming bills in next 7 days”

“How much on Subscriptions this month?”

“Summarize my goals”

“Export summary”

Example summary text (what judges will see)

Monthly Snapshot — Oct 2025

Income: ₹28,000 — Expenses: ₹5,139 — Net: ₹22,861

Top categories: Shopping ₹3,500; Transport ₹1,200; Subscriptions ₹599; Food ₹240

Upcoming bills: WiFi ₹699 — due Oct 18 (in 3 days); Electricity ₹1,800 — due Oct 20 (in 5 days)

Goals: Trip to Goa — 75% complete (₹7,500/₹10,000). Add ₹2,500 to complete.

Active challenge: Under ₹5000 Oct — limit ₹5,000; progress: ₹3,800 (76%)

(Format currency, include small badges icons, and a print/export button.)

4 — Sample README (paste into your repo)
Personal Finance Simplifier — Frontend (Hackathon)

Run

git clone ...

cd personal-finance-simplifier

npm install

npm start

What to check

First run auto-seeds localStorage['pfs_data'] with mock data from /src/data/mock.js.

Demo Steps panel (bottom-left of dashboard) lists quick steps to verify app flows.

Demo Steps (clickable inside app)

Open Dashboard (see totals & charts)

Mark Netflix subscription/bill paid (opens mark-paid modal and optionally adds expense)

Add ₹100 to Trip to Goa goal (open goals → add to goal) — watch confetti when target hit

Export expenses CSV (Settings → Export)

Open AI Assistant → ask “Summarize my finances for Oct 2025”

Where mock data is seeded

/src/data/mock.js — exact JSON.

Seed logic in /src/context/DataProvider.jsx — if no localStorage['pfs_data'] found, it writes mock.

AI Assistant

Built-in local summarizer: no external keys required.

Optional LLM integration: see /src/ai/README.md for instructions and a prompt template (note: if you use an external API, do not commit secrets).

5 — Judge checklist (quick verification)

 App auto-seeds data on first run (inspect localStorage)

 Dashboard: totals compute correctly from mock data

 Expenses page: Add/Edit/Delete works and updates dashboard

 Mark a subscription/bill paid: nextDue updates, expense added when chosen

 Goals: add to goal and reach a goal → confetti + badge created

 CSV export: downloads correct CSV

 Settings: Reset to Mock Data resets localStorage and UI

 AI Assistant: open widget → click “Summarize” → see correct values from mock data

 Demo Steps: clicking each step performs the actions described

 Responsive views: check mobile cards and desktop table

6 — Example sample outputs for hackathon submission (paste into your presentation)

AI Assistant sample response (from mock data):

Personal Finance Snapshot — Oct 2025

Income (so far): ₹28,000

Expenses (so far): ₹5,139

Net: ₹22,861

Monthly budget: ₹25,000 → Remaining: ₹19,861

Top spending categories: Shopping ₹3,500; Transport ₹1,200; Subscriptions ₹599; Food ₹240

Upcoming bills (nearest): WiFi — ₹699 (due 2025-10-18 — 3 days), Electricity — ₹1,800 (due 2025-10-20 — 5 days)

Goals: Trip to Goa — 75% complete (₹7,500 / ₹10,000). New Laptop — 25% complete.

Active challenge: "Under ₹5000 Oct" — limit ₹5,000; progress ₹3,800 (76% used).

Badges: first-saver
