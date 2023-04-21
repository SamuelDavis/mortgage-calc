<script lang="ts">
  import { derived, writable } from "svelte/store";

  const purchasePrice = writable(325000);
  const downPayment = writable(0.2);
  const principal = derived(
    [purchasePrice, downPayment],
    ([purchasePrice, downPayment]) => {
      return purchasePrice - purchasePrice * downPayment;
    }
  );
  const monthlyPayment = writable(1500);
  const annualInterestRate = writable(0.065);
  const monthlyInterestRate = derived(
    [annualInterestRate],
    ([value]) => value / 12
  );
  const amortization = derived(
    [principal, monthlyPayment, monthlyInterestRate],
    ([principal, monthlyPayment, monthlyInterestRate]) => {
      let schedule: [number, number, number, number][] = [];
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
          interestPayment,
          totalInterest,
          principalPayment,
          totalPrincipal,
        ]);
      }
      return schedule;
    }
  );

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
      Purchase Price $<input
        type="number"
        step="0.01"
        bind:value={$purchasePrice}
      />
    </label>
    <label>
      Down Payment %<input type="number" step="1" bind:value={$downPayment} />
    </label>
    <label>
      Monthly Payment $<input
        type="number"
        step="0.01"
        bind:value={$monthlyPayment}
        min={$annualInterestRate * $principal}
      />
    </label>
    <label>
      Annual Interest Rate %<input
        type="number"
        step="0.01"
        bind:value={$annualInterestRate}
      />
    </label>
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
  form {
    width: fit-content;
  }
  label {
    display: flex;
    justify-content: space-between;
    gap: 1em;
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
