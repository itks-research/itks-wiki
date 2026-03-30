# کاوشگر منابع

<div dir="rtl" markdown>

جستجو، فیلتر و مرور تمام منابع آکادمیک فهرست‌شده در سامانه دانش توسعه ایران.

<div id="explorer-app">

<div id="explorer-loading" style="text-align:center; padding:40px; color:#666;">
  ...در حال بارگذاری منابع
</div>

<div id="explorer-ui" style="display:none;">

<!-- Filter bar -->
<div id="filters" style="display:flex; flex-wrap:wrap; gap:8px; margin-bottom:16px; align-items:center;">
  <input type="text" id="search-input" placeholder="جستجو بر اساس عنوان، نویسنده، کلیدواژه..."
    style="flex:1; min-width:200px; padding:8px 12px; border:1px solid #ccc; border-radius:4px; font-size:14px;" />
  <select id="filter-case-study" style="padding:8px; border:1px solid #ccc; border-radius:4px;">
    <option value="">همه مطالعات موردی</option>
  </select>
  <select id="filter-category" style="padding:8px; border:1px solid #ccc; border-radius:4px;">
    <option value="">همه دسته‌بندی‌ها</option>
  </select>
  <select id="filter-relevance" style="padding:8px; border:1px solid #ccc; border-radius:4px;">
    <option value="">حداقل ارتباط</option>
    <option value="3">&#8805; ۳</option>
    <option value="4">&#8805; ۴</option>
    <option value="5">فقط ۵</option>
  </select>
  <select id="filter-reliability" style="padding:8px; border:1px solid #ccc; border-radius:4px;">
    <option value="">حداقل اعتبار</option>
    <option value="3">&#8805; ۳</option>
    <option value="4">&#8805; ۴</option>
  </select>
</div>

<div id="result-info" style="margin-bottom:12px; font-size:14px; color:#666;"></div>

<!-- Results table -->
<div style="overflow-x:auto;">
<table id="source-table" style="width:100%; border-collapse:collapse; font-size:14px;">
  <thead>
    <tr style="background:#f5f5f5; cursor:pointer;">
      <th data-sort="title" style="padding:10px 8px; text-align:start; border-bottom:2px solid #ddd;">عنوان &#x25B4;&#x25BE;</th>
      <th data-sort="year" style="padding:10px 8px; text-align:start; border-bottom:2px solid #ddd; white-space:nowrap;">سال &#x25B4;&#x25BE;</th>
      <th data-sort="case_study" style="padding:10px 8px; text-align:start; border-bottom:2px solid #ddd;">مطالعه موردی &#x25B4;&#x25BE;</th>
      <th data-sort="category" style="padding:10px 8px; text-align:start; border-bottom:2px solid #ddd;">دسته‌بندی &#x25B4;&#x25BE;</th>
      <th data-sort="relevance_score" style="padding:10px 8px; text-align:start; border-bottom:2px solid #ddd;">ارتباط &#x25B4;&#x25BE;</th>
      <th data-sort="citation_count" style="padding:10px 8px; text-align:start; border-bottom:2px solid #ddd;">استنادها &#x25B4;&#x25BE;</th>
    </tr>
  </thead>
  <tbody id="source-body"></tbody>
</table>
</div>

<!-- Pagination -->
<div id="pagination" style="display:flex; justify-content:center; gap:4px; margin-top:16px; flex-wrap:wrap;"></div>

</div><!-- explorer-ui -->
</div><!-- explorer-app -->

</div>

<style>
#source-table tbody tr.source-row { cursor: pointer; transition: background 0.15s; }
#source-table tbody tr.source-row:hover { background: #f0f7ff; }
#source-table tbody tr.source-row:nth-child(4n+1) { background: #fafafa; }
#source-table tbody tr.source-row:nth-child(4n+1):hover { background: #f0f7ff; }
#source-table tbody tr.detail-row { background: #f8f9fa; }
#source-table tbody tr.detail-row td { padding: 16px 12px; border-bottom: 1px solid #eee; }
.detail-content { max-width: 800px; }
.detail-content .meta-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 8px; margin-bottom: 12px; }
.detail-content .meta-item { font-size: 13px; }
.detail-content .meta-label { font-weight: 600; color: #555; }
.detail-content .abstract-text { font-size: 13px; line-height: 1.6; color: #333; margin-top: 8px; }
.page-btn { padding: 6px 12px; border: 1px solid #ccc; border-radius: 4px; background: white; cursor: pointer; font-size: 13px; }
.page-btn:hover { background: #e8f0fe; }
.page-btn.active { background: #1a73e8; color: white; border-color: #1a73e8; }
.page-btn:disabled { opacity: 0.4; cursor: default; }
</style>

<script>
(function() {
  const PER_PAGE = 50;
  let allSources = [];
  let filtered = [];
  let currentPage = 1;
  let sortField = 'relevance_score';
  let sortDir = -1;

  const isFA = true;

  const L = {
    loading: '...در حال بارگذاری',
    noResults: '.نتیجه‌ای یافت نشد',
    showing: 'نمایش',
    of: 'از',
    sources: 'منبع',
    authors: 'نویسندگان',
    year: 'سال',
    doi: 'شناسه دیجیتال',
    venue: 'نشریه',
    openAccess: 'دسترسی آزاد',
    viewPaper: 'مشاهده مقاله',
    relevance: 'ارتباط',
    reliability: 'اعتبار',
    citations: 'استنادها',
    abstract: 'چکیده',
  };

  fetch('../../data/sources.json')
    .then(r => r.json())
    .then(data => {
      allSources = data;
      populateDropdowns();
      readURLParams();
      applyFilters();
      document.getElementById('explorer-loading').style.display = 'none';
      document.getElementById('explorer-ui').style.display = 'block';
    })
    .catch(err => {
      document.getElementById('explorer-loading').textContent = 'خطا در بارگذاری: ' + err.message;
    });

  function populateDropdowns() {
    const csSet = new Set(), catSet = new Set();
    allSources.forEach(s => {
      if (s.case_study) csSet.add(s.case_study);
      if (s.category) catSet.add(s.category);
    });
    const csSelect = document.getElementById('filter-case-study');
    [...csSet].sort().forEach(cs => {
      const count = allSources.filter(s => s.case_study === cs).length;
      const opt = document.createElement('option');
      opt.value = cs;
      opt.textContent = cs + ' (' + count + ')';
      csSelect.appendChild(opt);
    });
    const catSelect = document.getElementById('filter-category');
    [...catSet].sort().forEach(cat => {
      const opt = document.createElement('option');
      opt.value = cat;
      opt.textContent = cat;
      catSelect.appendChild(opt);
    });
  }

  function readURLParams() {
    const params = new URLSearchParams(window.location.search);
    if (params.get('case_study')) document.getElementById('filter-case-study').value = params.get('case_study');
    if (params.get('category')) document.getElementById('filter-category').value = params.get('category');
    if (params.get('min_relevance')) document.getElementById('filter-relevance').value = params.get('min_relevance');
    if (params.get('min_reliability')) document.getElementById('filter-reliability').value = params.get('min_reliability');
    if (params.get('search')) document.getElementById('search-input').value = params.get('search');
    if (params.get('id')) window._highlightId = parseInt(params.get('id'));
  }

  function updateURL() {
    const params = new URLSearchParams();
    const search = document.getElementById('search-input').value.trim();
    const cs = document.getElementById('filter-case-study').value;
    const cat = document.getElementById('filter-category').value;
    const rel = document.getElementById('filter-relevance').value;
    const rl = document.getElementById('filter-reliability').value;
    if (search) params.set('search', search);
    if (cs) params.set('case_study', cs);
    if (cat) params.set('category', cat);
    if (rel) params.set('min_relevance', rel);
    if (rl) params.set('min_reliability', rl);
    const qs = params.toString();
    history.replaceState(null, '', window.location.pathname + (qs ? '?' + qs : ''));
  }

  function applyFilters() {
    const search = document.getElementById('search-input').value.trim().toLowerCase();
    const cs = document.getElementById('filter-case-study').value;
    const cat = document.getElementById('filter-category').value;
    const minRel = parseFloat(document.getElementById('filter-relevance').value) || 0;
    const minRl = parseFloat(document.getElementById('filter-reliability').value) || 0;

    filtered = allSources.filter(s => {
      if (cs && s.case_study !== cs) return false;
      if (cat && s.category !== cat) return false;
      if (minRel && (!s.relevance_score || s.relevance_score < minRel)) return false;
      if (minRl && (!s.reliability_score || s.reliability_score < minRl)) return false;
      if (search) {
        const haystack = ((s.title || '') + ' ' + (s.authors || []).join(' ') + ' ' + (s.abstract || '') + ' ' + (s.category || '')).toLowerCase();
        if (!haystack.includes(search)) return false;
      }
      return true;
    });

    sortSources();
    currentPage = 1;
    if (window._highlightId) {
      const idx = filtered.findIndex(s => s.id === window._highlightId);
      if (idx >= 0) currentPage = Math.floor(idx / PER_PAGE) + 1;
    }
    renderTable();
    renderPagination();
    updateResultInfo();
    updateURL();
  }

  function sortSources() {
    filtered.sort((a, b) => {
      let va = a[sortField], vb = b[sortField];
      if (va == null) va = sortDir > 0 ? Infinity : -Infinity;
      if (vb == null) vb = sortDir > 0 ? Infinity : -Infinity;
      if (typeof va === 'string') va = va.toLowerCase();
      if (typeof vb === 'string') vb = vb.toLowerCase();
      if (va < vb) return -1 * sortDir;
      if (va > vb) return 1 * sortDir;
      return 0;
    });
  }

  function renderTable() {
    const tbody = document.getElementById('source-body');
    tbody.innerHTML = '';
    const start = (currentPage - 1) * PER_PAGE;
    const page = filtered.slice(start, start + PER_PAGE);

    if (page.length === 0) {
      tbody.innerHTML = '<tr><td colspan="6" style="padding:20px; text-align:center; color:#999;">' + L.noResults + '</td></tr>';
      return;
    }

    page.forEach(s => {
      const tr = document.createElement('tr');
      tr.className = 'source-row';
      tr.dataset.id = s.id;
      tr.innerHTML =
        '<td style="padding:8px;">' + escHtml(s.title) + '</td>' +
        '<td style="padding:8px;">' + (s.year || '\u2014') + '</td>' +
        '<td style="padding:8px;">' + escHtml(s.case_study || '\u2014') + '</td>' +
        '<td style="padding:8px; font-size:12px;">' + escHtml(s.category || '\u2014') + '</td>' +
        '<td style="padding:8px;">' + (s.relevance_score ? s.relevance_score.toFixed(1) : '\u2014') + '</td>' +
        '<td style="padding:8px;">' + (s.citation_count || 0) + '</td>';
      tr.addEventListener('click', function() { toggleDetail(s, tr); });
      tbody.appendChild(tr);
    });

    if (window._highlightId) {
      const row = tbody.querySelector('tr[data-id="' + window._highlightId + '"]');
      if (row) {
        const src = filtered.find(s => s.id === window._highlightId);
        if (src) { row.style.background = '#fff3cd'; toggleDetail(src, row); }
      }
      window._highlightId = null;
    }
  }

  function toggleDetail(source, rowEl) {
    const existing = rowEl.nextElementSibling;
    if (existing && existing.classList.contains('detail-row')) { existing.remove(); return; }
    document.querySelectorAll('.detail-row').forEach(el => el.remove());

    const detailTr = document.createElement('tr');
    detailTr.className = 'detail-row';
    const abstractText = source.fa_summary || source.abstract || 'چکیده‌ای موجود نیست.';
    const authors = (source.authors || []).join(', ') || 'نامشخص';

    let html = '<div class="detail-content">';
    html += '<div class="meta-grid">';
    html += '<div class="meta-item"><span class="meta-label">' + L.authors + ':</span> ' + escHtml(authors) + '</div>';
    html += '<div class="meta-item"><span class="meta-label">' + L.year + ':</span> ' + (source.year || '\u2014') + '</div>';
    html += '<div class="meta-item"><span class="meta-label">' + L.relevance + ':</span> ' + (source.relevance_score ? source.relevance_score.toFixed(1) + '/5' : '\u2014') + '</div>';
    html += '<div class="meta-item"><span class="meta-label">' + L.reliability + ':</span> ' + (source.reliability_score ? source.reliability_score.toFixed(1) + '/5' : '\u2014') + '</div>';
    html += '<div class="meta-item"><span class="meta-label">' + L.citations + ':</span> ' + (source.citation_count || 0) + '</div>';
    if (source.venue_name) html += '<div class="meta-item"><span class="meta-label">' + L.venue + ':</span> ' + escHtml(source.venue_name) + '</div>';
    html += '</div>';
    if (source.doi) html += '<div style="margin:8px 0;"><span class="meta-label">' + L.doi + ':</span> <a href="https://doi.org/' + escHtml(source.doi) + '" target="_blank" rel="noopener">' + escHtml(source.doi) + '</a></div>';
    if (source.open_access_url) html += '<div style="margin:8px 0;"><a href="' + escHtml(source.open_access_url) + '" target="_blank" rel="noopener" style="color:#1a73e8;">' + L.viewPaper + ' (' + L.openAccess + ')</a></div>';
    html += '<div style="margin-top:12px;"><span class="meta-label">' + L.abstract + ':</span></div>';
    html += '<div class="abstract-text">' + escHtml(abstractText) + '</div>';
    html += '</div>';

    const td = document.createElement('td');
    td.colSpan = 6;
    td.innerHTML = html;
    detailTr.appendChild(td);
    rowEl.after(detailTr);
  }

  function renderPagination() {
    const totalPages = Math.ceil(filtered.length / PER_PAGE);
    const container = document.getElementById('pagination');
    container.innerHTML = '';
    if (totalPages <= 1) return;
    const prev = document.createElement('button');
    prev.className = 'page-btn'; prev.textContent = 'قبلی'; prev.disabled = currentPage === 1;
    prev.addEventListener('click', () => { currentPage--; renderTable(); renderPagination(); updateResultInfo(); });
    container.appendChild(prev);
    const start = Math.max(1, currentPage - 3);
    const end = Math.min(totalPages, currentPage + 3);
    if (start > 1) { addPageBtn(container, 1); if (start > 2) addEllipsis(container); }
    for (let i = start; i <= end; i++) addPageBtn(container, i);
    if (end < totalPages) { if (end < totalPages - 1) addEllipsis(container); addPageBtn(container, totalPages); }
    const next = document.createElement('button');
    next.className = 'page-btn'; next.textContent = 'بعدی'; next.disabled = currentPage === totalPages;
    next.addEventListener('click', () => { currentPage++; renderTable(); renderPagination(); updateResultInfo(); });
    container.appendChild(next);
  }

  function addPageBtn(container, num) {
    const btn = document.createElement('button');
    btn.className = 'page-btn' + (num === currentPage ? ' active' : '');
    btn.textContent = num;
    btn.addEventListener('click', () => { currentPage = num; renderTable(); renderPagination(); updateResultInfo(); });
    container.appendChild(btn);
  }
  function addEllipsis(container) {
    const span = document.createElement('span');
    span.textContent = '...'; span.style.padding = '6px 4px';
    container.appendChild(span);
  }

  function updateResultInfo() {
    const s = (currentPage - 1) * PER_PAGE + 1;
    const e = Math.min(currentPage * PER_PAGE, filtered.length);
    const info = document.getElementById('result-info');
    info.textContent = filtered.length === 0 ? L.noResults : L.showing + ' ' + s + '\u2013' + e + ' ' + L.of + ' ' + filtered.length + ' ' + L.sources;
  }

  function escHtml(str) {
    if (!str) return '';
    return str.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
  }

  let searchTimeout;
  document.getElementById('search-input').addEventListener('input', function() {
    clearTimeout(searchTimeout);
    searchTimeout = setTimeout(applyFilters, 300);
  });
  document.getElementById('filter-case-study').addEventListener('change', applyFilters);
  document.getElementById('filter-category').addEventListener('change', applyFilters);
  document.getElementById('filter-relevance').addEventListener('change', applyFilters);
  document.getElementById('filter-reliability').addEventListener('change', applyFilters);
  document.querySelectorAll('#source-table th[data-sort]').forEach(th => {
    th.addEventListener('click', function() {
      const field = this.dataset.sort;
      if (sortField === field) { sortDir *= -1; } else { sortField = field; sortDir = field === 'title' ? 1 : -1; }
      sortSources(); currentPage = 1; renderTable(); renderPagination(); updateResultInfo();
    });
  });
})();
</script>
