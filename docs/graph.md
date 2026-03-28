# Knowledge Graph

<div id="graph-container" style="width: 100%; height: 600px; border: 1px solid #ddd; border-radius: 8px; overflow: hidden;"></div>

<script src="https://d3js.org/d3.v7.min.js"></script>
<script>
fetch('data/graph_nodes.json')
  .then(r => r.json())
  .then(nodes => {
    fetch('data/graph_edges.json')
      .then(r => r.json())
      .then(edges => renderGraph(nodes, edges));
  });

function renderGraph(nodes, edges) {
  const container = document.getElementById('graph-container');
  const width = container.clientWidth;
  const height = 600;

  const svg = d3.select('#graph-container')
    .append('svg')
    .attr('width', width)
    .attr('height', height);

  // Color by case study
  const caseStudies = [...new Set(nodes.filter(n => n.type === 'source').map(n => n.case_study))];
  const color = d3.scaleOrdinal(d3.schemeTableau10).domain(caseStudies);

  // Filter to source nodes only for the main viz
  const sourceNodes = nodes.filter(n => n.type === 'source');
  const sourceIds = new Set(sourceNodes.map(n => n.id));
  const sourceEdges = edges.filter(e => sourceIds.has(e.source) && sourceIds.has(e.target));

  const simulation = d3.forceSimulation(sourceNodes)
    .force('link', d3.forceLink(sourceEdges).id(d => d.id).distance(80))
    .force('charge', d3.forceManyBody().strength(-30))
    .force('center', d3.forceCenter(width / 2, height / 2))
    .force('collision', d3.forceCollide().radius(8));

  const link = svg.append('g')
    .selectAll('line')
    .data(sourceEdges)
    .join('line')
    .attr('stroke', '#999')
    .attr('stroke-opacity', 0.4)
    .attr('stroke-width', d => d.confidence * 2);

  const node = svg.append('g')
    .selectAll('circle')
    .data(sourceNodes)
    .join('circle')
    .attr('r', d => Math.max(4, Math.sqrt(d.citations || 1) * 2))
    .attr('fill', d => color(d.case_study))
    .attr('stroke', '#fff')
    .attr('stroke-width', 1)
    .call(d3.drag()
      .on('start', (e, d) => { if (!e.active) simulation.alphaTarget(0.3).restart(); d.fx = d.x; d.fy = d.y; })
      .on('drag', (e, d) => { d.fx = e.x; d.fy = e.y; })
      .on('end', (e, d) => { if (!e.active) simulation.alphaTarget(0); d.fx = null; d.fy = null; }));

  node.append('title').text(d => `${d.label} (${d.case_study}, ${d.year})`);

  // Legend
  const legend = svg.append('g').attr('transform', 'translate(20, 20)');
  caseStudies.forEach((cs, i) => {
    legend.append('circle').attr('cx', 0).attr('cy', i * 20).attr('r', 6).attr('fill', color(cs));
    legend.append('text').attr('x', 12).attr('y', i * 20 + 4).text(cs).style('font-size', '12px');
  });

  simulation.on('tick', () => {
    link.attr('x1', d => d.source.x).attr('y1', d => d.source.y)
        .attr('x2', d => d.target.x).attr('y2', d => d.target.y);
    node.attr('cx', d => d.x).attr('cy', d => d.y);
  });
}
</script>

*Nodes represent academic sources. Size reflects citation count. Color represents case study. Edges represent relationships (supports, contradicts, extends, shared methodology). Drag nodes to explore.*
