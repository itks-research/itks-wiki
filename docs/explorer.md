# کاوشگر منابع

<div dir="rtl" markdown>

جستجو، فیلتر و مرور تمام منابع آکادمیک فهرست‌شده در سامانه دانش توسعه ایران.

<div id="explorer-app">

<div id="explorer-loading" style="text-align:center; padding:40px;">
  ...در حال بارگذاری منابع
</div>

<div id="explorer-ui" style="display:none;">

<div id="filters">
  <input type="text" id="search-input" placeholder="جستجو بر اساس عنوان، نویسنده، کلیدواژه..." style="flex:1; min-width:200px;" />
  <select id="filter-case-study"><option value="">همه مطالعات موردی</option></select>
  <select id="filter-category"><option value="">همه دسته‌بندی‌ها</option></select>
  <select id="filter-relevance"><option value="">حداقل ارتباط</option><option value="3">&#8805; ۳</option><option value="4">&#8805; ۴</option><option value="5">فقط ۵</option></select>
  <select id="filter-reliability"><option value="">حداقل اعتبار</option><option value="3">&#8805; ۳</option><option value="4">&#8805; ۴</option></select>
</div>

<div id="result-info"></div>

<div style="overflow-x:auto;">
<table id="source-table">
  <thead>
    <tr>
      <th data-sort="title">عنوان &#x25B4;&#x25BE;</th>
      <th data-sort="year" style="white-space:nowrap;">سال &#x25B4;&#x25BE;</th>
      <th data-sort="case_study">مطالعه موردی &#x25B4;&#x25BE;</th>
      <th data-sort="category">دسته‌بندی &#x25B4;&#x25BE;</th>
      <th data-sort="relevance_score">ارتباط &#x25B4;&#x25BE;</th>
      <th data-sort="citation_count">استنادها &#x25B4;&#x25BE;</th>
    </tr>
  </thead>
  <tbody id="source-body"></tbody>
</table>
</div>

<div id="pagination" style="display:flex; justify-content:center; gap:4px; margin-top:16px; flex-wrap:wrap;"></div>

</div>
</div>

</div>

<script>
(function() {
  const PER_PAGE = 50;
  let allSources = [], filtered = [], currentPage = 1, sortField = 'relevance_score', sortDir = -1;
  const isFA = true;
  const L = { loading:'...در حال بارگذاری', noResults:'.نتیجه‌ای یافت نشد', showing:'نمایش', of:'از', sources:'منبع', authors:'نویسندگان', year:'سال', doi:'شناسه دیجیتال', venue:'نشریه', openAccess:'دسترسی آزاد', viewPaper:'مشاهده مقاله', relevance:'ارتباط', reliability:'اعتبار', citations:'استنادها', abstract:'چکیده' };

  fetch('../data/sources.json').then(r=>r.json()).then(data => {
    allSources = data; populateDropdowns(); readURLParams(); applyFilters();
    document.getElementById('explorer-loading').style.display='none';
    document.getElementById('explorer-ui').style.display='block';
  }).catch(err => { document.getElementById('explorer-loading').textContent='خطا: '+err.message; });

  function populateDropdowns() {
    const csSet=new Set(),catSet=new Set();
    allSources.forEach(s=>{if(s.case_study)csSet.add(s.case_study);if(s.category)catSet.add(s.category);});
    const cs=document.getElementById('filter-case-study');
    [...csSet].sort().forEach(c=>{const o=document.createElement('option');o.value=c;o.textContent=c+' ('+allSources.filter(s=>s.case_study===c).length+')';cs.appendChild(o);});
    const cat=document.getElementById('filter-category');
    [...catSet].sort().forEach(c=>{const o=document.createElement('option');o.value=c;o.textContent=c;cat.appendChild(o);});
  }
  function readURLParams(){const p=new URLSearchParams(window.location.search);if(p.get('case_study'))document.getElementById('filter-case-study').value=p.get('case_study');if(p.get('category'))document.getElementById('filter-category').value=p.get('category');if(p.get('min_relevance'))document.getElementById('filter-relevance').value=p.get('min_relevance');if(p.get('min_reliability'))document.getElementById('filter-reliability').value=p.get('min_reliability');if(p.get('search'))document.getElementById('search-input').value=p.get('search');if(p.get('id'))window._highlightId=parseInt(p.get('id'));}
  function updateURL(){const p=new URLSearchParams();const v=id=>document.getElementById(id).value;const s=document.getElementById('search-input').value.trim();if(s)p.set('search',s);if(v('filter-case-study'))p.set('case_study',v('filter-case-study'));if(v('filter-category'))p.set('category',v('filter-category'));if(v('filter-relevance'))p.set('min_relevance',v('filter-relevance'));if(v('filter-reliability'))p.set('min_reliability',v('filter-reliability'));history.replaceState(null,'',window.location.pathname+(p.toString()?'?'+p:''));}
  function applyFilters(){const search=document.getElementById('search-input').value.trim().toLowerCase();const cs=document.getElementById('filter-case-study').value;const cat=document.getElementById('filter-category').value;const minRel=parseFloat(document.getElementById('filter-relevance').value)||0;const minRl=parseFloat(document.getElementById('filter-reliability').value)||0;filtered=allSources.filter(s=>{if(cs&&s.case_study!==cs)return false;if(cat&&s.category!==cat)return false;if(minRel&&(!s.relevance_score||s.relevance_score<minRel))return false;if(minRl&&(!s.reliability_score||s.reliability_score<minRl))return false;if(search){const h=((s.title||'')+' '+(s.authors||[]).join(' ')+' '+(s.abstract||'')+' '+(s.category||'')).toLowerCase();if(!h.includes(search))return false;}return true;});sortSources();currentPage=1;if(window._highlightId){const idx=filtered.findIndex(s=>s.id===window._highlightId);if(idx>=0)currentPage=Math.floor(idx/PER_PAGE)+1;}renderTable();renderPagination();updateResultInfo();updateURL();}
  function sortSources(){filtered.sort((a,b)=>{let va=a[sortField],vb=b[sortField];if(va==null)va=sortDir>0?Infinity:-Infinity;if(vb==null)vb=sortDir>0?Infinity:-Infinity;if(typeof va==='string'){va=va.toLowerCase();vb=(vb||'').toLowerCase();}return va<vb?-sortDir:va>vb?sortDir:0;});}
  function renderTable(){const tbody=document.getElementById('source-body');tbody.innerHTML='';const start=(currentPage-1)*PER_PAGE;const page=filtered.slice(start,start+PER_PAGE);if(!page.length){tbody.innerHTML='<tr><td colspan="6" style="padding:20px;text-align:center;">'+L.noResults+'</td></tr>';return;}page.forEach(s=>{const tr=document.createElement('tr');tr.className='source-row';tr.dataset.id=s.id;tr.innerHTML='<td>'+escHtml(s.title)+'</td><td>'+(s.year||'\u2014')+'</td><td>'+escHtml(s.case_study||'\u2014')+'</td><td style="font-size:12px;">'+escHtml(s.category||'\u2014')+'</td><td>'+(s.relevance_score?s.relevance_score.toFixed(1):'\u2014')+'</td><td>'+(s.citation_count||0)+'</td>';tr.addEventListener('click',()=>toggleDetail(s,tr));tbody.appendChild(tr);});if(window._highlightId){const row=tbody.querySelector('tr[data-id="'+window._highlightId+'"]');if(row){const src=filtered.find(s=>s.id===window._highlightId);if(src){row.style.background='var(--itks-hover)';toggleDetail(src,row);}}window._highlightId=null;}}
  function toggleDetail(source,rowEl){const ex=rowEl.nextElementSibling;if(ex&&ex.classList.contains('detail-row')){ex.remove();return;}document.querySelectorAll('.detail-row').forEach(el=>el.remove());const tr=document.createElement('tr');tr.className='detail-row';const abs=source.fa_summary||source.abstract||'چکیده‌ای موجود نیست.';const auth=(source.authors||[]).join(', ')||'نامشخص';let h='<div class="detail-content"><div class="meta-grid">';h+='<div class="meta-item"><span class="meta-label">'+L.authors+':</span> '+escHtml(auth)+'</div>';h+='<div class="meta-item"><span class="meta-label">'+L.year+':</span> '+(source.year||'\u2014')+'</div>';h+='<div class="meta-item"><span class="meta-label">'+L.relevance+':</span> '+(source.relevance_score?source.relevance_score.toFixed(1)+'/5':'\u2014')+'</div>';h+='<div class="meta-item"><span class="meta-label">'+L.reliability+':</span> '+(source.reliability_score?source.reliability_score.toFixed(1)+'/5':'\u2014')+'</div>';h+='<div class="meta-item"><span class="meta-label">'+L.citations+':</span> '+(source.citation_count||0)+'</div>';if(source.venue_name)h+='<div class="meta-item"><span class="meta-label">'+L.venue+':</span> '+escHtml(source.venue_name)+'</div>';h+='</div>';if(source.doi)h+='<div style="margin:8px 0;"><span class="meta-label">'+L.doi+':</span> <a href="https://doi.org/'+escHtml(source.doi)+'" target="_blank" rel="noopener">'+escHtml(source.doi)+'</a></div>';if(source.open_access_url)h+='<div style="margin:8px 0;"><a href="'+escHtml(source.open_access_url)+'" target="_blank" rel="noopener">'+L.viewPaper+' ('+L.openAccess+')</a></div>';h+='<div style="margin-top:12px;"><span class="meta-label">'+L.abstract+':</span></div><div class="abstract-text">'+escHtml(abs)+'</div></div>';const td=document.createElement('td');td.colSpan=6;td.innerHTML=h;tr.appendChild(td);rowEl.after(tr);}
  function renderPagination(){const tp=Math.ceil(filtered.length/PER_PAGE),c=document.getElementById('pagination');c.innerHTML='';if(tp<=1)return;const mk=(txt,pg,dis)=>{const b=document.createElement('button');b.className='page-btn'+(pg===currentPage?' active':'');b.textContent=txt;b.disabled=!!dis;b.addEventListener('click',()=>{currentPage=pg;renderTable();renderPagination();updateResultInfo();});return b;};c.appendChild(mk('قبلی',currentPage-1,currentPage===1));const s=Math.max(1,currentPage-3),e=Math.min(tp,currentPage+3);if(s>1){c.appendChild(mk('1',1));if(s>2){const sp=document.createElement('span');sp.textContent='...';sp.style.padding='6px 4px';c.appendChild(sp);}}for(let i=s;i<=e;i++)c.appendChild(mk(i,i));if(e<tp){if(e<tp-1){const sp=document.createElement('span');sp.textContent='...';sp.style.padding='6px 4px';c.appendChild(sp);}c.appendChild(mk(tp,tp));}c.appendChild(mk('بعدی',currentPage+1,currentPage===tp));}
  function updateResultInfo(){const s=(currentPage-1)*PER_PAGE+1,e=Math.min(currentPage*PER_PAGE,filtered.length);document.getElementById('result-info').textContent=!filtered.length?L.noResults:L.showing+' '+s+'\u2013'+e+' '+L.of+' '+filtered.length+' '+L.sources;}
  function escHtml(s){return s?s.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;'):'';}
  let st;document.getElementById('search-input').addEventListener('input',()=>{clearTimeout(st);st=setTimeout(applyFilters,300);});
  ['filter-case-study','filter-category','filter-relevance','filter-reliability'].forEach(id=>document.getElementById(id).addEventListener('change',applyFilters));
  document.querySelectorAll('#source-table th[data-sort]').forEach(th=>th.addEventListener('click',function(){const f=this.dataset.sort;if(sortField===f)sortDir*=-1;else{sortField=f;sortDir=f==='title'?1:-1;}sortSources();currentPage=1;renderTable();renderPagination();updateResultInfo();}));
})();
</script>
