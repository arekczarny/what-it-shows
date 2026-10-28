<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import { groupByCause, uniqueById, type Cause, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  /** Maps each cause to the desk ticket (GitHub issue) URL for that problem. */
  let {
    ticket,
    issueByCause,
  }: {
    ticket: number;
    issueByCause: Partial<Record<Cause | 'unplaced', string>>;
  } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));

  const label: Record<string, string> = {
    parser: 'Shared parser reads dates month-first; day-first tenants get month/day flipped.',
    hint: 'booking-v3 prints an MM/DD/YYYY hint; the form reads day-first. People type the hint; the form flips it.',
    unplaced: 'No tenant on file, or tenant locale is month-first.',
  };
</script>

<p class="sum">
  <strong>{merged.length}</strong> unique reports merged into #{ticket}
  · after dropping export-page duplicates via <code>uniqueById</code>
</p>

<table>
  <thead>
    <tr><th>cause</th><th>what breaks</th><th>reports</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.cause)}
      <tr>
        <td><code>{g.cause}</code></td>
        <td>{label[g.cause] ?? ''}</td>
        <td>{g.reports.length}</td>
        <td>
          {#if issueByCause[g.cause]}
            <a href={issueByCause[g.cause]}>{issueByCause[g.cause]?.replace(/^.*\//, '#')}</a>
          {:else}
            no ticket yet
          {/if}
        </td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
</style>
