<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  // The problems inside one ticket: the desk's one-day moves are pulled out of the cause they were filed under.
  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const rows = $derived.by(() => {
    const oneDay = merged.filter((r) => notes.oneDayMove.includes(r.id));
    const rest = merged.filter((r) => !notes.oneDayMove.includes(r.id));
    const groups = groupByCause(rest, tenantOf);
    return [
      ...groups.map((g) => ({
        problem: g.cause,
        count: g.reports.length,
        tenants: g.tenants,
        channels: [...new Set(g.reports.map((r) => r.channel))].sort().join(', '),
        wronglyTagged: g.reports.filter((r) => r.meta.intakeTag !== g.cause).length,
      })),
      {
        problem: 'one-day move',
        count: oneDay.length,
        tenants: new Set(oneDay.map((r) => r.tenant)).size,
        channels: [...new Set(oneDay.map((r) => r.channel))].sort().join(', '),
        wronglyTagged: oneDay.length,
      },
    ];
  });
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports in #{ticket} · <strong>{rows.length}</strong> problems ·
  {rows.map((r) => r.count).join(' + ')} = {rows.reduce((n, r) => n + r.count, 0)}
</p>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>tenants</th><th>channels</th><th>intake tagged it differently</th></tr></thead>
  <tbody>
    {#each rows as r (r.problem)}
      <tr><td><code>{r.problem}</code></td><td>{r.count}</td><td>{r.tenants}</td><td>{r.channels}</td><td>{r.wronglyTagged}</td></tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
</style>
