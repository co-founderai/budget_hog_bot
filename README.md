Personal Budget Telegram Bot

A Telegram bot to manage your personal budget with free-form message input, automated categorization, and beautiful visual reports. Built on a n8n workflow engine, Supabase for data storage, and uses HTML templates + Chart.js + Google Sankey + Gotenberg/Stirling for PDF/image report generation.

🚀 Features
	•	Quick Entry: Send messages like income $2000 salary or spent $30 restaurant to record transactions.
	•	Automated Parsing: LLM-Agent extracts type, amount, currency, category, description, and date or asks for clarification.
	•	Categorization: Assigns transactions to predefined categories (e.g., Groceries, Salary).
	•	Scheduled Reports: Weekly and monthly automated reports with interactive charts.
	•	Visual Reports: PDF or image exports with:
	•	Sankey flow: income → expenses
	•	Donut charts: income vs. expense distribution
	•	Line chart: balance over time
	•	Premium Subscriptions: Stripe integration for monthly/yearly plans; keeps profiles.premium flag in sync.
	•	Savings Tracker: Progress bar for custom saving goals.

📦 Architecture

Telegram Bot
   ↕ Webhook
     n8n Workflows
       • Message Parsing → LLM → Supabase
       • Cron Triggers → Data Aggregation → HTML Templates → Gotenberg/Stirling
       • Stripe Webhook → Supabase (premium flag)
Supabase DB
HTML Templates + Chart.js + Sankey
Gotenberg (HTML→PDF)
Stirling (HTML→Image)

🔧 Prerequisites
	•	Node.js & Docker (for n8n)
	•	Supabase account
	•	Stripe account
	•	Gotenberg & Stirling services (Docker containers)
	•	OpenAI API key (for LLM-Agent)

🛠 Setup
	1.	Clone Repo

git clone https://github.com/yourorg/budget-telegram-bot.git
cd budget-telegram-bot


	2.	Configure Supabase
	•	Create a new project.
	•	Run SQL schema in Supabase SQL editor:

create extension if not exists pgcrypto;
create table profiles (
  id uuid primary key default gen_random_uuid(),
  telegram_id bigint unique not null,
  stripe_customer_id text,
  stripe_subscription_id text,
  premium boolean default false,
  saving_goal numeric,
  created_at timestamp default now()
);

create table transactions (
  id uuid primary key default gen_random_uuid(),
  telegram_id bigint not null references profiles(telegram_id),
  type text check(type in ('income','spending','saving')) not null,
  amount numeric not null,
  currency text not null default 'USD',
  category text,
  description text,
  occurred_at date not null default current_date,
  created_at timestamp default now()
);


	•	Enable RLS and policies for profiles and transactions.

	3.	Deploy n8n
	•	Run Docker:

docker run -it --rm \
  -p 5678:5678 \
  -e N8N_BASIC_AUTH_ACTIVE=true \
  -e N8N_BASIC_AUTH_USER=admin \
  -e N8N_BASIC_AUTH_PASSWORD=strongpassword \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n


	•	Import workflows in .n8n/workflows directory.
	•	Set environment variables for Supabase, Stripe, OpenAI.

	4.	Configure Stripe
	•	Create Payment Links or Checkout Sessions for your plans.
	•	Add webhook endpoint: https://your.n8n.domain/webhook/budgethog.
	•	Subscribe to events: checkout.session.completed, invoice.paid, customer.subscription.updated, customer.subscription.deleted.
	5.	Run Gotenberg & Stirling

docker run -d -p 3000:3000 thecodingmachine/gotenberg:latest
docker run -d -p 8080:8080 stirling/example:latest



📋 n8n Workflow Highlights

1. Message Handler
	•	Webhook → LLM-Agent → Supabase → Telegram confirmation.

2. Report Generator
	•	Cron (Weekly/Monthly) → Compute Period → Supabase Query → Function (aggregate) → DocumentGenerator (HTML template) → Gotenberg → Telegram.

3. Savings Tracker
	•	User enters saving → transactions.type=‘saving’ → accumulate via Function node → HTML template for progress bar.

4. Stripe Integration
	•	/subscribe command → Send Payment Link
	•	Webhook → Switch by event.type → Supabase update premium flag.

👥 Contributing
	1.	Fork the repository
	2.	Create a feature branch (git checkout -b feature/foo)
	3.	Commit your changes (git commit -am 'Add foo')
	4.	Push to the branch (git push origin feature/foo)
	5.	Open a Pull Request

⚖️ License

MIT © Kir Leshkevich
