<script lang="ts">
  import { v4 as uuid } from "uuid"

  type Item = {
    id: string
    description: string
    startDate: string
    everyNDays: string
    amount: string
  }
  let now = new Date()
  let sixMonths = 6 * 30 * 24 * 60 * 60 * 1000
  let endDate = $state(formatDate(new Date(now.valueOf() + sixMonths)))
  let startingBalance = $state("100000")
  let items = $state<Item[]>([
    {
      id: uuid(),
      description: "Payday",
      startDate: formatDate(now),
      everyNDays: "14",
      amount: "5000",
    },
    {
      id: uuid(),
      description: "Rent",
      startDate: formatDate(new Date(now.getFullYear(), now.getMonth(), 1)),
      everyNDays: "30",
      amount: "-900",
    },
    {
      id: uuid(),
      description: "Insurance",
      startDate: formatDate(new Date(now.getFullYear(), now.getMonth(), 1)),
      everyNDays: "30",
      amount: "-200",
    },
    {
      id: uuid(),
      description: "Groceries",
      startDate: formatDate(now),
      everyNDays: "7",
      amount: "-100",
    },
    {
      id: uuid(),
      description: "Gym",
      startDate: formatDate(new Date(now.getFullYear(), now.getMonth(), 1)),
      everyNDays: "30",
      amount: "-20",
    },
  ])

  type Transaction = {
    id: string
    itemId: string
    date: Date
    description: string
    amount: number
  }
  let transactions = $derived.by(() => {
    let transactions: Transaction[] = []
    items.forEach((item) => {
      if (item.startDate && item.everyNDays && item.amount) {
        let date = new Date(item.startDate)
        while (date.valueOf() < new Date(endDate).valueOf()) {
          transactions.push({
            id: uuid(),
            itemId: item.id,
            date,
            description: item.description,
            amount: Number(item.amount),
          })
          date = new Date(date)
          date.setDate(date.getDate() + Number(item.everyNDays))
        }
      }
    })
    transactions.sort((a, b) => a.date.getTime() - b.date.getTime())
    return transactions
  })

  function formatAmount(amount: number) {
    return amount.toLocaleString("en-US", {
      style: "currency",
      currency: "USD",
    })
  }

  function formatDate(date: Date) {
    return date.toISOString().slice(0, 10)
  }
</script>

<main class="mx-auto flex max-w-screen-xl flex-col items-start gap-6 p-6">
  <h1 class="text-lg font-medium">Simulator</h1>
  <p>
    This is a very simple tool to help simulate recurring transactions over a
    period of time. Simply configure the recurring transactions and watch
    transactions appear in the table below. This tool simply exists because I am
    faster with JS than with Excel.
  </p>

  <div>
    <label class="block" for="end">End date</label>
    <input
      class="w-60 border border-neutral-200"
      bind:value={endDate}
      min={formatDate(now)}
      id="end"
      type="date"
    />
  </div>

  <div class="flex gap-6">
    <div>
      <label class="block" for="balance">Starting balance</label>
      <input
        class="w-60 border border-neutral-200"
        bind:value={startingBalance}
        id="balance"
        type="number"
      />
    </div>
    <div>
      <div class="block font-medium">Ending balance</div>
      <div class="w-60 border border-neutral-200">
        {formatAmount(
          transactions.reduce(
            (total, t) => total + t.amount,
            Number(startingBalance),
          ),
        )}
      </div>
    </div>
  </div>

  <table class="w-full">
    <thead class="text-left">
      <tr>
        <th>Description</th>
        <th>Start date</th>
        <th>Every N days</th>
        <th>Amount</th>
        <th class="text-right">Total</th>
      </tr>
    </thead>
    <tbody>
      {#each items as item}
        <tr>
          <td><input bind:value={item.description} type="text" /></td>
          <td><input bind:value={item.startDate} type="date" /></td>
          <td><input bind:value={item.everyNDays} type="number" /></td>
          <td><input bind:value={item.amount} type="number" /></td>
          <td class="text-right"
            >{formatAmount(
              transactions
                .filter((t) => t.itemId === item.id)
                .reduce((total, t) => total + t.amount, 0),
            )}</td
          >
        </tr>
      {/each}
    </tbody>
  </table>

  <button
    class="bg-blue-500 px-2 py-0.5 text-white"
    onclick={() =>
      items.push({
        id: uuid(),
        description: "",
        startDate: "",
        everyNDays: "",
        amount: "",
      })}
  >
    Add item
  </button>

  <table class="w-full">
    <thead class="text-left">
      <tr>
        <th>Date</th>
        <th>Description</th>
        <th class="text-right">Amount</th>
        <th class="text-right">Balance</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>{new Date().toLocaleDateString()}</td>
        <td>Starting balance</td>
        <td class="text-right">{formatAmount(Number(startingBalance))}</td>
        <td class="text-right">{formatAmount(Number(startingBalance))}</td>
      </tr>
      {#each transactions as transaction, index}
        <tr>
          <td>{transaction.date.toLocaleDateString()}</td>
          <td>{transaction.description}</td>
          <td class="text-right">{formatAmount(transaction.amount)}</td>
          <td class="text-right"
            >{formatAmount(
              transactions
                .slice(0, index + 1)
                .reduce(
                  (total, t) => total + t.amount,
                  Number(startingBalance),
                ),
            )}</td
          >
        </tr>
      {/each}
    </tbody>
  </table>
</main>

<style>
  td {
    border: 1px solid #e5e5e5;
    border-collapse: collapse;
  }

  th {
    font-weight: 500;
  }

  label {
    font-weight: 500;
  }

  table input {
    width: 100%;
  }
</style>
