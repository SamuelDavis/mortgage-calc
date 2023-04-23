<script lang="ts">
  import { derived, writable } from "svelte/store";

  const purchasePrice = writable(325000);
  const downPayment = writable(0.2);
  const principal = derived(
    [purchasePrice, downPayment],
    ([purchasePrice, downPayment]) =>
      downPayment < 1
        ? purchasePrice - purchasePrice * downPayment
        : purchasePrice - downPayment
  );
  const annualInterestRate = writable(0.065);
  const monthlyInterestRate = derived(
    [annualInterestRate],
    ([annualInterestRate]) => annualInterestRate / 12
  );
  const monthlyPayment = writable(
    Math.ceil($principal * $monthlyInterestRate * 1.2)
  );
  const amortization = derived(
    [principal, monthlyInterestRate, monthlyPayment],
    ([principal, monthlyInterestRate, monthlyPayment]) => {
      let schedule: any[] = [];
      let totalInterest = 0;
      let totalPrincipal = 0;
      while (principal > 0) {
        const interestPayment = principal * monthlyInterestRate;
        const principalPayment = Math.min(
          principal,
          monthlyPayment - interestPayment
        );
        if (principalPayment <= 0) return [];
        principal -= principalPayment;
        totalInterest += interestPayment;
        totalPrincipal += principalPayment;
        schedule.push([
          totalInterest,
          totalPrincipal,
          interestPayment,
          principalPayment,
        ]);
      }
      return schedule;
    }
  );
  const totals = derived([amortization], ([amortization]) => {
    const months = amortization.length + 1;
    const years = Math.ceil(months / 12);
    const interest =
      amortization.length === 0 ? 0 : amortization[amortization.length - 1][0];

    return { months, years, interest };
  });

  function formatNumber(value: number): string {
    const [integerPart, fractionPart] = value.toFixed(2).split(".");
    return `${parseInt(integerPart, 10).toLocaleString()}.${fractionPart}`;
  }
</script>

<main>
  <form on:submit|preventDefault>
    <fieldset>
      <legend>Input</legend>
      <label>
        <span>Purchase Price</span>
        <input bind:value={$purchasePrice} min="0" step="1" type="number" />
      </label>
      <label>
        <span>Down Payment</span>
        <input bind:value={$downPayment} min="0" step="0.05" type="number" />
      </label>
      <label>
        Interest Rate <input
          bind:value={$annualInterestRate}
          max="1"
          min="0"
          step="0.0025"
          type="number"
        />
      </label>
      <label>
        Monthly Payment <input
          bind:value={$monthlyPayment}
          min="0"
          step="100"
          type="number"
        />
      </label>
    </fieldset>
    <fieldset>
      <legend>Summary</legend>
      <label>
        <span>Principal</span>
        <output>{formatNumber($principal)}</output>
      </label>
      <label>
        <span>Down Payment</span>
        <output>
          {$downPayment < 1
            ? formatNumber($purchasePrice * $downPayment)
            : ($downPayment / $purchasePrice).toFixed(2)}
        </output>
      </label>
      <label>
        <span>Interest</span>
        <output>{formatNumber($totals.interest)}</output>
      </label>
      <label>
        Months
        <output>{$totals.months.toLocaleString()}</output>
      </label>
      <label>
        Years
        <output>{$totals.years.toLocaleString()}</output>
      </label>
    </fieldset>
  </form>
  <hr />
  <table>
    <thead>
      <tr>
        <th>Interest Total</th>
        <th>Principal Total</th>
        <th>Interest Payment</th>
        <th>Principal Payment</th>
        <th>Months</th>
        <th>Years</th>
      </tr>
    </thead>
    <tbody>
      {#each $amortization as values, i}
        <tr>
          {#each values as value}
            <td>{formatNumber(value)}</td>
          {/each}
          <td>{i + 1}</td>
          <td>{Math.ceil((i + 1) / 12)}</td>
        </tr>
      {/each}
    </tbody>
  </table>
</main>

<style>
  form {
    max-width: fit-content;
    display: flex;
  }

  label {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 0.5em;
  }

  table {
    border-collapse: collapse;
  }

  th,
  td {
    border: 1px solid black;
    padding: 0.5em;
    text-align: right;
  }
</style>
