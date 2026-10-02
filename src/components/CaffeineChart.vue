<template>
  <div class="chart-card">
    <div class="header-section">
      <h2>Weekly Caffeine Intake</h2>

      <p class="subtitle">Daily caffeine consumption over one week</p>
    </div>

    <div class="legend-box">
      <div class="legend-title">Caffeine level</div>

      <div class="legend-items">
        <div class="legend-item">
          <span class="legend-color low"></span>
          <span>Low (0–80 mg)</span>
        </div>

        <div class="legend-item">
          <span class="legend-color moderate"></span>
          <span>Moderate (81–120 mg)</span>
        </div>

        <div class="legend-item">
          <span class="legend-color high"></span>
          <span>High (121–160 mg)</span>
        </div>

        <div class="legend-item">
          <span class="legend-color very-high"></span>
          <span>Very high (&gt;160 mg)</span>
        </div>
      </div>
    </div>

    <div class="chart-wrapper">
      <svg ref="chart"></svg>
    </div>

    <p class="interaction-hint">Hover over a bar to view details.</p>

    <div v-if="selectedData" class="tooltip-box">
      <div class="tooltip-day">
        {{ selectedData.day }}
      </div>

      <div class="tooltip-detail">
        <span>Caffeine</span>
        <strong>{{ selectedData.caffeine }} mg</strong>
      </div>

      <div class="tooltip-detail">
        <span>Drink</span>
        <strong>{{ selectedData.drink }}</strong>
      </div>
    </div>

    <p class="source">Data source: Sample weekly caffeine intake data.</p>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import * as d3 from 'd3';

const chart = ref(null);
const selectedData = ref(null);

const data = [
  { day: 'Mon', caffeine: 120, drink: 'Coffee' },
  { day: 'Tue', caffeine: 180, drink: 'Coffee + Matcha' },
  { day: 'Wed', caffeine: 80, drink: 'Matcha' },
  { day: 'Thu', caffeine: 200, drink: 'Coffee + Matcha' },
  { day: 'Fri', caffeine: 150, drink: 'Coffee' },
  { day: 'Sat', caffeine: 100, drink: 'Matcha' },
  { day: 'Sun', caffeine: 60, drink: 'Tea' },
];

function getBarColor(value) {
  if (value <= 80) return '#d9e4f2';
  if (value <= 120) return '#aac4e4';
  if (value <= 160) return '#7fa2dd';

  return '#5f86c2';
}

onMounted(() => {
  const width = 760;
  const height = 460;

  const margin = {
    top: 45,
    right: 30,
    bottom: 65,
    left: 75,
  };

  const svg = d3
    .select(chart.value)
    .attr('viewBox', `0 0 ${width} ${height}`)
    .attr('preserveAspectRatio', 'xMidYMid meet');

  const x = d3
    .scaleBand()
    .domain(data.map((d) => d.day))
    .range([margin.left, width - margin.right])
    .padding(0.28);

  const y = d3
    .scaleLinear()
    .domain([0, d3.max(data, (d) => d.caffeine)])
    .nice()
    .range([height - margin.bottom, margin.top]);

  // Draw bars
  svg
    .selectAll('.bar')
    .data(data)
    .join('rect')
    .attr('class', 'bar')
    .attr('x', (d) => x(d.day))
    .attr('y', (d) => y(d.caffeine))
    .attr('width', x.bandwidth())
    .attr('height', (d) => y(0) - y(d.caffeine))
    .attr('rx', 4)
    .attr('fill', (d) => getBarColor(d.caffeine))
    .style('cursor', 'pointer')

    .on('mouseover', function (event, d) {
      d3.select(this).transition().duration(120).attr('fill', '#f2aa3b');

      selectedData.value = d;
    })

    .on('mouseout', function (event, d) {
      d3.select(this)
        .transition()
        .duration(120)
        .attr('fill', getBarColor(d.caffeine));

      selectedData.value = null;
    });

  // Value labels
  svg
    .selectAll('.value-label')
    .data(data)
    .join('text')
    .attr('class', 'value-label')
    .attr('x', (d) => x(d.day) + x.bandwidth() / 2)
    .attr('y', (d) => y(d.caffeine) - 10)
    .attr('text-anchor', 'middle')
    .attr('font-size', 14)
    .attr('font-weight', 600)
    .attr('fill', '#222')
    .text((d) => `${d.caffeine} mg`);

  // X axis
  svg
    .append('g')
    .attr('transform', `translate(0,${height - margin.bottom})`)
    .call(d3.axisBottom(x))
    .selectAll('text')
    .attr('font-size', 13);

  // Y axis
  svg
    .append('g')
    .attr('transform', `translate(${margin.left},0)`)
    .call(d3.axisLeft(y).ticks(6))
    .selectAll('text')
    .attr('font-size', 12);

  // Y axis label
  svg
    .append('text')
    .attr('transform', 'rotate(-90)')
    .attr('x', -height / 2)
    .attr('y', 22)
    .attr('text-anchor', 'middle')
    .attr('font-size', 14)
    .attr('font-weight', 600)
    .text('Caffeine (mg)');

  // X axis label
  svg
    .append('text')
    .attr('x', width / 2)
    .attr('y', height - 14)
    .attr('text-anchor', 'middle')
    .attr('font-size', 14)
    .attr('font-weight', 600)
    .text('Day of Week');
});
</script>

<style scoped>
.chart-card {
  max-width: 900px;
  margin: 36px auto;
  padding: 28px 30px 24px;
  background: #ffffff;
  border-radius: 18px;
  box-shadow: 0 4px 18px rgba(0, 0, 0, 0.08);
}

.header-section {
  margin-bottom: 18px;
  text-align: center;
}

h2 {
  margin: 0;
  font-size: 28px;
  font-weight: 700;
  color: #222;
}

.subtitle {
  margin: 6px 0 0;
  font-size: 15px;
  color: #666;
}

.legend-box {
  margin: 0 auto 18px;
  padding: 10px 14px;
  background: #fafafa;
  border: 1px solid #dedede;
  border-radius: 10px;
}

.legend-title {
  margin-bottom: 8px;
  text-align: center;
  font-size: 14px;
  font-weight: 700;
  color: #333;
}

.legend-items {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 8px 18px;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 12px;
  color: #444;
  white-space: nowrap;
}

.legend-color {
  display: inline-block;
  width: 14px;
  height: 14px;
  border: 1px solid #999;
  border-radius: 3px;
  flex-shrink: 0;
}

.low {
  background: #d9e4f2;
}

.moderate {
  background: #aac4e4;
}

.high {
  background: #7fa2dd;
}

.very-high {
  background: #5f86c2;
}

.chart-wrapper {
  width: 100%;
}

svg {
  display: block;
  width: 100%;
  height: auto;
}

.interaction-hint {
  margin: 12px 0 0;
  text-align: center;
  font-size: 13px;
  color: #777;
}

.tooltip-box {
  width: min(100%, 320px);
  margin: 14px auto 0;
  padding: 12px 16px;
  background: #f5f6f8;
  border: 1px solid #e2e4e8;
  border-radius: 10px;
  text-align: left;
}

.tooltip-day {
  margin-bottom: 8px;
  text-align: center;
  font-size: 18px;
  font-weight: 700;
  color: #222;
}

.tooltip-detail {
  display: flex;
  justify-content: space-between;
  gap: 16px;
  padding: 3px 0;
  font-size: 14px;
}

.tooltip-detail span {
  color: #666;
}

.tooltip-detail strong {
  color: #222;
  text-align: right;
}

.source {
  margin: 18px 0 0;
  text-align: center;
  font-size: 12px;
  color: #888;
}

@media (max-width: 640px) {
  .chart-card {
    margin: 16px;
    padding: 20px 16px;
  }

  h2 {
    font-size: 24px;
  }

  .subtitle {
    font-size: 14px;
  }

  .legend-items {
    justify-content: flex-start;
  }

  .legend-item {
    font-size: 11px;
  }
}
</style>
