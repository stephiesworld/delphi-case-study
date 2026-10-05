# Delphi: building an AI analyst in a day

*A case study in turning a dinner-table habit into a working research product, and in what it takes to trust work done by AI agents.*

**Live site:** [delphi-stocks.vercel.app](https://delphi-stocks.vercel.app)

---

## The idea

The best explanations of companies I've heard weren't in an app. They were stories. Starbucks had a new CEO from Chipotle. Lines at the drive-thru were too long, mobile orders were swamping baristas, and people who saw the line drove away. So the new CEO was cutting the menu and limiting customizations to make every drink faster. It made sense in a way a stock chart never does.

The Robinhood app gives you facts without the story. I wanted that kind of explanation for every company people actually own: Robinhood's 100 most popular stocks, each with a clear story, the latest earnings, and a recommendation.

I built Delphi in one long day, working with Claude Code. This is what we built, the decisions that shaped it, and the mistakes that taught us the most.

## What Delphi does

For each company:

- **The takeaway:** one headline a portfolio manager gets in ten seconds, then a short narrative.
- **The recommendation:** Buy, Hold, or Sell, with a 12-month target and how it compares with the S&P 500, plus the reason in one sentence.
- **Bull vs. bear:** the one question the market is arguing about, and each side's case.
- **The backstory:** the problem driving the company's strategy, why it matters, and what the company is doing about it.
- **Earnings:** what Wall Street expected, what happened, why the stock moved, and what it means.
- **What we're less sure about:** where to be skeptical.
- **Sources** for every claim, and a public track record of every rating.

Prices update every evening. When a company reports earnings, an AI agent rewrites its story and rating the next evening, on its own.

## How it was built: the turns that mattered

### 1. A template is the wrong answer

My first instinct was a standard framework: who's the CEO, what's their track record, what's the plan. It worked for Starbucks and nothing else. NVIDIA's story is a founder who has run it for 30 years. Micron's is a memory market that booms and busts. AMC's is pandemic debt.

So every story starts by working out three things:

- **What kind of story is this?** A turnaround, a founder-led leader, a boom-and-bust business, and so on.
- **Who are the real rivals?** You only understand Starbucks next to McDonald's and Dutch Bros.
- **What is the market actually arguing about?** One question, answerable with a number.

### 2. "It looks like AI made it"

The first design was cream background, elegant serif, gold accents. The default look of AI-built sites. I rejected it. We mocked up three trading-terminal styles and picked the most readable one, not the coolest. The two flashier options looked great and told me nothing.

The writing had the same problem. The first rewrite made it punchy ("Traffic is fixed. Margin isn't. That's the whole trade now."). An analyst would laugh at that. The second made it plain but dense with jargon ("upside requires box office to cut leverage without dilution"). I didn't know what that meant, and I'm the target reader.

The fix was a writing rule that serves two readers at once: a portfolio manager with ten seconds, and a smart friend with no finance background. Lead with the conclusion. Spell out every piece of shorthand. Say what each number means.

> **Before:** "Upside requires box office to cut leverage without dilution."
> **After:** "AMC needs a strong movie year to pay down debt without selling more shares."

### 3. Facts aren't a story

Even with clear writing, something was missing. The Starbucks page listed the steps (menu cut, more baristas, a four-minute target) but never said *why*. Every step was a response to one problem: slow morning lines were costing visits. Without that, "menu cut" is trivia.

So every story got a **backstory**: the problem, why it matters in dollars or customers, and what the company is doing about it. Every company has one. Once you know it, the strategy reads like a plot instead of a list.

### 4. Getting the recommendation right took four tries

A hedge-fund analyst asks "long, short, or neither?" Most Robinhood users never short, so the labels are Buy, Hold, and Sell, where Sell means "don't buy now, consider trimming."

The method went through four versions, and each failure taught something:

1. **Every stock came out Hold.** The research agents anchored every target on Wall Street's consensus and gave every scenario the same 25/50/25 odds. If you assume the market is right, you never disagree with it.
2. **Some came out absurd.** Fixing that produced a +116% expected return for Amazon. Its valuation history included years of unusually low profits, which made a "normal" valuation look huge.
3. **The agents improvised.** Each one made slightly different choices, so the ratings weren't comparable across companies.
4. **The version that works:** the math moved into code, identical for every company. Agents only gather standard inputs (estimates, analyst ranges, valuation history) and use judgment in two places: the odds, and the plain-English explanation. And every stock is compared with **the S&P 500 run through the same math.** Wall Street's estimates run optimistic for every company. Comparing against the index cancels that shared bias out. A Buy means "expected to beat the market," which is the question an investor actually asks.

### 5. Publishing without review means building guardrails

I chose to let ratings publish automatically. That is only reasonable with safety checks that run every time:

- The label is calculated, never chosen.
- Ratings pause after an earnings report until the story is refreshed, and after 120 days regardless.
- Extreme results pause instead of publishing, because they usually mean a bad input.
- Stocks listed less than two years aren't rated: no history to anchor on.
- Stock splits are detected automatically and flagged for rescaling.
- Every rating change is logged with the date, price, and S&P 500 level, and the log is never edited.

## Working with a team of AI agents

Most of the research was done by AI agents running in parallel: one per company to write, more to fact-check, more to audit. Several dozen in one day. That is how ten companies get researched in the time one would take.

The lessons were less about the agents and more about the system around them.

**The brief is the product.** Every quality jump came from better instructions: the voice rules, the backstory requirement, one valuation method for everyone. When an instruction was wrong, every agent faithfully did the wrong thing. One early rule for counting future share sales cancelled itself out in the math, and Lucid came out a high-conviction Buy. Corrected, it's a Sell.

**Checking catches numbers. It didn't catch framing.** Fact-check agents compared every headline number against company filings and found real errors: a share count that rose 74%, not 42%; debt of $98 billion, not $121 billion; a 64th consecutive dividend increase, not the 63rd. But I found the error that mattered most myself. The Starbucks story said U.S. transactions fell 10% "in Niccol's first quarter." The number was right. But Niccol started three weeks before that quarter ended. The decline happened under the previous CEO, and it's why he was replaced. Every fact was true, and the sentence still blamed the wrong person.

That led to a third kind of check: an **adversarial audit** whose only job is to attack each story's reasoning. Who was in charge when each number happened? Does each "because" follow from the evidence? What would an informed skeptic say is missing? On its first pass it found problems no number check could:

- **Coca-Cola:** the same error as Starbucks, in reverse. The story credited the new CEO with a strategy that started under his predecessor; he took over three days before the quarter ended. This time the audit caught it, not me.
- **Tesla:** the story said car profits were shrinking. Car margins actually rose; profit fell because operating expenses jumped 47% on AI research and stock pay.
- **Microsoft:** an 84% jump in signed contracts read as broad demand. Most of it was OpenAI; without OpenAI, growth was 25%.
- **NVIDIA:** a 70% growth forecast credited to the CEO was the finance chief's company guidance, and a quote attributed to NVIDIA came from U.S. officials.
- **Alphabet:** "ad profits no longer cover the spending" rested on one negative quarter; the past twelve months were still $53 billion positive.
- **AMD:** two "6 gigawatt" AI chip deals were presented as firm. Only the first gigawatt of each is binding.
- **Micron:** the 2023 memory bust was blamed on new factories. It was mainly customers cutting orders.

The audit also writes a "What we're less sure about" box for every story, so readers know where to be skeptical without needing their own background knowledge.

**Agents will do things you didn't ask for.** While downloading SEC filings, a few agents put my name and email in the request header, because the SEC asks automated tools for contact details. It went only to sec.gov and was low-risk, but they hadn't asked. Every brief now forbids it. The lesson: write down the rules you assume are obvious.

**Verification is the product, not a step.** The reason to trust Delphi isn't that AI wrote it. It's that every claim has a source, every number was checked against filings, every story's logic was attacked, uncertainties are shown on the page, and every rating is scored in public against the market.

## How it runs

- **Daily, after the market close:** a scheduled job pulls prices and earnings data for all 100 stocks, recomputes every rating against the new prices, and logs any changes.
- **Every weekday evening:** a cloud agent checks which companies have reported earnings, rewrites their stories and rating inputs, passes the checks, and publishes.
- **Data costs:** it runs on a free market-data tier (250 calls a day) by never calling the data provider when someone visits a page. Pages read a saved daily snapshot.

## What's next

- Coverage for the rest of the 100, in order of popularity.
- A way for readers to flag mistakes.
- Time. The track record is the real test, and it takes months to mean anything.

---

*Delphi explains businesses. Its ratings are model-generated and are not personalized investment advice.*
