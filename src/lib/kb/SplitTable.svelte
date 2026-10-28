<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  // The desk's view: reports counted under the cause they are filed by. The desk's one-day moves
  // stay inside their cause and are shown as a note, so each report is counted once.
  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const oneDay = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));
  const total = $derived(groups.reduce((n, g) => n + g.reports.length, 0));</script>

<p class="sum">
  <strong>{merged.length}</strong> reports in #{ticket} · <strong>{groups.length}</strong> causes ·
  {groups.map((g) => g.reports.length).join(' + ')} = {total}
</p>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>tenants</th><th>channels</th><th>intake tagged it differently</th></tr></thead>
  <tbody>
    {#each groups as g (g.cause)}
      <tr>
        <td><code>{g.cause}</code></td>
        <td>{g.reports.length}</td>
        <td>{g.tenants}</td>
        <td>{[...new Set(g.reports.map((r) => r.channel))].sort().join(', ')}</td>
        <td>{g.reports.filter((r) => r.meta.intakeTag !== g.cause).length}</td>
      </tr>
    {/each}
  </tbody>
</table>

<p class="note">
  Of those, <strong>{oneDay.length}</strong> reports read as a one-day move, not a swapped month and day. They are filed under
  <code>{groups.find((g) => oneDay.every((r) => g.reports.includes(r)))?.cause ?? 'parser'}</code> by tenant and counted there once. They are a third problem with its own ticket.
</p>
<style>
  .sum { font-size: 1.1rem; }
  .note { color: var(--muted); font-size: 0.9em; }
</style>
