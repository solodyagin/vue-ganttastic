<template>
  <div>
    <select v-model="theme">
      <option v-for="option in themes" :key="option" :value="option">
        {{ option }}
      </option>
    </select>

    <h2>Chart #1</h2>

    <g-gantt-chart
      :chart-start="chart1.chartStart"
      :chart-end="chart1.chartEnd"
      :grid="chart1.grid"
      :grid-size="chart1.gridSize"
      day-format="dddd, DD. MMMM"
      :hide-timeaxis="chart1.hideTimeaxis"
      :push-on-overlap="chart1.pushOnOverlap"
      snap-back-on-overlap
      :precision="chart1.precision"
      :is-magnetic="chart1.isMagnetic"
      :highlighted-hours="chart1.highlightedHours"
      :highlighted-days="chart1.highlightedDays"
      :row-label-width="chart1.rowLabelWidth"
      :row-height="chart1.rowHeight"
      :theme="theme"
      bar-config-key="config"
      bar-start-key="myStart"
      bar-end-key="myEnd"
      @dragend-bar="onDragend($event)"
    >
      <g-gantt-row
        v-for="row in chart1.rows"
        :key="row.label"
        :label="row.label"
        :label-style="row.labelStyle"
        :row-style="row.style"
        :bars="row.bars"
        :highlight-on-hover="chart1.highlightOnHover"
      >
        <template #bar-label="{ bar }">
          <span>{{ bar.label }}</span>
        </template>
      </g-gantt-row>
    </g-gantt-chart>

    <h2>Chart #2</h2>

    <g-gantt-chart
      :chart-start="chart2.chartStart"
      :chart-end="chart2.chartEnd"
      :grid="chart2.grid"
      :grid-size="chart2.gridSize"
      :hide-timeaxis="chart2.hideTimeaxis"
      :push-on-overlap="chart2.pushOnOverlap"
      snap-back-on-overlap
      :precision="chart2.precision"
      :is-magnetic="chart2.isMagnetic"
      :highlighted-days="chart2.highlightedDays"
      :row-label-width="chart2.rowLabelWidth"
      :row-height="chart2.rowHeight"
      :theme="theme"
      :width="chart2.width"
      :height="chart2.height"
      :allow-add="chart2.allowAdd"
    >
      <g-gantt-row
        v-for="row in chart2.rows"
        :key="row.label"
        :label="row.label"
        :row-style="row.style"
        :bars="row.bars"
        :highlight-on-hover="chart2.highlightOnHover"
      >
        <template #bar-label="{ bar }">
          <span>{{ bar.label }}</span>
        </template>
      </g-gantt-row>
    </g-gantt-chart>
  </div>
</template>

<script>
export default {
  data: () => ({
    theme: 'default',
    themes: [
      'creamy',
      'crimson',
      'dark',
      'default',
      'flare',
      'fuchsia',
      'grove',
      'material-blue',
      'sky',
      'slumber',
      'vue'
    ],
    chart1: {
      chartStart: '2020-03-02 00:00',
      chartEnd: '2020-03-10 10:00',
      precision: 'day',
      pushOnOverlap: true,
      isMagnetic: true,
      grid: true,
      gridSize: 30,
      rowHeight: 40,
      rowLabelWidth: 200,
      hideTimeaxis: false,
      highlightOnHover: true,
      highlightedDays: ['2020-03-08'],
      highlightedHours: [10, 12],
      rows: [
        {
          label: 'Row #1',
          bars: [
            {
              myStart: '2020-03-03 18:00',
              myEnd: '2020-03-03 23:00',
              label: 'Immobile',
              tooltip: 'Bar tooltip',
              config: {
                color: 'white',
                backgroundColor: '#404040',
                opacity: 0.5,
                immobile: true
              }
            },
            {
              myStart: '2020-03-03 04:00',
              myEnd: '2020-03-03 15:00',
              label: 'Bar',
              tooltip: 'Bar tooltip',
              config: {
                color: 'white',
                backgroundColor: '#2e74a3',
                bundle: 'blueBundle'
              }
            }
          ]
        },
        {
          label: 'Row #2',
          labelStyle: {
            justifyContent: 'end'
          },
          style: {
            background: '#ffb0b07f'
          },
          bars: [
            {
              myStart: '2020-03-02 09:00',
              myEnd: '2020-03-02 18:00',
              label: 'Bar',
              tooltip: 'Bar tooltip',
              config: {
                color: 'white',
                backgroundColor: '#de3b26',
                bundle: 'redBundle'
              }
            },
            {
              myStart: '2020-03-03 04:00',
              myEnd: '2020-03-03 15:00',
              label: 'We belong together ^',
              tooltip: 'Bar tooltip',
              config: {
                color: 'white',
                backgroundColor: '#2e74a3',
                bundle: 'blueBundle'
              }
            },
            {
              myStart: '2020-03-03 18:00',
              myEnd: '2020-03-03 22:00',
              label: 'Bar',
              tooltip: 'Bar tooltip',
              config: { color: 'white', backgroundColor: '#aa34a3' }
            }
          ]
        },
        {
          label: 'Row #3',
          bars: [
            {
              myStart: '2020-03-02 09:00',
              myEnd: '2020-03-02 18:00',
              label: 'We belong together ^',
              tooltip: 'Bar tooltip',
              config: {
                color: 'white',
                backgroundColor: '#de3b26',
                bundle: 'redBundle'
              }
            },
            {
              myStart: '2020-03-02 22:30',
              myEnd: '2020-03-03 05:00',
              label: 'With handles!',
              tooltip: 'Bar tooltip',
              config: {
                color: 'white',
                backgroundColor: '#a23def',
                handles: true
              }
            },
            {
              myStart: '2020-03-02 01:00',
              myEnd: '2020-03-02 07:00',
              label: 'Bar',
              tooltip: 'Bar tooltip',
              config: {
                color: 'white',
                backgroundColor: '#5effad',
                pushOnOverlap: false,
                zIndex: 2
              }
            },
            {
              myStart: '2020-03-03 14:00',
              myEnd: '2020-03-03 20:00',
              label: 'Woooow!',
              tooltip: 'Bar tooltip',
              config: {
                color: 'white',
                background: 'repeating-linear-gradient(45deg, #de7359, #de7359 10px, #ffc803 10px, #ffc803 20px)'
              }
            }
          ]
        },
        {
          label: 'Row #4',
          bars: [
            {
              myStart: '2020-03-03 06:30',
              myEnd: '2020-03-03 20:00',
              label: 'Bar',
              tooltip: 'Bar tooltip',
              config: {
                color: 'white',
                backgroundColor: '#d18aaf',
                handles: true
              }
            },
            {
              myStart: '2020-03-02 00:30',
              myEnd: '2020-03-03 01:00',
              label: 'Rectangular',
              tooltip: 'Bar tooltip',
              config: {
                color: 'white',
                backgroundColor: '#f2840f',
                borderRadius: 0
              }
            }
          ]
        }
      ]
    },
    chart2: {
      chartStart: '2020-03-01 00:00',
      chartEnd: '2020-04-01 00:00',
      precision: 'month',
      pushOnOverlap: false,
      isMagnetic: true,
      grid: true,
      gridSize: 50,
      rowHeight: 40,
      rowLabelWidth: 300,
      hideTimeaxis: false,
      highlightOnHover: true,
      highlightedDays: ['2020-03-01', '2020-03-08', '2020-03-15', '2020-03-22', '2020-03-29'],
      width: '90vw',
      height: '250px',
      allowAdd: false,
      rows: [
        {
          label: 'Row #1',
          bars: [
            {
              start: '2020-03-05 00:00',
              end: '2020-03-10 23:59',
              label: 'Bar',
              tooltip: 'Bar tooltip',
              ganttBarConfig: {
                color: 'white',
                backgroundColor: '#2e74a3',
                bundle: 'blueBundle'
              }
            }
          ]
        },
        {
          label: 'Row #2',
          bars: [
            {
              start: '2020-03-02 00:00',
              end: '2020-03-09 23:59',
              label: 'We belong together ^',
              tooltip: 'Bar tooltip',
              ganttBarConfig: {
                color: 'white',
                backgroundColor: '#2e74a3',
                bundle: 'blueBundle'
              }
            },
            {
              start: '2020-03-24 00:00',
              end: '2020-03-26 23:00',
              label: 'Bar',
              tooltip: 'Bar tooltip',
              ganttBarConfig: {
                color: 'white',
                backgroundColor: '#de3b26'
              }
            }
          ]
        },
        {
          label: 'Row #3',
          bars: [
            {
              start: '2020-03-15 00:00',
              end: '2020-03-18 23:59',
              label: 'Bar',
              tooltip: 'Bar tooltip',
              ganttBarConfig: {
                color: 'white',
                backgroundColor: '#408e2f'
              }
            }
          ],
          style: {
            background: 'linear-gradient(-45deg, rgba(0, 0, 0, 0) 48%, rgba(0, 0, 0, 0.2) 50%, rgba(0, 0, 0, 0) 52%)',
            backgroundSize: '1em 1em'
          }
        },
        {
          label:
            'Lorem ipsum dolor sit amet. Vel odit debitis qui aliquam sequi et reprehenderit Quis. Et ipsam enim aut culpa quia sed maiores veniam in consequuntur accusantium.',
          bars: []
        },
        {
          label: 'Row #5',
          bars: []
        },
        {
          label: 'Row #6',
          bars: []
        },
        {
          label: 'Row #7',
          bars: []
        },
        {
          label: 'Row #8',
          bars: []
        }
      ]
    }
  }),

  methods: {
    onDragend(e) {
      const { event, bar, movedBars } = e
      // eslint-disable-next-line
      console.log('onDragend', { event: event.type, bar, movedBars })
    }
  }
}
</script>

<style lang="scss" src="../lib/scss/index.scss"></style>
