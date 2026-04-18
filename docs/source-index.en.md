---
search:
  exclude: true
---

# Source Index

Interactive sortable index of all **16,651** sources in the ITKS research database.
Use the search box, column sorting, and dropdown filters to explore.

<div id="filter-row" style="margin-bottom:1rem;display:flex;gap:1rem;flex-wrap:wrap;">
  <select id="filter-casestudy" class="form-select" style="max-width:220px;padding:6px 10px;border:1px solid #ccc;border-radius:4px;">
    <option value="">All Case Studies</option>
  </select>
  <select id="filter-category" class="form-select" style="max-width:300px;padding:6px 10px;border:1px solid #ccc;border-radius:4px;">
    <option value="">All Categories</option>
  </select>
  <select id="filter-score" class="form-select" style="max-width:180px;padding:6px 10px;border:1px solid #ccc;border-radius:4px;">
    <option value="">All Scores</option>
    <option value="5">5.0</option>
    <option value="4">4.0+</option>
    <option value="3">3.0+</option>
    <option value="2">2.0+</option>
    <option value="1">1.0+</option>
  </select>
</div>

<table id="sources-table" class="display compact stripe" style="width:100%">
  <thead>
    <tr>
      <th>ID</th>
      <th>Title</th>
      <th>Year</th>
      <th>Case Study</th>
      <th>Category</th>
      <th>Relevance</th>
      <th>Citations</th>
      <th>Source</th>
    </tr>
  </thead>
</table>

<link rel="stylesheet" href="https://cdn.datatables.net/1.13.8/css/jquery.dataTables.min.css">
<script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
<script src="https://cdn.datatables.net/1.13.8/js/jquery.dataTables.min.js"></script>

<script>
(function initSourceIndex() {
  if (typeof $ === 'undefined' || typeof $.fn.DataTable === 'undefined') {
    setTimeout(initSourceIndex, 50);
    return;
  }
  fetch('data/sources_index.json')
    .then(r => r.json())
    .then(data => {
      const caseStudies = [...new Set(data.map(d => d.case_study).filter(Boolean))].sort();
      const categories = [...new Set(data.map(d => d.category).filter(Boolean))].sort();
      const csSel = document.getElementById('filter-casestudy');
      const catSel = document.getElementById('filter-category');
      caseStudies.forEach(cs => { const o = document.createElement('option'); o.value = cs; o.textContent = cs; csSel.appendChild(o); });
      categories.forEach(cat => { const o = document.createElement('option'); o.value = cat; o.textContent = cat; catSel.appendChild(o); });

      const table = $('#sources-table').DataTable({
        data: data,
        columns: [
          { data: 'id' },
          { data: 'title', render: function(d, type, row) {
              if (type === 'display') {
                const fn = String(row.id).padStart(4,'0') + '-' + d.replace(/[^a-zA-Z0-9 \-_]/g,'').trim().replace(/ /g,'-').toLowerCase().substring(0,80);
                return '<a href="sources/' + fn + '/">' + (d.length > 80 ? d.substring(0,77) + '...' : d) + '</a>';
              }
              return d;
            }
          },
          { data: 'year' },
          { data: 'case_study' },
          { data: 'category' },
          { data: 'relevance_score', render: function(d) { return d ? Number(d).toFixed(1) : '-'; } },
          { data: 'citation_count', render: function(d) { return d || 0; } },
          { data: 'source_api' }
        ],
        pageLength: 50,
        order: [[5, 'desc'], [6, 'desc']],
        language: { search: 'Filter:' },
        deferRender: true
      });

      csSel.addEventListener('change', function() { table.column(3).search(this.value ? '^' + $.fn.dataTable.util.escapeRegex(this.value) + '$' : '', true, false).draw(); });
      catSel.addEventListener('change', function() { table.column(4).search(this.value ? '^' + $.fn.dataTable.util.escapeRegex(this.value) + '$' : '', true, false).draw(); });
      document.getElementById('filter-score').addEventListener('change', function() {
        var v = this.value;
        $.fn.dataTable.ext.search.length = 0;
        if (v) {
          $.fn.dataTable.ext.search.push(function(settings, data) { return parseFloat(data[5]) >= parseFloat(v); });
        }
        table.draw();
      });
    });
})();
</script>

<style>
#sources-table_wrapper .dataTables_filter { margin-bottom: 0.5rem; }
#sources-table td { font-size: 0.85rem; }
#sources-table th { font-size: 0.85rem; }
</style>

