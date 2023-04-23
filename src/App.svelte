<script lang="ts">
  import { derived, writable } from "svelte/store";

  const purchasePrice = writable(325000);
  const downPayment = writable(0.2);
  const principal = derived(
    [purchasePrice, downPayment],
    ([purchasePrice, downPayment]) =>
      downPayment > 1
        ? purchasePrice - downPayment
        : purchasePrice * downPayment
  );
  const annualInterestRate = writable(0.065);
  const monthlyInterestRate = derived(
    [annualInterestRate],
    ([value]) => value / 12
  );
  const monthlyPayment = writable(2000);
  const amortization = derived(
    [principal, monthlyInterestRate, monthlyPayment],
    ([principal, monthlyInterestRate, monthlyPayment]) => {
      monthlyPayment = Math.max(
        principal * monthlyInterestRate + 1,
        monthlyPayment
      );

      let schedule: [number, number, number, number][] = [];
      let interestTotal = 0;
      let principalTotal = 0;
      while (principal > 0) {
        const interestPayment = principal * monthlyInterestRate;
        const principalPayment = Math.min(
          principal,
          monthlyPayment - interestPayment
        );
        principal -= principalPayment;
        interestTotal += interestPayment;
        principalTotal += principalPayment;
        schedule.push([
          interestPayment,
          interestTotal,
          principalPayment,
          principalTotal,
        ]);
      }

      return schedule;
    }
  );
  const totals = derived([amortization], ([amortization]) => {
    const [, interest] = amortization[amortization.length - 1] ?? [
      undefined,
      0,
    ];
    return {
      months: amortization.length + 1,
      years: Math.ceil((amortization.length + 1) / 12),
      interest,
    };
  });

  function formatNumber(number: number) {
    const [value, fraction] = number
      .toFixed(2)
      .split(".")
      .map((n) => parseInt(n));
    return `${value.toLocaleString()}.${fraction}`;
  }
</script>

<main>
  <form on:submit|preventDefault>
    <label>
      Purchase Price <input
        bind:value={$purchasePrice}
        class="dollar"
        min="0"
        step="0.01"
        type="number"
      />
    </label>
    <label>
      Down Payment <input
        bind:value={$downPayment}
        class="percent"
        type="number"
      />
    </label>
    <label>
      Monthly Payment <input
        bind:value={$monthlyPayment}
        class="dollar"
        min={$monthlyInterestRate * $principal}
        step="50"
        type="number"
      />
    </label>
    <label>
      Annual Interest Rate <input
        bind:value={$annualInterestRate}
        class="percent"
        min="0"
        step="0.01"
        type="number"
      />
    </label>
    <fieldset>
      <legend>Summary</legend>
      <label>
        Down Payment
        {#if $downPayment < 1}
          $
          <output>
            {formatNumber($purchasePrice * $downPayment)}
          </output>
        {:else}
          %
          <output>{formatNumber(($downPayment / $purchasePrice) * 100)}</output>
        {/if}
      </label>
      <label>
        Months
        <output>{$totals.months}</output>
      </label>
      <label>
        Years
        <output>{$totals.years}</output>
      </label>
      <label>
        Interest
        <output class="dollar">{formatNumber($totals.interest)}</output>
      </label>
    </fieldset>
  </form>

  <hr />

  <table>
    <thead>
      <tr>
        <th>Interest Payment</th>
        <th>Interest Total</th>
        <th>Principal Payment</th>
        <th>Principal Total</th>
        <th>Month</th>
        <th>Year</th>
      </tr>
    </thead>
    <tbody>
      {#each $amortization as values, month}
        <tr class:year={(month + 1) % 12 === 0}>
          {#each values as value}
            <td>{formatNumber(value)}</td>
          {/each}
          <td>{month + 1}</td>
          <td>{Math.ceil((month + 1) / 12)}</td>
        </tr>
      {/each}
    </tbody>
  </table>
</main>

<style lang="css">
  .dollar::before {
    content: "$";
  }

  .percent::before {
    content: "%";
  }

  form {
    width: fit-content;
  }

  label {
    display: grid;
    justify-content: space-between;
    gap: 1em;
    grid-template-columns: repeat(2, 1fr);
  }

  table {
    border-collapse: collapse;
    position: relative;
  }

  thead {
    position: sticky;
    top: 0;
    background-color: white;
  }

  th,
  td {
    border: 1px solid black;
    padding: 0.25em;
  }

  tr.year {
    background-color: lightgray;
  }

  td {
    text-align: right;
  }
</style>
