<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { uniqueById, type Report } from '#lib/kb/causes';
  import { groupByProblem, type Problem } from '#lib/kb/problems';
  import { tenantOf } from '#lib/kb/locales';

  // problem → its ticket: { href, label }. Problems without one show as unticketed.
  let { ticket, tickets = {} }: { ticket: number; tickets?: Partial<Record<Problem, { href: string; label: string }>> } =
    $props();

  const rows = $derived(reports as Report[]);
  const unique = $derived(uniqueById(rows).filter((r) => r.mergedInto === ticket));
  const exported = $derived(rows.filter((r) => r.mergedInto === ticket).length);
  const groups = $derived(groupByProblem(rows, ticket, tenantOf, notes.oneDayMove));
</script>

<p class="sum">
  <strong>{unique.length}</strong> reports merged into #{ticket} ·
  <strong>{groups.length}</strong>
  {groups.length === 1 ? 'problem' : 'problems'}
  {#if exported !== unique.length}
    <span class="note">({exported} rows in the export; {exported - unique.length} came twice)</span>
  {/if}
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.problem)}
      <tr>
        <td><code>{g.problem}</code></td>
        <td>{g.reports.length}</td>
        <td>{g.tenants}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>
          {#if tickets[g.problem]}<a href={tickets[g.problem]?.href}>{tickets[g.problem]?.label}</a>{:else}<span class="none">no ticket</span>{/if}
        </td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
  .note { color: var(--muted); font-size: 0.85em; }
  .none { color: var(--warn); font-weight: 600; }
</style>
