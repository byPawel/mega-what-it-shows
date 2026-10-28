<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // Counted with the desk's own view (causes.ts), plus where each problem is filed now.
  // tickets: { parser, hint, shift } → issue URL. `shift` is the desk's one-day moves (desk-notes.json).
  let { ticket, tickets = {} }: { ticket: number; tickets?: Record<string, string> } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const shift = $derived(groups.map((g) => ({ cause: g.cause, n: readAs(g, notes.oneDayMove) })).filter((m) => m.n));
  const issue = (url?: string) => (url ? '#' + url.split('/').pop() : '');
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports in #{ticket} ·
  <strong>{groups.length + (shift.length ? 1 : 0)}</strong> problems
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.cause)}
      <tr>
        <td><code>{g.cause}</code></td>
        <td>{g.reports.length}</td>
        <td>{g.tenants}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>{#if tickets[g.cause]}<a href={tickets[g.cause]}>{issue(tickets[g.cause])}</a>{:else}<span class="none">none yet</span>{/if}</td>
      </tr>
    {/each}
    {#each shift as m (m.cause)}
      <tr class="sub">
        <td><code>shift</code></td>
        <td>{m.n}</td>
        <td colspan="2">read as a one-day move; counted under <code>{m.cause}</code> by tenant</td>
        <td>{#if tickets.shift}<a href={tickets.shift}>{issue(tickets.shift)}</a>{:else}<span class="none">none yet</span>{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
  .sub td { color: var(--muted); font-size: 0.9em; }
  .none { color: var(--warn); font-weight: 600; }
</style>
