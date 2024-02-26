<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let data = [
    { year: 2020, exported: 0, produced: 300000 },
    { year: 2021, exported: 700000, produced: 200000 },
    { year: 2022, exported: 1000000, produced: 100000 }
  ];

  let tooltipContent, tooltipX, tooltipY, showTooltip = false;

  onMount(() => {
    const margin = { top: 20, right: 30, bottom: 40, left: 50 };
    const width = 700 - margin.left - margin.right;
    const height = 400 - margin.top - margin.bottom;

    const svg = d3.select('#chart')
                  .append('svg')
                  .attr('width', width + margin.left + margin.right)
                  .attr('height', height + margin.top + margin.bottom)
                  .append('g')
                  .attr('transform', 'translate(' + margin.left + ',' + margin.top + ')');

    const x = d3.scaleLinear()
                .domain(d3.extent(data, d => d.year))
                .range([0, width]);

    const y = d3.scaleLinear()
                .domain([0, d3.max(data, d => d.exported)])
                .range([height, 0]);

    svg.append('g')
       .attr('transform', 'translate(0,' + height + ')')
       .call(d3.axisBottom(x).ticks(data.length).tickFormat(d3.format('d')));

    svg.append('g')
       .call(d3.axisLeft(y));

    const lineExported = d3.line()
                           .x(d => x(d.year))
                           .y(d => y(d.exported));

    const lineProduced = d3.line()
                           .x(d => x(d.year))
                           .y(d => y(d.produced));

    svg.append('path')
       .datum(data)
       .attr('fill', 'none')
       .attr('stroke', 'brown')
       .attr('stroke-width', 1.5)
       .attr('d', lineExported);

    svg.append('path')
       .datum(data)
       .attr('fill', 'none')
       .attr('stroke', 'orange')
       .attr('stroke-width', 1.5)
       .attr('d', lineProduced);

    svg.selectAll('.dot')
       .data(data)
       .enter().append('circle')
       .attr('class', 'dot')
       .attr('cx', d => x(d.year))
       .attr('cy', d => y(d.exported))
       .attr('r', 5)
       .attr('fill', 'brown')
       .on('mouseover', (event, d) => {
         showTooltip = true;
         tooltipContent = `Year: ${d.year}<br>Exported: ${d.exported}<br>Produced: ${d.produced}`;
         tooltipX = event.pageX;
         tooltipY = event.pageY;
       })
       .on('mouseout', () => showTooltip = false);
  });
</script>

<div id="chart"></div>
{#if showTooltip}
  <div class="tooltip" style="left: {tooltipX}px; top: {tooltipY}px">
    {@html tooltipContent}
  </div>
{/if}

<style>
  .tooltip {
    position: absolute;
    background-color: #fff;
    padding: 10px;
    border: 1px solid #ddd;
    pointer-events: none;
  }
</style>
