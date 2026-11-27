<script setup lang="ts">
import { computed, ref } from 'vue'

const WORKING_DAYS_PER_YEAR = 21 * 12
const WEEKS_PER_YEAR = 52
const SPEEDUP = 1.5
const WAITING_SHARE = 1 / 3
const RUNS_PER_PERSON = 10
const RUNNER_PRICE = 0.012
const RUNNERS = 4
const TESTPRO_PER_YEAR = 995
const WORK_HOURS_PER_YEAR = WORKING_DAYS_PER_YEAR * 8

const developers = ref(20)
const minutesPerRun = ref(20)
const salary = ref(0)

const dollars = new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD', maximumFractionDigits: 0 })
const count = new Intl.NumberFormat('en-US', { maximumFractionDigits: 0 })
const plural = (n: number, word: string) => `${count.format(n)} ${word}${n === 1 ? '' : 's'}`

const chipGroups = [
  {
    name: 'calc-team',
    legend: 'Team size',
    options: [
      { value: 1, label: 'Just me' },
      { value: 5, label: 'Small' },
      { value: 20, label: 'Medium' },
      { value: 50, label: 'Big' },
    ],
    model: developers,
  },
  {
    name: 'calc-run',
    legend: 'CI run time',
    options: [10, 20, 40].map((v) => ({ value: v, label: `${v} min` })),
    model: minutesPerRun,
  },
  {
    name: 'calc-salary',
    legend: 'Developer salary',
    options: [
      { value: 0, label: 'Skip' },
      { value: 100000, label: '$100k' },
    ],
    model: salary,
  },
]

const savedShare = 1 - 1 / SPEEDUP
const runsPerYear = computed(() => developers.value * RUNS_PER_PERSON * WORKING_DAYS_PER_YEAR)
const ciSaved = computed(() => runsPerYear.value * minutesPerRun.value * RUNNERS * RUNNER_PRICE * savedShare)
const waitingHours = computed(() => (runsPerYear.value * minutesPerRun.value * savedShare * WAITING_SHARE) / 60)
const waitingWorth = computed(() => (waitingHours.value * salary.value) / WORK_HOURS_PER_YEAR)
const saved = computed(() => ciSaved.value + waitingWorth.value)
const paybackWeeks = computed(() =>
  saved.value >= TESTPRO_PER_YEAR ? Math.ceil((WEEKS_PER_YEAR * TESTPRO_PER_YEAR) / saved.value) : null,
)
</script>

<template>
  <div class="calc">
    <div class="calc-inputs">
      <fieldset v-for="group in chipGroups" :key="group.name" class="calc-group">
        <legend>{{ group.legend }}</legend>
        <div class="calc-chips">
          <label v-for="option in group.options" :key="String(option.value)" class="calc-chip">
            <input
              v-model="group.model.value"
              class="visually-hidden"
              type="radio"
              :name="group.name"
              :value="option.value"
            />
            <span>{{ option.label }}</span>
          </label>
        </div>
      </fieldset>
    </div>

    <div class="calc-results" aria-live="polite" aria-atomic="true">
      <p v-if="paybackWeeks !== null" class="calc-headline">
        TestPro pays for itself in about <strong>{{ plural(paybackWeeks, 'week') }}</strong>
      </p>
      <p v-else class="calc-headline">
        TestPro saves you <strong>{{ plural(waitingHours, 'hour') }}</strong> of waiting a year
      </p>

      <dl class="calc-numbers">
        <div>
          <dt>CI minutes saved</dt>
          <dd>{{ dollars.format(ciSaved) }} a year</dd>
        </div>
        <div>
          <dt>Developer time saved</dt>
          <dd>{{ plural(waitingHours, 'hour') }} a year</dd>
          <dd v-if="salary" class="calc-worth">worth {{ dollars.format(waitingWorth) }}</dd>
        </div>
      </dl>

      <p class="calc-assumptions">
        Assumes {{ plural(developers, 'developer') }} pushing {{ RUNS_PER_PERSON }} times a day, a {{ SPEEDUP }}x faster CI run
        on {{ RUNNERS }} GitHub runners at ${{ RUNNER_PRICE }} a minute, developers waiting for a third of each run<template v-if="salary"> at {{ dollars.format(salary) }} a year for 8-hour days</template>,
        and TestPro at {{ dollars.format(TESTPRO_PER_YEAR) }} a year.
      </p>
    </div>
  </div>
</template>

<style scoped>
.calc {
  display: grid;
  grid-template-columns: auto 1fr;
  gap: 24px;
}

.calc-inputs,
.calc-results {
  min-width: 0;
  padding: 24px;
  border: 1px solid var(--vp-c-divider);
  border-radius: 12px;
  background-color: var(--vp-c-bg);
}

.calc-group {
  margin: 0;
  padding: 0;
  border: 0;
  min-width: 0;
}

.calc-group + .calc-group {
  margin-top: 18px;
}

.calc-group legend {
  padding: 0;
  font-size: 0.9rem;
  color: var(--vp-c-text-2);
}

.calc-chips {
  display: grid;
  grid-auto-flow: column;
  grid-auto-columns: 1fr;
  width: max-content;
  gap: 4px;
  margin-top: 8px;
}

.calc-chip span {
  display: block;
  text-align: center;
  padding: 6px 8px 4px;
  border: 1px solid var(--vp-c-divider);
  border-radius: 999px;
  font-size: 0.85rem;
  line-height: 1.5;
  color: var(--vp-c-text-1);
  white-space: nowrap;
  cursor: pointer;
  transition: background-color 0.2s, border-color 0.2s, color 0.2s;
}

.calc-chip:hover span {
  border-color: var(--vp-c-brand-3);
}

.calc-chip input:checked + span {
  border-color: var(--vp-c-brand-3);
  background-color: var(--vp-c-brand-soft);
  color: var(--vp-c-brand-3);
  text-shadow: 0 0 0.6px currentColor;
}

.calc-chip input:focus-visible + span {
  outline: 2px solid var(--vp-c-brand-1);
  outline-offset: 2px;
}

.calc-headline {
  margin: 0;
  font-size: 1.5rem;
  font-weight: 700;
  line-height: 1.3;
  text-wrap: balance;
}

.calc-headline strong {
  color: var(--vp-c-brand-3);
}

.calc-numbers {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
  margin: 24px 0 0;
  padding-top: 24px;
  border-top: 1px solid var(--vp-c-divider);
}

.calc-numbers dt {
  font-size: 0.85rem;
  color: var(--vp-c-text-2);
}

.calc-numbers dd {
  margin: 4px 0 0;
  font-size: 1.15rem;
  font-weight: 700;
  font-variant-numeric: tabular-nums;
  color: var(--vp-c-text-1);
}

.calc-numbers .calc-worth {
  margin-top: 0;
  font-size: 0.85rem;
  font-weight: 400;
  color: var(--vp-c-text-2);
}

.calc-assumptions {
  margin: 24px 0 0;
  font-size: 0.8rem;
  line-height: 1.6;
  color: var(--vp-c-text-2);
}

@media (max-width: 768px) {
  .calc {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 640px) {
  .calc-numbers {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 400px) {
  .calc-inputs,
  .calc-results {
    padding: 16px;
  }

  .calc-chips {
    width: 100%;
  }
}

@media (prefers-reduced-motion: reduce) {
  .calc * {
    transition: none !important;
  }
}
</style>
