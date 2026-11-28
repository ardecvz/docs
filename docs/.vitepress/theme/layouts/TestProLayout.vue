<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'
import TestProCalculator from '../components/TestProCalculator.vue'

// Eval bench results: geometric mean speed-up of 3 trials per app and arm.
const benchResults = [
  { app: 'Mastodon (models)', alone: 3.59, pro: 3.84 },
  { app: 'Mastodon (requests)', alone: 2.22, pro: 2.48 },
  { app: 'Discourse (jobs and integrity)', alone: 2.74, pro: 2.15 },
  { app: 'Forem (models)', alone: 1.53, pro: 1.73 },
  { app: 'Forem (admin requests)', alone: 2.04, pro: 2.57 },
  { app: 'Forem (system specs)', alone: 2.55, pro: 2.91 },
  { app: 'Redmine 7 (Rails 8.1, unit tests)', alone: 2.09, pro: 3.47 },
  { app: 'Redmine 5 (Rails 6.1, unit tests)', alone: 1.16, pro: 1.68 },
  { app: 'Lobsters (whole suite)', alone: 1.22, pro: 1.45 },
  { app: 'Consul (models)', alone: 1.62, pro: 2.36 },
  { app: 'HCB (models)', alone: 1.48, pro: 2.20 },
  { app: 'Evil Martians CRM (whole suite)', alone: 1.02, pro: 4.44 },
]

const AXIS_MAX = 5
const ticks = [0, 1, 2, 3, 4, 5]
const position = (value: number) => `${(value / AXIS_MAX) * 100}%`
const times = (value: number, digits = 2) => `${value.toFixed(digits)}x`

const medianOf = (values: number[]) => {
  const sorted = [...values].sort((a, b) => a - b)
  const half = sorted.length / 2
  return sorted.length % 2 ? sorted[Math.floor(half)] : (sorted[half - 1] + sorted[half]) / 2
}
const speedups = {
  alone: benchResults.map((result) => result.alone),
  pro: benchResults.map((result) => result.pro),
}
const ahead = benchResults.filter((result) => result.pro > result.alone)
const best = benchResults.reduce((top, result) => (result.pro / result.alone > top.pro / top.alone ? result : top))
const gain = (result: (typeof benchResults)[number]) => `${Math.round((result.pro / result.alone - 1) * 100)}%`
// Median agent minutes per run.
const agentMinutes = { alone: 40, pro: 34 }

// Cyber week: 50% off the first year until the end of 11 Dec 2026, Pacific time.
// Checked again in the browser, so the regular prices come back without a rebuild.
const PROMO_ENDS_AT = Date.parse('2026-12-12T08:00:00Z')
const promoActive = ref(Date.now() < PROMO_ENDS_AT)

// Launch: 1 Dec 2026, midnight Pacific. The hero stamp reads "New" from then on.
const LAUNCH_AT = Date.parse('2026-12-01T08:00:00Z')
const launched = ref(Date.now() >= LAUNCH_AT)
onMounted(() => {
  promoActive.value = Date.now() < PROMO_ENDS_AT
  launched.value = Date.now() >= LAUNCH_AT
})

// Stripe Payment Links, one per plan (yearly subscriptions). Paste the live URLs here.
// During cyber week the link pre-fills the promotion code; Stripe checks its expiry too.
const CHECKOUT = {
  starter: 'https://buy.stripe.com/cNi14n7KUeHc8tM8hSdUY04',
  team: 'https://buy.stripe.com/bJedR9fdm42ybFYaq0dUY03',
  business: 'https://buy.stripe.com/bJe8wP2qAbv0h0i55GdUY02',
}
const PROMO_CODE = 'CYBER50'
const checkoutUrl = (plan: keyof typeof CHECKOUT) =>
  promoActive.value ? `${CHECKOUT[plan]}?prefilled_promo_code=${PROMO_CODE}` : CHECKOUT[plan]

const playing = ref(false)
const prefersReducedMotion = () => window.matchMedia('(prefers-reduced-motion: reduce)').matches

const clip = [
  { kind: 'prompt', text: '> Our test suite is slow. Make it faster.' },
  { kind: 'step', text: '● Baseline: 3:32, all green' },
  { kind: 'step', text: '● Profiling where the time goes...' },
  { kind: 'step', text: '● Checking for known issues...' },
  { kind: 'step', text: '● Optimizing files, applying best practice fixes...' },
  { kind: 'step', text: '● Writing a project skill to prevent future slow tests...' },
  { kind: 'done', text: 'Tests 4.6x faster: 3:32 -> 0:46, all green' },
]
const clipStep = ref(0)
// The timer in the terminal bar runs from the baseline to the result as the steps appear.
const CLIP_FROM = 212
const CLIP_TO = 46
const timerText = computed(() => {
  const share = Math.min(1, Math.max(0, (clipStep.value - 1) / (clip.length - 2)))
  const seconds = Math.round(CLIP_FROM - (CLIP_FROM - CLIP_TO) * share)
  return `${Math.floor(seconds / 60)}:${String(seconds % 60).padStart(2, '0')}`
})
const CLIP_FALLBACK_MS = 2000
let clipTimer: number | undefined

const playClip = () => {
  if (clipTimer !== undefined) return
  clipStep.value = 1
  clipTimer = window.setInterval(() => {
    clipStep.value += 1
    if (clipStep.value >= clip.length) window.clearInterval(clipTimer)
  }, 800)
}

const play = () => {
  playing.value = true
  if (prefersReducedMotion()) {
    clipStep.value = clip.length
  } else {
    window.setTimeout(playClip, CLIP_FALLBACK_MS)
  }
}

onMounted(() => {
  if (document.visibilityState === 'visible') play()
  else document.addEventListener('visibilitychange', play, { once: true })
})

onBeforeUnmount(() => {
  document.removeEventListener('visibilitychange', play)
  window.clearInterval(clipTimer)
})

const goToPricing = () => {
  document.getElementById('pricing')?.scrollIntoView({ behavior: prefersReducedMotion() ? 'auto' : 'smooth' })
  history.replaceState(null, '', '#pricing')
}

const faq = [
  { q: 'What does it work with?', a: 'Rails apps with RSpec or Minitest. Agents: Claude Code, Codex, Cursor and OpenCode, or any other MCP client. The agent installs anything else it needs.' },
  { q: 'What exactly do I install?', a: 'A private gem from GitHub Packages. After you buy, we email your access token and install steps within one business day. Install it on each developer\'s machine, or in your Gemfile for the whole team. Running test-pro install adds the TestPro MCP server and its skill to your agent\'s config.' },
  { q: 'Does it change my app?', a: 'Fixes stay on the test side: test config, spec support, factories and specs. App code changes only when a fix leaves the app\'s behavior the same, for example a setting that applies in tests only. The summary lists every app change.' },
  { q: 'Will it break my tests?', a: 'No test is skipped or removed and no assertion is weakened. Each fix is kept only after the suite passes with it. Tests that already failed before the run are recorded as known failures, and flaky ones are listed in the summary.' },
  { q: 'How long does a run take?', a: 'As long as you allow. Without a budget, the agent gets about 15 times your suite\'s run time, 30 minutes at least, and starts wrapping up at 80% of it. In our benchmark a run took 34 minutes on average.' },
  { q: 'Does my code leave my machine?', a: 'The gem sends nothing anywhere. The whole catalog ships inside the gem, so your agent reads it locally. Your agent\'s model provider sees what it always sees.' },
  { q: 'What if my suite is already fast?', a: 'Run it once: it follows TestProf\'s own profiling flow and shows whether there is time left to win. If it finds nothing worth keeping, ask for the refund.' },
  { q: 'Why pay every year after the first run?', a: 'Your app keeps moving: new gems, Rails upgrades, new kinds of tests. Each new version of the gem brings new causes and fixes to the catalog, so a rerun after an upgrade or a burst of new specs finds what changed.' },
  { q: 'What happens if I don\'t renew?', a: 'Everything the runs produced stays in your repo: the fixes, the summaries and the project skill. New versions of the gem need an active subscription.' },
  { q: 'How is it different from TestProf?', a: 'TestProf is the open-source toolbox and always stays free. TestPro is the gem your agent works through: the catalog of known causes, the playbooks, and tools that make the agent measure every fix before keeping it. Both come from the TestProf team.' },
  { q: 'Who counts as a developer?', a: 'Your engineering headcount, as you count it. We don\'t check; we trust you.' },
  { q: 'We have more than 100 developers. Which plan fits?', a: 'The speedup sprint. It includes 12 months of TestPro for your whole engineering team, with no developer cap, and our engineers work on your suite with you for two weeks, longer for very large suites. After the first year, the licence renews at $9,900 a year.' },
  { q: 'Can I run it in CI?', a: 'Yes. The gem installs with a token, like other private gems. Install it in the CI job and run your agent headless there, so new slow tests do not creep in.' },
]
</script>

<template>
  <div class="pro">
    <!-- Hero -->
    <section class="hero" :class="{ playing }">
      <div class="hero-inner">
        <div class="hero-copy">
          <span class="title" aria-hidden="true">
            TestPro<span class="f">f</span>
            <span class="stamp" @animationstart="playClip">{{ launched ? 'New' : 'Early access' }}</span>
          </span>

          <h1>Your coding agents run your tests 3 to 10 times per task. Make them twice as fast.</h1>

          <p class="hero-lead">
            Install one gem, ask Claude Code, Codex or Cursor to speed up your Rails tests, and review a branch where every fix is measured.
          </p>

          <div class="hero-actions">
            <a class="btn btn-brand" href="#pricing" @click.prevent="goToPricing">See plans</a>
            <p class="hero-note">From $395 a year</p>
          </div>

          <ul class="assurances">
            <li>Every test stays green, every assertion kept</li>
            <li>Your code stays on your machine</li>
            <li>Not faster? Full refund on any plan within 14 days</li>
          </ul>

          <p class="hero-proof">From the team behind TestProf, the open-source Rails test profiler used by GitLab and Discourse.</p>
        </div>

        <div class="terminal">
          <div class="terminal-bar">
            <span class="terminal-dots" aria-hidden="true"><span></span><span></span><span></span></span>
            <span class="terminal-title" aria-hidden="true">claude</span>
            <span class="terminal-timer" :class="{ done: clipStep >= clip.length }" aria-hidden="true">{{ timerText }}</span>
          </div>
          <div
            class="terminal-body"
            role="img"
            aria-label="An example run. The baseline 3:32, then 0:46, 4.6x faster, all green."
          >
            <p
              v-for="(line, i) in clip"
              :key="i"
              class="terminal-line"
              :class="[`line-${line.kind}`, { shown: clipStep > i }]"
            ><template v-if="line.kind === 'done'"><span class="check">✓ </span>Tests 4.6x faster, all green<span class="result"><s>3:32</s> 0:46</span></template><template v-else>{{ line.text }}</template></p>
          </div>
        </div>
      </div>
    </section>

    <!-- What you get -->
    <section class="band">
      <div class="inner">
        <h2>What you get</h2>
        <p class="section-lead">A private gem your agent works through. Install it once, ask your agent to make the tests faster, and review the branch it hands back.</p>
        <div class="grid-3 get-steps">
          <div class="card">
            <p class="get-step">1. Install</p>
            <pre class="get-code"><code>gem install test-pro \
  --source &lt;your private URL&gt;
test-pro install</code></pre>
            <p>Sets up Claude Code, Codex, Cursor or OpenCode: the TestPro MCP server and its skill. Or add the gem to your Gemfile for the whole team.</p>
          </div>
          <div class="card">
            <p class="get-step">2. Ask your agent</p>
            <pre class="get-code"><code>&gt; Our test suite is slow.
  Make it faster.</code></pre>
            <p>The agent checks your code against 40 known causes of slow Rails tests, runs TestProf's profilers and measures every fix before it keeps it. You set the time budget.</p>
          </div>
          <div class="card">
            <p class="get-step">3. Review the branch</p>
            <ul class="get-list">
              <li>One commit per verified fix</li>
              <li>A summary: before and after times, each fix, flaky tests found, what is left to win</li>
              <li>A decision log: why each fix was kept or declined</li>
              <li>A project skill, so your agents write fast specs from now on</li>
            </ul>
          </div>
        </div>
      </div>
    </section>

    <!-- Results -->
    <section id="results" class="band">
      <div class="inner">
        <h2>Measured, not promised</h2>
        <p class="section-lead">
          {{ benchResults.length }} test suites of real Rails apps, with and without TestPro:
          Mastodon, Discourse, Forem, Redmine, Lobsters, Consul and more.
        </p>

        <div class="card chart">
          <table class="dot-plot" role="table">
            <caption class="visually-hidden">Speed-up by app, TestProf only and TestProf plus TestPro</caption>
            <thead role="rowgroup">
              <tr role="row">
                <th scope="col" role="columnheader">App</th>
                <th scope="col" role="columnheader" class="num"><span class="key key-alone" aria-hidden="true"></span>TestProf</th>
                <th scope="col" role="columnheader" class="num"><span class="key key-pro" aria-hidden="true"></span>TestProf + <span class="pro-word">TestPro</span></th>
                <th class="axis" aria-hidden="true">
                  <span class="axis-ticks">
                    <span v-for="tick in ticks" :key="tick" :style="{ left: position(tick) }">{{ tick }}x</span>
                  </span>
                </th>
              </tr>
            </thead>
            <tbody role="rowgroup">
              <tr v-for="result in benchResults" :key="result.app" role="row">
                <th scope="row" role="rowheader">{{ result.app }}</th>
                <td class="num" role="cell">{{ times(result.alone) }}</td>
                <td class="num pro-value" role="cell">{{ times(result.pro) }}</td>
                <td class="track-cell" aria-hidden="true">
                  <span class="track">
                    <span
                      class="link"
                      :style="{
                        left: position(Math.min(result.alone, result.pro)),
                        width: position(Math.abs(result.pro - result.alone)),
                      }"
                    ></span>
                    <span class="dot dot-alone" :style="{ left: position(result.alone) }"></span>
                    <span class="dot dot-pro" :style="{ left: position(result.pro) }"></span>
                  </span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <dl class="stats">
          <div class="stat">
            <dt>Median speed-up</dt>
            <dd>
              <span class="stat-value">{{ times(medianOf(speedups.pro), 1) }}</span>
              <span class="stat-note">vs. TestProf only: {{ times(medianOf(speedups.alone), 1) }}</span>
            </dd>
          </div>
          <div class="stat">
            <dt>Apps where TestPro wins</dt>
            <dd>
              <span class="stat-value">{{ ahead.length }} of {{ benchResults.length }}</span>
              <span class="stat-note">up to {{ gain(best) }} faster</span>
            </dd>
          </div>
          <div class="stat">
            <dt>Agent time per run</dt>
            <dd>
              <span class="stat-value">{{ agentMinutes.pro }} min</span>
              <span class="stat-note">vs. TestProf only: {{ agentMinutes.alone }} min</span>
            </dd>
          </div>
        </dl>

        <details class="disclosure">
          <summary>How we measure</summary>
          <div class="disclosure-body">
            <p>
              Claude Code with Opus 5.5 (default effort), the same task with and without TestPro: make the app's tests faster. 3 runs per app, averaged.
              A run counts only if the suite stays green and every assertion is kept.
            </p>
          </div>
        </details>
      </div>
    </section>

    <!-- Calculator -->
    <section class="band">
      <div class="inner">
        <h2>What slow tests cost you</h2>
        <TestProCalculator :checkout-url="checkoutUrl" />
      </div>
    </section>

    <!-- Pricing -->
    <section id="pricing" class="band">
      <div class="inner">
        <h2>Pricing</h2>

        <p v-if="promoActive" class="promo"><strong>Cyber week:</strong> 50% off your first year until December 11.</p>

        <div class="plans plans-3">
          <div class="card plan">
            <h3>Starter</h3>
            <p class="plan-who">Up to 5 developers</p>
            <p class="price"><span class="price-value">{{ promoActive ? '$197.50' : '$395' }}</span>/year</p>
            <p v-if="promoActive" class="price-then">First year. Then $395/year.</p>
            <ul>
              <li>Unlimited repos and CI runs</li>
              <li>A project skill that keeps your agents from writing slow tests</li>
              <li>Catalog updates</li>
              <li>Email support</li>
            </ul>
            <a class="btn btn-alt" :href="checkoutUrl('starter')">Buy Starter</a>
          </div>
          <div class="card plan plan-featured">
            <h3>Team <span class="badge">Most teams</span></h3>
            <p class="plan-who">6 to 20 developers</p>
            <p class="price"><span class="price-value">{{ promoActive ? '$497.50' : '$995' }}</span>/year</p>
            <p v-if="promoActive" class="price-then">First year. Then $995/year.</p>
            <ul>
              <li>Everything in Starter, for a team of up to 20</li>
            </ul>
            <a class="btn btn-brand" :href="checkoutUrl('team')">Buy Team</a>
          </div>
          <div class="card plan">
            <h3>Business</h3>
            <p class="plan-who">Up to 100 developers</p>
            <p class="price"><span class="price-value">{{ promoActive ? '$2,450' : '$4,900' }}</span>/year</p>
            <p v-if="promoActive" class="price-then">First year. Then $4,900/year.</p>
            <ul>
              <li>Everything in Team</li>
              <li>A review of your first run with the TestProf maintainers</li>
              <li>Private catalog pages for your own slow-test causes</li>
              <li>Invoice and custom terms: <a class="text-link" href="https://cal.com/team/evilmartians/exploration?notes=We%27re%20interested%20in%20TestPro.%20" target="_blank" rel="noopener">book a call</a></li>
            </ul>
            <a class="btn btn-alt" :href="checkoutUrl('business')">Buy Business</a>
          </div>
        </div>

        <div class="sprint-band">
          <p><strong>Over 20 developers, or more than 100?</strong> A two-week speedup sprint for $12,000, with a year of TestPro for your whole team included. Large suites can add weeks at $6,000 each.</p>
          <a class="text-link" href="#sprint">How the sprint works</a>
        </div>

        <p class="plan-note">Billed yearly.<template v-if="promoActive"> Cyber week prices cover the first year; renewals are at the regular price.</template><br />Not faster? Write to us within 14 days for a full refund on any plan.</p>
      </div>
    </section>

    <!-- Speedup sprint -->
    <section id="sprint" class="band soft">
      <div class="inner">
        <h2>Speedup sprint for large suites</h2>
        <p class="section-lead">
          A big monolith's suite grows every month, and coding agents now write tests faster than anyone can optimize them.
          In a sprint, the TestProf team at Evil Martians cuts your build time with your engineers, then sets up your agents so the cut holds.
        </p>

        <div class="grid-3 sprint-weeks">
          <div class="card">
            <p class="sprint-week">Week 1</p>
            <h3>Profile and measure</h3>
            <p>We run TestProf on your CI and record the baseline: wall-clock per build, machine minutes, and the length of one agent test loop. Every fix is ranked by speedup per line of changed code.</p>
            <p class="sprint-price">Can be booked alone: $6,000.</p>
          </div>
          <div class="card">
            <p class="sprint-week">Week 2</p>
            <h3>Fixes and guardrails</h3>
            <p>Rails boot, environment load, database setup and factory cascades: the fixes that speed up every test at once, and every agent loop with them. Then skills and CI hooks for your codebase, so slow tests stay out.</p>
          </div>
          <div class="card">
            <p class="sprint-week">Extra weeks</p>
            <h3>Patterns across a large suite</h3>
            <p>For large or complex suites, we spread the winning patterns, like let_it_be, file by file with AI agents under our review.</p>
            <p class="sprint-price">$6,000 a week, as many as your suite needs.</p>
          </div>
        </div>

        <div class="grid-3 sprint-facts">
          <div>
            <h3>Measured on your own builds</h3>
            <p>Before and after numbers from your CI. Every spec stays green and every assertion is kept. If week 1 finds little time to win, we tell you and you stop there.</p>
          </div>
          <div>
            <h3>Works alongside your test splitter</h3>
            <p>Knapsack Pro and Buildkite Test Engine share the suite across machines; we make what they share smaller. Flaky tests go through the same playbook.</p>
          </div>
          <div>
            <h3>The saving shows up as fewer CI machines</h3>
            <p>A faster build lets you retire machines, or keep the fleet and absorb a year or two of test growth without buying more.</p>
          </div>
        </div>

        <dl class="stats sprint-stats">
          <div class="stat">
            <dt>Power Home Remodeling</dt>
            <dd><span class="stat-value">53 → 11 min</span><span class="stat-note">core suite of a large modular monolith, two sprints</span></dd>
          </div>
          <div class="stat">
            <dt>Whop</dt>
            <dd><span class="stat-value">2x</span><span class="stat-note">faster test suite and CI</span></dd>
          </div>
        </dl>

        <p class="sprint-proof">
          Read the <a class="text-link" href="https://evilmartians.com/clients/power-home-remodeling" target="_blank" rel="noopener">Power Home Remodeling</a> and <a class="text-link" href="https://evilmartians.com/chronicles/the-whop-chop-how-we-cut-a-rails-test-suite-and-ci-time-in-half" target="_blank" rel="noopener">Whop</a> stories.
        </p>

        <div class="sprint-cta">
          <p>Two weeks, $12,000. Extra weeks at $6,000 each, depending on the size and complexity of your suite. Includes 12 months of TestPro for your whole engineering team, with no developer cap. After that, TestPro renews at $4,900 a year for up to 100 developers or $9,900 with no cap.</p>
          <a class="btn btn-brand" href="https://cal.com/team/evilmartians/exploration?notes=We%27re%20interested%20in%20a%20TestPro%20speedup%20sprint.%20" target="_blank" rel="noopener">Book a call about a sprint</a>
        </div>
      </div>
    </section>

    <!-- FAQ -->
    <section class="band">
      <div class="inner narrow">
        <h2>Questions</h2>
        <details v-for="item in faq" :key="item.q" class="disclosure faq">
          <summary>{{ item.q }}</summary>
          <div class="disclosure-body"><p>{{ item.a }}</p></div>
        </details>
        <p class="final-actions">
          <a class="btn btn-brand" href="#pricing" @click.prevent="goToPricing">See plans</a>
          <a class="text-link" href="https://cal.com/team/evilmartians/exploration?notes=We%27re%20interested%20in%20TestPro.%20" target="_blank" rel="noopener">Book a call about a sprint</a>
        </p>
      </div>
    </section>

    <!-- Footer -->
    <footer class="footer">
      <p>
        <a href="https://github.com/test-prof/test-prof" target="_blank" rel="noopener">GitHub</a>
        <span class="sep">&middot;</span>
        <a href="/guide/getting_started">Documentation</a>
        <span class="sep">&middot;</span>
        <a href="https://evilmartians.com/?utm_source=test-prof" target="_blank" rel="noopener">Evil Martians</a>
        <span class="sep">&middot;</span>
        <a href="https://evilmartians.com/privacy" target="_blank" rel="noopener">Privacy policy</a>
      </p>
      <p class="footer-badge">
        <a href="https://evilmartians.com/?utm_source=test-prof" target="_blank" rel="noopener">
          <img
            src="https://evilmartians.com/badges/sponsored-by-evil-martians.svg"
            alt="Sponsored by Evil Martians"
            width="236"
            height="54"
          />
        </a>
      </p>
    </footer>
  </div>
</template>

<style scoped>
.pro {
  font-family: var(--vp-font-family-base);
  color: var(--vp-c-text-1);
}

.pro code {
  font-family: var(--vp-font-family-mono);
  font-size: 0.85em;
  padding: 1px 5px;
  border-radius: 4px;
  background-color: var(--vp-c-brand-soft);
  color: var(--vp-c-text-1);
}

/* Hero */
.hero {
  --fall: 0.6s;
  --stamp: 1.35s;
  padding: 64px 24px;
  overflow-x: clip;
}

.hero-inner {
  display: grid;
  grid-template-columns: 1.1fr 1fr;
  gap: 48px;
  align-items: center;
  max-width: 1152px;
  margin: 0 auto;
}

/* Title */
.title {
  position: relative;
  display: inline-flex;
  align-items: baseline;
  font-size: 4rem;
  font-weight: 700;
  line-height: 1;
  letter-spacing: -0.01em;
}

.f {
  display: inline-block;
  pointer-events: none;
  max-width: 1ch;
  transform-origin: 20% 90%;
}

.playing .f {
  animation:
    fall 0.7s cubic-bezier(0.5, 0, 0.9, 0.6) var(--fall) forwards,
    collapse 0.4s ease-in-out calc(var(--fall) + 0.35s) forwards;
}

@keyframes fall {
  25% { transform: rotate(35deg); }
  100% { opacity: 0; transform: translate(16px, 110px) rotate(120deg); }
}

@keyframes collapse {
  to { max-width: 0; }
}

/* Early access stamp */
.stamp {
  position: absolute;
  left: calc(100% + 14px);
  top: 22%;
  display: inline-flex;
  padding: 6px 10px;
  border-radius: 6px;
  background-color: var(--vp-c-danger-2);
  color: var(--vp-c-white);
  font-family: var(--vp-font-family-mono);
  font-size: 1rem;
  font-weight: 500;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  white-space: nowrap;
  opacity: 0;
  transform: translateY(-50%) rotate(-7deg);
  box-shadow: 3px 3px 0 rgba(0, 0, 0, 0.18);
}

.playing .stamp {
  animation: stamp 0.55s cubic-bezier(0.2, 1.4, 0.4, 1) var(--stamp) forwards;
}

@keyframes stamp {
  0% { opacity: 0; transform: translateY(-50%) scale(2.4) rotate(-22deg); }
  60% { opacity: 1; transform: translateY(-50%) scale(0.92) rotate(-4deg); }
  100% { opacity: 1; transform: translateY(-50%) scale(1) rotate(-7deg); }
}

.hero h1 {
  margin: 24px 0 0;
  font-size: 2.25rem;
  font-weight: 700;
  line-height: 1.2;
  letter-spacing: -0.01em;
  text-wrap: balance;
}

.hero-lead {
  margin: 20px 0 0;
  font-size: 1.15rem;
  line-height: 1.6;
  color: var(--vp-c-text-2);
}

.hero-lead span {
  display: block;
}

.hero-actions {
  display: inline-grid;
  grid-template-columns: auto auto;
  align-items: center;
  gap: 8px 12px;
  margin-top: 24px;
}

.btn {
  display: inline-block;
  padding: 11.5px 24px 8.5px;
  border: 1px solid transparent;
  border-radius: 8px;
  font-size: 0.95rem;
  font-weight: 600;
  line-height: 1.5;
  text-align: center;
  text-decoration: none;
  white-space: nowrap;
  transition: background-color 0.2s, border-color 0.2s, color 0.2s;
}

.btn-brand {
  background-color: var(--vp-c-brand-3);
  color: var(--vp-c-white);
}

.btn-brand:hover {
  background-color: var(--vp-c-brand-1);
}

.btn-alt {
  border-color: var(--vp-c-brand-3);
  color: var(--vp-c-brand-3);
}

.btn-alt:hover {
  background-color: var(--vp-c-brand-soft);
}

.btn:disabled {
  opacity: 0.6;
  cursor: default;
}

.hero-copy {
  min-width: 0;
}

.hero-note {
  margin: 10px 0 0;
  font-size: 0.85rem;
  color: var(--vp-c-text-2);
}

/* Terminal */
.terminal {
  min-width: 0;
  border: 1px solid #2a2430;
  border-radius: 12px;
  background-color: #1e1a22;
  color: rgba(255, 255, 255, 0.87);
  box-shadow: 0 12px 32px rgba(54, 38, 64, 0.22);
  overflow: hidden;
}

.terminal-bar {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px 16px;
  border-bottom: 1px solid #2a2430;
}

.terminal-dots {
  display: inline-flex;
  gap: 6px;
}

.terminal-dots span {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background-color: #4a4252;
}

.terminal-title {
  font-family: var(--vp-font-family-mono);
  font-size: 0.75rem;
  color: rgba(235, 235, 245, 0.6);
}

.terminal-body {
  padding: 16px 16px 24px;
  font-family: var(--vp-font-family-mono);
  font-size: 0.85rem;
  line-height: 1.6;
}

.terminal-line {
  margin: 0;
  padding-left: 2ch;
  text-indent: -2ch;
  visibility: hidden;
}

.terminal-line + .terminal-line {
  margin-top: 6px;
}

.terminal-line.shown {
  visibility: visible;
  animation: line-in 0.3s ease-out;
}

@keyframes line-in {
  from { opacity: 0; transform: translateY(4px); }
}

.line-prompt {
  color: #c4a8d4;
  margin-bottom: 16px !important;
}

.line-step {
  color: rgba(235, 235, 245, 0.7);
}

.line-done {
  margin-top: 20px !important;
  color: #fff;
  font-weight: 500;
}

.check {
  color: #3dd68c;
  font-weight: 700;
}

/* Bands */
.band {
  padding: 64px 24px;
}

.hero + .band,
.band + .band:not(.soft) {
  border-top: 1px solid var(--vp-c-divider);
}

.band.soft {
  background-color: var(--vp-c-bg-soft);
}

.inner {
  max-width: 960px;
  margin: 0 auto;
}

.inner.narrow {
  max-width: 720px;
}

.band h2 {
  margin: 0 0 12px;
  font-size: 1.75rem;
  font-weight: 700;
  line-height: 1.3;
  text-align: center;
  text-wrap: balance;
}

.section-lead {
  max-width: 640px;
  margin: 0 auto;
  text-align: center;
  line-height: 1.6;
  color: var(--vp-c-text-2);
}

.band h2 + :not(.section-lead) {
  margin-top: 32px;
}

.section-lead + * {
  margin-top: 32px;
}

.card {
  background-color: var(--vp-c-bg);
  border: 1px solid var(--vp-c-divider);
  border-radius: 12px;
  padding: 24px;
}

.card h3 {
  margin: 0;
  font-size: 1.05rem;
  font-weight: 700;
  line-height: 1.4;
  color: var(--vp-c-brand-3);
}

.card p {
  margin: 10px 0 0;
  font-size: 0.9rem;
  line-height: 1.5;
  color: var(--vp-c-text-2);
}

.grid-3 {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
}

.part h3 {
  display: flex;
  align-items: center;
  gap: 10px;
}

.part p {
  margin-top: 4px;
}

.part-kind {
  display: inline-block;
  padding: 2px 8px;
  border: 1px solid transparent;
  border-radius: 999px;
  font-size: 0.7rem;
  font-weight: 700;
  letter-spacing: 0.05em;
  text-transform: uppercase;
}

.kind-direction {
  background-color: var(--vp-c-brand-3);
  color: var(--vp-c-white);
}

.kind-context {
  background-color: var(--vp-c-brand-soft);
  color: var(--vp-c-brand-3);
}

.kind-verification {
  border-color: var(--vp-c-brand-3);
  color: var(--vp-c-brand-3);
}

/* Disclosures */
.disclosure {
  margin-top: 24px;
  border: 1px solid var(--vp-c-divider);
  border-radius: 12px;
  background-color: var(--vp-c-bg);
}

.stats + .disclosure {
  margin-top: 32px;
}

.disclosure + .disclosure {
  margin-top: 12px;
}

.disclosure summary {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 16px 24px;
  font-weight: 600;
  cursor: pointer;
  list-style: none;
}

.disclosure summary::-webkit-details-marker {
  display: none;
}

.disclosure summary::before {
  content: "";
  width: 7px;
  height: 7px;
  border-right: 2px solid var(--vp-c-brand-3);
  border-bottom: 2px solid var(--vp-c-brand-3);
  transform: rotate(-45deg);
  transition: transform 0.2s;
  flex-shrink: 0;
}

.disclosure[open] summary::before {
  transform: rotate(45deg);
}

.disclosure summary:hover {
  color: var(--vp-c-brand-3);
}

.disclosure-body {
  padding: 0 24px 20px;
  font-size: 0.9rem;
  line-height: 1.6;
  color: var(--vp-c-text-2);
}

.disclosure-body p {
  margin: 0;
}

/* Dot plot */
.chart {
  padding: 16px 24px 24px;
  overflow-x: auto;
}

.dot-plot {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.9rem;
}

.dot-plot th,
.dot-plot td {
  padding: 8px 12px 8px 0;
  border-bottom: 1px solid var(--vp-c-divider);
  text-align: left;
  vertical-align: middle;
}

.dot-plot tbody tr:last-child > * {
  border-bottom: none;
}

.dot-plot thead th {
  padding-bottom: 12px;
  font-size: 0.8rem;
  line-height: 20px;
  vertical-align: bottom;
  font-weight: 600;
  color: var(--vp-c-text-2);
  white-space: nowrap;
}

.dot-plot tbody th {
  font-weight: 400;
}

.dot-plot .num {
  padding-left: 12px;
  font-family: var(--vp-font-family-mono);
  text-align: center;
  white-space: nowrap;
}

.dot-plot thead .num {
  font-family: var(--vp-font-family-base);
}

.dot-plot td.num {
  color: var(--vp-c-text-2);
}

.dot-plot td.pro-value {
  font-weight: 500;
  color: var(--vp-c-text-1);
}

.key {
  display: inline-block;
  width: 10px;
  height: 10px;
  margin-right: 6px;
  border-radius: 50%;
  vertical-align: -1px;
}

.pro-word {
  color: var(--vp-c-brand-1);
}

.key-alone,
.dot-alone {
  border: 2px solid var(--vp-c-text-2);
  background-color: var(--vp-c-bg);
}

.key-pro,
.dot-pro {
  background-color: var(--vp-c-brand-3);
}

.axis,
.track-cell {
  width: 45%;
  padding-left: 32px !important;
  padding-right: 12px !important;
}

.axis-ticks {
  position: relative;
  display: block;
  height: 20px;
}

.axis-ticks span {
  position: absolute;
  top: 0;
  transform: translateX(-50%);
}

.track {
  position: relative;
  display: block;
  height: 14px;
  background-image:
    linear-gradient(to right, var(--vp-c-divider) 1px, transparent 1px),
    linear-gradient(var(--vp-c-divider), var(--vp-c-divider));
  background-size: 20% 100%, 1px 100%;
  background-position: 0 0, 100% 0;
  background-repeat: repeat-x, no-repeat;
}

.link {
  position: absolute;
  top: 50%;
  height: 2px;
  margin-top: -1px;
  background-color: var(--vp-c-brand-1);
  opacity: 0.6;
}

.dot {
  position: absolute;
  top: 50%;
  width: 12px;
  height: 12px;
  border-radius: 50%;
  transform: translate(-50%, -50%);
}

.dot-pro {
  box-shadow: 0 0 0 2px var(--vp-c-bg);
}

/* Stats */
.stats {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
  margin: 32px 0 0;
}

.stat {
  padding: 4px 0;
  text-align: center;
}

.stat dt {
  font-size: 0.9rem;
  line-height: 1.4;
  font-weight: 600;
  color: var(--vp-c-text-2);
}

.stat dd {
  margin: 4px 0 0;
}

.stat-value {
  display: block;
  font-size: 1.6rem;
  font-weight: 700;
  line-height: 1.3;
  color: var(--vp-c-brand-3);
}

.stat-note {
  display: block;
  font-size: 0.85rem;
  line-height: 1.4;
  color: var(--vp-c-text-2);
}

/* Pricing */
.plan {
  display: flex;
  flex-direction: column;
}

.plan h3 {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 0.8rem;
  letter-spacing: 0.05em;
  text-transform: uppercase;
}

.plan-featured {
  border-color: var(--vp-c-brand-3);
  box-shadow: 0 0 0 1px var(--vp-c-brand-3);
}

.badge {
  display: inline-block;
  position: relative;
  top: -2px;
  margin: -4px 0;
  padding: 2px 8px;
  line-height: 1.4;
  border-radius: 999px;
  background-color: var(--vp-c-brand-soft);
  color: var(--vp-c-brand-3);
  font-size: 0.7rem;
  font-weight: 700;
  letter-spacing: 0.05em;
}

.card .price {
  margin-top: 12px;
  font-size: 1rem;
  color: var(--vp-c-text-2);
}

.price-value {
  font-size: 2rem;
  font-weight: 700;
  line-height: 1.2;
  color: var(--vp-c-text-1);
}

.plan ul {
  margin: 16px 0 24px;
  padding: 0;
  list-style: none;
  font-size: 0.9rem;
  line-height: 1.5;
  color: var(--vp-c-text-2);
}

.plan li {
  position: relative;
  padding-left: 20px;
}

.plan li + li {
  margin-top: 8px;
}

.plan li::before {
  content: "";
  position: absolute;
  left: 2px;
  top: 0.3em;
  width: 5px;
  height: 10px;
  border-right: 2px solid var(--vp-c-brand-3);
  border-bottom: 2px solid var(--vp-c-brand-3);
  transform: rotate(45deg);
}

.plan .btn {
  margin-top: auto;
}

.plan-note {
  margin: 24px 0 0;
  text-align: center;
  line-height: 1.6;
  color: var(--vp-c-text-2);
}

.text-link {
  color: var(--vp-c-brand-3);
  font-weight: 600;
  text-decoration: underline;
  text-underline-offset: 3px;
}

.text-link:hover {
  color: var(--vp-c-brand-1);
}

.final-actions {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
  gap: 24px;
  margin: 32px 0 0;
}

/* Footer */
.footer {
  padding: 32px 24px;
  border-top: 1px solid var(--vp-c-divider);
  text-align: center;
}

.footer p {
  margin: 0;
  font-size: 0.9rem;
  color: var(--vp-c-text-2);
}

.footer a {
  color: var(--vp-c-brand-3);
  text-decoration: none;
}

.footer a:hover {
  text-decoration: underline;
}

.footer-badge {
  margin-top: 16px !important;
}

.footer-badge a {
  display: inline-block;
}

.sep {
  margin: 0 8px;
  color: var(--vp-c-text-3);
}

/* Responsive */
@media (min-width: 961px) and (max-width: 1200px) {
  .terminal-body {
    font-size: 0.75rem;
  }
}

@media (min-width: 961px) and (max-width: 1100px) {
  .terminal-body {
    font-size: 0.7rem;
  }
}

@media (max-width: 960px) {
  .hero-inner {
    grid-template-columns: 1fr;
  }

  .grid-3 {
    grid-template-columns: 1fr;
  }
}

@media (min-width: 640px) and (max-width: 960px) {
  .grid-3 {
    grid-template-columns: repeat(2, 1fr);
  }

  .plans {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 768px) {
  .hero {
    padding: 48px 16px 40px;
  }

  .title {
    font-size: 2.8rem;
  }

  .stamp {
    font-size: 0.8rem;
    letter-spacing: 0.06em;
  }

  .hero h1 {
    font-size: 1.75rem;
  }

  .hero-lead {
    font-size: 1.05rem;
  }

  .band {
    padding: 48px 16px;
  }

  .band h2 {
    font-size: 1.4rem;
  }

  .chart {
    padding: 12px;
  }

  .stats {
    grid-template-columns: 1fr;
    gap: 12px;
  }
}

@media (max-width: 640px) {
  .hero-lead span {
    display: inline;
  }

  .hero-lead span + span::before {
    content: " ";
  }

  .hero-actions {
    display: grid;
    grid-template-columns: 1fr;
  }

  .terminal-body {
    font-size: 0.72rem;
  }

  .dot-plot,
  .dot-plot thead,
  .dot-plot tbody {
    display: block;
  }

  .dot-plot tr {
    display: grid;
    grid-template-columns: 1fr auto auto;
    align-items: center;
    column-gap: 12px;
    padding: 8px 0;
    border-bottom: 1px solid var(--vp-c-divider);
  }

  .dot-plot tbody tr:last-child {
    border-bottom: none;
  }

  .dot-plot th,
  .dot-plot td {
    padding: 0;
    border-bottom: none;
  }

  .dot-plot .axis,
  .dot-plot .track-cell {
    display: none;
  }
}

@media (max-width: 400px) {
  .title {
    font-size: 2.4rem;
  }

  .stamp {
    left: calc(100% + 10px);
    font-size: 0.7rem;
  }
}

@media (prefers-reduced-motion: reduce) {
  .pro * {
    animation-duration: 1ms !important;
    animation-delay: 0s !important;
    animation-iteration-count: 1 !important;
    transition: none !important;
  }
}

/* Four plans */
.plans-4 {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
}

.plan-who {
  margin: 4px 0 0;
  font-size: 0.9rem;
  color: var(--vp-c-text-2);
}

.plan .price s {
  margin-left: 4px;
  color: var(--vp-c-text-3);
}

.promo {
  margin: 0 0 24px;
  padding: 10px 16px;
  border-radius: 12px;
  background-color: var(--vp-c-brand-soft);
  color: var(--vp-c-brand-3);
  text-align: center;
}

@media (max-width: 1100px) {
  .plans-4 {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 640px) {
  .plans-4 {
    grid-template-columns: 1fr;
  }
}

/* Speedup sprint */
.sprint-week {
  margin: 0 0 6px !important;
  font-size: 0.75rem !important;
  font-weight: 700;
  letter-spacing: 0.05em;
  text-transform: uppercase;
  color: var(--vp-c-text-3) !important;
}

.card .sprint-price {
  margin-top: 14px;
  font-weight: 700;
  color: var(--vp-c-brand-3);
}

.sprint-facts {
  margin-top: 40px;
}

.sprint-facts h3 {
  margin: 0;
  font-size: 1rem;
  font-weight: 700;
  line-height: 1.4;
}

.sprint-facts p {
  margin: 8px 0 0;
  font-size: 0.9rem;
  line-height: 1.5;
  color: var(--vp-c-text-2);
}

.sprint-proof {
  margin: 40px auto 0;
  max-width: 720px;
  text-align: center;
  line-height: 1.6;
  color: var(--vp-c-text-2);
}

.sprint-cta {
  margin-top: 24px;
  text-align: center;
}

.sprint-cta p {
  margin: 0 0 16px;
  font-weight: 600;
}

@media (max-width: 960px) {
  .sprint-weeks,
  .sprint-facts {
    grid-template-columns: 1fr;
  }
}

/* What you get */
.get-steps .card p {
  margin-top: 12px;
}

.get-step {
  margin: 0 !important;
  font-weight: 700;
  color: var(--vp-c-brand-3) !important;
}

.get-code {
  margin: 12px 0 0;
  padding: 12px 14px;
  overflow-x: auto;
  border-radius: 8px;
  background-color: var(--vp-c-bg-alt);
  font-family: var(--vp-font-family-mono);
  font-size: 0.8rem;
  line-height: 1.6;
}

.get-list {
  margin: 12px 0 0;
  padding-left: 18px;
  font-size: 0.9rem;
  line-height: 1.5;
  color: var(--vp-c-text-2);
}

.get-list li + li {
  margin-top: 6px;
}

@media (max-width: 960px) {
  .get-steps {
    grid-template-columns: 1fr;
  }
}

.plan-actions {
  display: grid;
  gap: 10px;
  margin-top: auto;
  text-align: center;
}

.plan-actions .btn {
  margin-top: 0;
}

.plan-actions .text-link {
  font-size: 0.9rem;
}

/* Review fixes, 27 Nov */
.title {
  font-size: 2.25rem;
}

.hero h1 {
  font-size: 2.4rem;
}

.btn-brand {
  background-color: #5f3f73;
  color: #fff;
}

.btn-brand:hover {
  background-color: #4e3360;
}

.dark .btn-brand {
  background-color: #c4a8d4;
  color: #1b1520;
}

.dark .btn-brand:hover {
  background-color: #d6c0e2;
}

.hero-note {
  margin: 0;
  font-size: 0.9rem;
  color: var(--vp-c-text-2);
}

.assurances {
  display: flex;
  flex-wrap: wrap;
  gap: 8px 20px;
  margin: 20px 0 0;
  padding: 0;
  list-style: none;
  font-size: 0.9rem;
  color: var(--vp-c-text-1);
}

.assurances li {
  position: relative;
  padding-left: 22px;
}

.assurances li::before {
  content: "✓";
  position: absolute;
  left: 0;
  font-weight: 700;
  color: #2fb67c;
}

.hero-proof {
  margin: 20px 0 0;
  font-size: 0.85rem;
  color: var(--vp-c-text-2);
}

.terminal {
  border-color: #3a3242;
}

.terminal-timer {
  margin-left: auto;
  font-family: var(--vp-font-family-mono);
  font-size: 0.85rem;
  font-variant-numeric: tabular-nums;
  color: rgba(235, 235, 245, 0.7);
}

.terminal-timer.done {
  color: #3dd68c;
  font-weight: 700;
}

.line-done .result {
  display: block;
  margin-top: 8px;
  padding-left: 2ch;
  font-size: 2.2rem;
  font-weight: 700;
  line-height: 1.1;
  color: #fff;
}

.line-done .result s {
  margin-right: 12px;
  font-size: 1.1rem;
  font-weight: 400;
  color: rgba(235, 235, 245, 0.45);
}

.get-code code {
  padding: 0;
  border-radius: 0;
  background: none;
  font-size: inherit;
}

.get-list {
  padding-left: 0;
  list-style: none;
}

.plans-3 {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
}

.price-then {
  margin: 2px 0 0 !important;
  font-size: 0.85rem !important;
  color: var(--vp-c-text-3) !important;
}

.sprint-band {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;
  gap: 12px 24px;
  margin-top: 20px;
  padding: 18px 24px;
  border: 1px solid var(--vp-c-divider);
  border-radius: 12px;
  background-color: var(--vp-c-bg-soft);
}

.sprint-band p {
  margin: 0;
  max-width: 680px;
  line-height: 1.5;
  color: var(--vp-c-text-2);
}

.sprint-band strong {
  color: var(--vp-c-text-1);
}

.sprint-stats {
  grid-template-columns: repeat(2, 1fr);
  max-width: 640px;
  margin: 40px auto 0;
}

.faq.disclosure {
  margin-top: 0;
  border: 0;
  border-bottom: 1px solid var(--vp-c-divider);
  border-radius: 0;
  background: none;
}

.faq.disclosure:first-of-type {
  border-top: 1px solid var(--vp-c-divider);
}

.faq.disclosure summary {
  padding: 16px 4px;
}

.faq.disclosure .disclosure-body {
  padding: 0 4px 18px 21px;
}

@media (max-width: 960px) {
  .plans-3 {
    grid-template-columns: 1fr;
  }

  .plans-3 .plan-featured {
    order: -1;
  }

  .hero h1 {
    font-size: 1.9rem;
  }
}
</style>
