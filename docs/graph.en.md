# Knowledge Graph

<style>
#graph-container {
  width: 100%; height: 700px; border: 1px solid var(--itks-border, #d4cdc0); border-radius: 8px;
  overflow: hidden; position: relative; background: var(--itks-surface, #ede7db);
}
#graph-controls {
  display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px; align-items: center;
}
#graph-controls button, #graph-controls select {
  padding: 6px 12px; border: 1px solid var(--itks-border, #d4cdc0); border-radius: 4px;
  background: var(--itks-surface, #ede7db); cursor: pointer; font-size: 13px;
  font-family: var(--itks-font-ui, sans-serif); color: var(--itks-text, #1a1a1a);
}
#graph-controls button:hover { background: var(--itks-hover, #e8e1d4); }
#graph-controls button.active { background: var(--itks-accent, #c23b22); color: #fff; border-color: var(--itks-accent, #c23b22); }
#graph-tooltip {
  position: absolute; top: 12px; right: 12px; width: 280px;
  background: rgba(245,240,232,0.97); border: 1px solid var(--itks-border, #d4cdc0); border-radius: 8px;
  padding: 14px; font-size: 13px; line-height: 1.5; display: none;
  box-shadow: 0 4px 16px rgba(42,37,32,0.12); max-height: 320px; overflow-y: auto;
  pointer-events: auto; z-index: 10; font-family: var(--itks-font-ui, sans-serif);
}
#graph-tooltip h3 { margin: 0 0 6px; font-size: 14px; font-family: var(--itks-font-heading, serif); }
#graph-tooltip .meta { color: var(--itks-text-sec, #5c5449); font-size: 12px; margin-bottom: 4px; }
#graph-tooltip .connections { margin-top: 8px; border-top: 1px solid var(--itks-border-lt, #e2dbd0); padding-top: 8px; }
#graph-tooltip .conn-item { font-size: 12px; padding: 2px 0; }
#graph-tooltip .close-btn {
  position: absolute; top: 6px; right: 10px; cursor: pointer;
  font-size: 16px; color: var(--itks-text-muted, #8a7e72); background: none; border: none; padding: 0;
}
#graph-legend {
  position: absolute; bottom: 12px; left: 12px;
  background: rgba(245,240,232,0.96); border: 1px solid var(--itks-border, #d4cdc0); border-radius: 8px;
  padding: 10px 14px; font-size: 12px; z-index: 10;
  max-height: 200px; overflow-y: auto; font-family: var(--itks-font-ui, sans-serif);
}
#graph-legend .legend-item { display: flex; align-items: center; gap: 6px; padding: 2px 0; cursor: pointer; }
#graph-legend .legend-dot { width: 10px; height: 10px; border-radius: 50%; flex-shrink: 0; }
#graph-legend .legend-item.dimmed { opacity: 0.3; }
#zoom-hint {
  position: absolute; bottom: 12px; right: 12px;
  background: rgba(245,240,232,0.92); border-radius: 6px; padding: 6px 10px;
  font-size: 11px; color: var(--itks-text-muted, #8a7e72); z-index: 10;
  font-family: var(--itks-font-ui, sans-serif);
}
.edge-supports { stroke: #4caf50; }
.edge-contradicts { stroke: #f44336; }
.edge-extends { stroke: #2196f3; }
.edge-shared_methodology { stroke: #ff9800; }
</style>

<div id="graph-controls">
  <button id="btn-zoom-in" title="Zoom in">+</button>
  <button id="btn-zoom-out" title="Zoom out">−</button>
  <button id="btn-reset" title="Reset view">Reset</button>
  <select id="filter-category"><option value="">All categories</option></select>
  <select id="load-case-data" title="Load graph data for a case study">
    <option value="iran" selected>Iran (default)</option>
    <option value="poland">Poland</option>
    <option value="indonesia">Indonesia</option>
    <option value="south-korea">South Korea</option>
    <option value="spain">Spain</option>
    <option value="tunisia">Tunisia</option>
    <option value="czech-republic">Czech Republic</option>
    <option value="chile">Chile</option>
  </select>
  <select id="filter-case"><option value="">All case studies</option></select>
  <label style="font-size:13px; display:flex; align-items:center; gap:4px;">
    <input type="checkbox" id="toggle-labels"> Labels
  </label>
  <label style="font-size:13px; display:flex; align-items:center; gap:4px;">
    <input type="checkbox" id="toggle-cluster"> Cluster by category
  </label>
</div>

<div id="graph-container">
  <div id="graph-tooltip">
    <button class="close-btn" onclick="document.getElementById('graph-tooltip').style.display='none'">&times;</button>
    <div id="tooltip-content"></div>
  </div>
  <div id="graph-legend"></div>
  <div id="zoom-hint">Scroll to zoom &middot; Drag to pan &middot; Click node for details</div>
</div>

<script src="https://d3js.org/d3.v7.min.js"></script>
<script>
(function() {
  const WIDTH_RATIO = 1;
  const HEIGHT = 700;

  // Lazy loading: start with Iran, fetch others on demand
  let currentCase = 'iran';
  let graphRendered = false;

  const isFA = !window.location.pathname.match(/\/en\//);
  const dataPrefix = isFA ? '../data/' : '../../data/';

  function loadGraphData(caseName) {
    const file = dataPrefix + 'graph_' + caseName + '.json';
    return fetch(file).then(r => r.json());
  }

  // Load Iran first (primary focus), then render
  loadGraphData('iran')
    .then(data => {
      // Progressive: show high-relevance nodes first
      const highRel = data.nodes.filter(n => !n.relevance || n.relevance >= 4 || n.type === 'category');
      const lowRel = data.nodes.filter(n => n.relevance && n.relevance < 4 && n.type !== 'category');
      const highIds = new Set(highRel.map(n => n.id));
      const highEdges = data.edges.filter(e => highIds.has(e.source) && highIds.has(e.target));

      renderGraph(highRel, highEdges);
      graphRendered = true;

      // After initial render, add remaining nodes
      if (lowRel.length > 0) {
        setTimeout(() => {
          renderGraph(data.nodes, data.edges);
        }, 500);
      }
    })
    .catch(() => {
      // Fallback to combined files
      fetch(dataPrefix + 'graph_nodes.json')
        .then(r => r.json())
        .then(nodes => {
          fetch(dataPrefix + 'graph_edges.json')
            .then(r => r.json())
            .then(edges => renderGraph(nodes, edges));
        });
    });

  function renderGraph(allNodes, allEdges) {
    const container = document.getElementById('graph-container');
    const width = container.clientWidth;

    // Prepare data
    const sourceNodes = allNodes.filter(n => n.type === 'source');
    const sourceIds = new Set(sourceNodes.map(n => n.id));
    const sourceEdges = allEdges.filter(e => sourceIds.has(e.source) && sourceIds.has(e.target));

    // Scales
    const caseStudies = [...new Set(sourceNodes.map(n => n.case_study))].sort();
    const categories = [...new Set(sourceNodes.map(n => n.category))].sort();
    const color = d3.scaleOrdinal(d3.schemeTableau10).domain(caseStudies);
    const radius = d => Math.max(3, Math.min(40, Math.sqrt(d.citations || 1) * 0.7));

    // Build adjacency for tooltip
    const adjacency = {};
    sourceEdges.forEach(e => {
      const s = typeof e.source === 'object' ? e.source.id : e.source;
      const t = typeof e.target === 'object' ? e.target.id : e.target;
      if (!adjacency[s]) adjacency[s] = [];
      if (!adjacency[t]) adjacency[t] = [];
      adjacency[s].push({ id: t, type: e.type, confidence: e.confidence });
      adjacency[t].push({ id: s, type: e.type, confidence: e.confidence });
    });

    // Populate filter dropdowns
    const catSelect = document.getElementById('filter-category');
    categories.forEach(c => {
      const opt = document.createElement('option');
      opt.value = c; opt.textContent = c;
      catSelect.appendChild(opt);
    });
    const caseSelect = document.getElementById('filter-case');
    caseStudies.forEach(c => {
      const opt = document.createElement('option');
      opt.value = c; opt.textContent = c;
      caseSelect.appendChild(opt);
    });

    // SVG setup with zoom
    const svg = d3.select('#graph-container')
      .append('svg')
      .attr('width', width)
      .attr('height', HEIGHT);

    const g = svg.append('g');

    const zoom = d3.zoom()
      .scaleExtent([0.1, 8])
      .on('zoom', (e) => {
        g.attr('transform', e.transform);
        updateLabelsVisibility(e.transform.k);
      });
    svg.call(zoom);

    // Edge type colors
    const edgeColor = { supports: '#4caf50', contradicts: '#f44336', extends: '#2196f3', shared_methodology: '#ff9800' };

    // Draw edges
    const linkG = g.append('g');
    const link = linkG.selectAll('line')
      .data(sourceEdges)
      .join('line')
      .attr('stroke', d => edgeColor[d.type] || '#999')
      .attr('stroke-opacity', 0.15)
      .attr('stroke-width', d => Math.max(0.5, d.confidence * 1.5));

    // Draw nodes
    const nodeG = g.append('g');
    const node = nodeG.selectAll('circle')
      .data(sourceNodes)
      .join('circle')
      .attr('r', radius)
      .attr('fill', d => color(d.case_study))
      .attr('stroke', '#fff')
      .attr('stroke-width', 0.5)
      .attr('cursor', 'pointer')
      .call(d3.drag()
        .on('start', dragStart)
        .on('drag', dragged)
        .on('end', dragEnd));

    // Draw labels (hidden by default, shown on zoom or toggle)
    const labelG = g.append('g');
    const labels = labelG.selectAll('text')
      .data(sourceNodes)
      .join('text')
      .text(d => d.label.length > 30 ? d.label.slice(0, 28) + '...' : d.label)
      .attr('font-size', 4)
      .attr('dx', d => radius(d) + 2)
      .attr('dy', 1.5)
      .attr('fill', '#333')
      .attr('pointer-events', 'none')
      .attr('opacity', 0);

    // Node interactions
    let selectedNode = null;
    const nodeById = {};
    sourceNodes.forEach(n => nodeById[n.id] = n);

    node.on('click', (e, d) => {
      e.stopPropagation();
      selectedNode = d;
      showTooltip(d);
      highlightConnections(d);
    });

    node.on('mouseenter', (e, d) => {
      if (!selectedNode) highlightConnections(d);
    });

    node.on('mouseleave', (e, d) => {
      if (!selectedNode) resetHighlight();
    });

    svg.on('click', () => {
      selectedNode = null;
      document.getElementById('graph-tooltip').style.display = 'none';
      resetHighlight();
    });

    function highlightConnections(d) {
      const connected = new Set((adjacency[d.id] || []).map(c => c.id));
      connected.add(d.id);
      node.attr('opacity', n => connected.has(n.id) ? 1 : 0.08);
      link.attr('stroke-opacity', l => {
        const sid = typeof l.source === 'object' ? l.source.id : l.source;
        const tid = typeof l.target === 'object' ? l.target.id : l.target;
        return (sid === d.id || tid === d.id) ? 0.6 : 0.02;
      });
      labels.attr('opacity', n => connected.has(n.id) ? 1 : 0);
    }

    function resetHighlight() {
      const showLabels = document.getElementById('toggle-labels').checked;
      node.attr('opacity', d => d._filtered === false ? 0.05 : 1);
      link.attr('stroke-opacity', 0.15);
      labels.attr('opacity', d => {
        if (d._filtered === false) return 0;
        return showLabels ? 1 : 0;
      });
    }

    function showTooltip(d) {
      const tt = document.getElementById('graph-tooltip');
      const content = document.getElementById('tooltip-content');
      const conns = adjacency[d.id] || [];
      const edgeTypes = { supports: '🟢 supports', contradicts: '🔴 contradicts', extends: '🔵 extends', shared_methodology: '🟠 shared method' };

      let connHTML = '';
      if (conns.length > 0) {
        const grouped = {};
        conns.forEach(c => {
          const label = edgeTypes[c.type] || c.type;
          if (!grouped[label]) grouped[label] = [];
          const n = nodeById[c.id];
          if (n) grouped[label].push(n);
        });
        connHTML = '<div class="connections"><strong>Connections (' + conns.length + '):</strong>';
        for (const [type, items] of Object.entries(grouped)) {
          connHTML += '<div style="margin-top:4px;font-weight:600;font-size:11px;">' + type + ' (' + items.length + ')</div>';
          items.slice(0, 5).forEach(n => {
            connHTML += '<div class="conn-item">' + n.label.slice(0, 50) + ' <span style="color:#999">(' + n.case_study + ')</span></div>';
          });
          if (items.length > 5) connHTML += '<div class="conn-item" style="color:#999">...and ' + (items.length - 5) + ' more</div>';
        }
        connHTML += '</div>';
      }

      content.innerHTML =
        '<h3>' + d.label + '</h3>' +
        '<div class="meta">📍 ' + d.case_study + ' &middot; 📅 ' + d.year + '</div>' +
        '<div class="meta">📂 ' + d.category + '</div>' +
        '<div class="meta">📚 ' + (d.citations || 0).toLocaleString() + ' citations &middot; Relevance: ' + d.relevance + '/5</div>' +
        connHTML;
      tt.style.display = 'block';
    }

    // Simulation
    const simulation = d3.forceSimulation(sourceNodes)
      .force('link', d3.forceLink(sourceEdges).id(d => d.id).distance(40).strength(0.3))
      .force('charge', d3.forceManyBody().strength(-15))
      .force('center', d3.forceCenter(width / 2, HEIGHT / 2))
      .force('collision', d3.forceCollide().radius(d => radius(d) + 1))
      .force('x', d3.forceX(width / 2).strength(0.03))
      .force('y', d3.forceY(HEIGHT / 2).strength(0.03));

    simulation.on('tick', () => {
      link.attr('x1', d => d.source.x).attr('y1', d => d.source.y)
          .attr('x2', d => d.target.x).attr('y2', d => d.target.y);
      node.attr('cx', d => d.x).attr('cy', d => d.y);
      labels.attr('x', d => d.x).attr('y', d => d.y);
    });

    // Drag
    function dragStart(e, d) {
      if (!e.active) simulation.alphaTarget(0.3).restart();
      d.fx = d.x; d.fy = d.y;
    }
    function dragged(e, d) { d.fx = e.x; d.fy = e.y; }
    function dragEnd(e, d) {
      if (!e.active) simulation.alphaTarget(0);
      d.fx = null; d.fy = null;
    }

    // Label visibility based on zoom
    function updateLabelsVisibility(k) {
      if (document.getElementById('toggle-labels').checked) {
        labels.attr('opacity', d => d._filtered === false ? 0 : 1);
      } else {
        labels.attr('opacity', d => {
          if (d._filtered === false) return 0;
          return k > 2.5 ? 1 : 0;
        });
      }
    }

    // Legend
    const legendDiv = document.getElementById('graph-legend');
    let activeCases = new Set(caseStudies);
    caseStudies.forEach(cs => {
      const item = document.createElement('div');
      item.className = 'legend-item';
      item.innerHTML = '<span class="legend-dot" style="background:' + color(cs) + '"></span><span>' + cs + '</span>';
      item.addEventListener('click', () => {
        if (activeCases.has(cs)) { activeCases.delete(cs); item.classList.add('dimmed'); }
        else { activeCases.add(cs); item.classList.remove('dimmed'); }
        applyFilters();
      });
      legendDiv.appendChild(item);
    });

    // Edge type legend
    const edgeLegend = document.createElement('div');
    edgeLegend.style.cssText = 'margin-top:8px;padding-top:6px;border-top:1px solid #eee;font-size:11px;';
    edgeLegend.innerHTML = '<div style="font-weight:600;margin-bottom:3px;">Edge types</div>' +
      '<div><span style="color:#4caf50">━</span> supports</div>' +
      '<div><span style="color:#f44336">━</span> contradicts</div>' +
      '<div><span style="color:#2196f3">━</span> extends</div>' +
      '<div><span style="color:#ff9800">━</span> shared methodology</div>';
    legendDiv.appendChild(edgeLegend);

    // Filtering
    function applyFilters() {
      const catFilter = catSelect.value;
      const caseFilter = caseSelect.value;

      sourceNodes.forEach(n => {
        const catOk = !catFilter || n.category === catFilter;
        const caseOk = !caseFilter || n.case_study === caseFilter;
        const legendOk = activeCases.has(n.case_study);
        n._filtered = catOk && caseOk && legendOk;
      });

      const visibleIds = new Set(sourceNodes.filter(n => n._filtered !== false).map(n => n.id));
      node.attr('opacity', d => d._filtered === false ? 0.05 : 1);
      link.attr('stroke-opacity', l => {
        const sid = typeof l.source === 'object' ? l.source.id : l.source;
        const tid = typeof l.target === 'object' ? l.target.id : l.target;
        return (visibleIds.has(sid) && visibleIds.has(tid)) ? 0.15 : 0.01;
      });
      labels.attr('opacity', d => d._filtered === false ? 0 : (document.getElementById('toggle-labels').checked ? 1 : 0));
    }

    catSelect.addEventListener('change', applyFilters);
    caseSelect.addEventListener('change', applyFilters);

    // Controls
    document.getElementById('btn-zoom-in').addEventListener('click', () => svg.transition().duration(300).call(zoom.scaleBy, 1.5));
    document.getElementById('btn-zoom-out').addEventListener('click', () => svg.transition().duration(300).call(zoom.scaleBy, 0.67));
    document.getElementById('btn-reset').addEventListener('click', () => {
      svg.transition().duration(500).call(zoom.transform, d3.zoomIdentity);
      catSelect.value = ''; caseSelect.value = '';
      activeCases = new Set(caseStudies);
      legendDiv.querySelectorAll('.legend-item').forEach(i => i.classList.remove('dimmed'));
      applyFilters();
      selectedNode = null;
      document.getElementById('graph-tooltip').style.display = 'none';
      resetHighlight();
    });

    document.getElementById('toggle-labels').addEventListener('change', (e) => {
      const show = e.target.checked;
      labels.attr('opacity', d => d._filtered === false ? 0 : (show ? 1 : 0));
    });

    // Load different case study data
    document.getElementById('load-case-data').addEventListener('change', (e) => {
      const caseName = e.target.value;
      loadGraphData(caseName).then(data => {
        // Clear existing SVG and re-render
        d3.select('#graph-container svg').remove();
        document.getElementById('graph-legend').innerHTML = '';
        document.getElementById('filter-category').innerHTML = '<option value="">All categories</option>';
        document.getElementById('filter-case').innerHTML = '<option value="">All case studies</option>';
        renderGraph(data.nodes, data.edges);
      }).catch(err => {
        console.error('Failed to load graph for ' + caseName + ':', err);
      });
    });

    // Cluster by category
    document.getElementById('toggle-cluster').addEventListener('change', (e) => {
      if (e.target.checked) {
        const catList = [...new Set(sourceNodes.map(n => n.category))].sort();
        const cols = Math.ceil(Math.sqrt(catList.length));
        const cellW = width / cols;
        const cellH = HEIGHT / Math.ceil(catList.length / cols);
        const catPos = {};
        catList.forEach((c, i) => {
          catPos[c] = { x: (i % cols) * cellW + cellW / 2, y: Math.floor(i / cols) * cellH + cellH / 2 };
        });
        simulation
          .force('x', d3.forceX(d => catPos[d.category]?.x || width / 2).strength(0.4))
          .force('y', d3.forceY(d => catPos[d.category]?.y || HEIGHT / 2).strength(0.4))
          .alpha(0.8).restart();
      } else {
        simulation
          .force('x', d3.forceX(width / 2).strength(0.03))
          .force('y', d3.forceY(HEIGHT / 2).strength(0.03))
          .alpha(0.8).restart();
      }
    });
  }
})();
</script>

*Nodes represent academic sources. Size reflects citation count. Color = case study. Edges show relationships (supports, contradicts, extends, shared methodology). Click a node for details. Scroll to zoom, drag to pan.*
