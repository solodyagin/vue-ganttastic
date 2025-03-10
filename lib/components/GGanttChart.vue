<template>
  <div class="g-gantt-chart-container" :data-theme="theme" :style="{ width, height }">
    <div class="g-gantt-chart">
      <g-gantt-timeaxis
        v-if="!hideTimeaxis"
        :chart-start="chartStart"
        :chart-end="chartEnd"
        :row-label-width="rowLabelWidth"
        :timemarker-offset="timemarkerOffset"
        :locale="locale"
        :precision="precision"
        :time-format="timeFormat"
        :time-count="timeCount"
        :grid-size="gridSize"
        :day-format="dayFormat"
        :month-format="monthFormat"
      />

      <div class="g-gantt-rows-container" :style="{ width: `${timeCount * gridSize + rowLabelWidth}px` }">
        <g-gantt-grid
          v-if="grid"
          :chart-start="chartStart"
          :chart-end="chartEnd"
          :row-label-width="rowLabelWidth"
          :highlighted-hours="highlightedHours"
          :highlighted-days="highlightedDays"
          :precision="precision"
          :time-count="timeCount"
          :grid-size="gridSize"
        />
        <slot />
      </div>
    </div>
  </div>
</template>

<script>
import moment from 'moment'
import GGanttTimeaxis from './GGanttTimeaxis.vue'
import GGanttGrid from './GGanttGrid.vue'
import GGanttRow from './GGanttRow.vue'
import GGanttBar from './GGanttBar.vue'

export default {
  name: 'GGanttChart',

  components: {
    GGanttTimeaxis,
    GGanttGrid
  },

  props: {
    chartStart: { type: String, required: true },
    chartEnd: { type: String, required: true },
    hideTimeaxis: { type: Boolean, default: false },
    rowLabelWidth: { type: Number, default: 200 },
    rowHeight: { type: Number, default: 40 },
    locale: { type: String, default: 'en' },
    theme: { type: String },
    grid: { type: Boolean, default: false },
    gridSize: { type: Number, default: 30 },
    highlightedHours: { type: Array, default: () => [] },
    highlightedDays: { type: Array, default: () => [] }, // format YYYY-MM-DD
    width: { type: String, default: '100%' }, // the total width of the entire ganttastic component in %
    height: { type: String, default: '100%' },
    pushOnOverlap: { type: Boolean },
    isMagnetic: { type: Boolean },
    snapBackOnOverlap: { type: Boolean },
    minGapBetweenBars: { type: Number, default: 0 },
    defaultBarLength: { type: Number, default: 1 },
    precision: { type: String, default: 'month' }, // 'month', 'day'
    barConfigKey: { type: String, default: 'ganttBarConfig' },
    barStartKey: { type: String, default: 'start' }, // property name of the bar objects that represents the start datetime
    barEndKey: { type: String, default: 'end' }, // property name of the bar objects that represents the end datetime
    allowAdd: { type: Boolean, default: true },
    dayFormat: { type: String, default: 'ddd DD MMMM' },
    monthFormat: { type: String, default: 'MMMM YYYY' },
    tooltipFormat: {
      type: String,
      default: '{start} - {end} duration: {duration}'
    }
  },

  data() {
    return {
      timemarkerOffset: 0,
      movedBarsInDrag: new Set()
    }
  },

  computed: {
    timeUnit() {
      return this.precision === 'month' ? 'days' : 'hours'
    },

    timeFormat() {
      return this.precision === 'month' ? 'YYYY-MM-DD HH' : 'YYYY-MM-DD HH:mm'
    },

    timeCount() {
      let momentChartStart = moment(this.chartStart)
      let momentChartEnd = moment(this.chartEnd)
      return Math.floor(momentChartEnd.diff(momentChartStart, this.timeUnit, true))
    }
  },

  methods: {
    getGanttBarChildrenList() {
      let ganttBarChildren = []
      let ganttRowChildrenList = this.$children.filter(childComp => childComp.$options.name === GGanttRow.name)
      ganttRowChildrenList.forEach(row => {
        let ganttBarChildrenOfRow = row.$children.filter(childComp => childComp.$options.name === GGanttBar.name)
        ganttBarChildren.push(...ganttBarChildrenOfRow)
      })
      return ganttBarChildren
    },

    getBarsFromBundle(bundleId) {
      if (bundleId === undefined || bundleId === null) {
        return []
      }
      return this.getGanttBarChildrenList().filter(ganttBarChild => ganttBarChild.barConfig.bundle === bundleId)
    },

    initDragOfBarsFromBundle(gGanttBar, e) {
      gGanttBar.initDrag(e)
      this.movedBarsInDrag.add(gGanttBar.bar)
      if (gGanttBar.barConfig.bundle !== null && gGanttBar.barConfig.bundle !== undefined) {
        this.getGanttBarChildrenList().forEach(ganttBarChild => {
          if (ganttBarChild.barConfig.bundle === gGanttBar.barConfig.bundle && ganttBarChild !== gGanttBar) {
            ganttBarChild.initDrag(e)
            this.movedBarsInDrag.add(ganttBarChild.bar)
          }
        })
      }
    },

    moveBarsFromBundleOfPushedBar(pushedBar, minuteDiff, overlapType) {
      this.movedBarsInDrag.add(pushedBar)
      let bundleId = pushedBar[this.barConfigKey] ? pushedBar[this.barConfigKey].bundle : null
      if (bundleId === undefined || bundleId === null) {
        return
      }
      this.getGanttBarChildrenList().forEach(ganttBarChild => {
        if (ganttBarChild.barConfig.bundle === bundleId && ganttBarChild.bar !== pushedBar) {
          ganttBarChild.moveBarByChildPointsAndPush(minuteDiff, overlapType)
          this.movedBarsInDrag.add(ganttBarChild.bar)
        }
      })
    },

    shouldSnapBackBar(ganttBar) {
      if (this.snapBackOnOverlap && ganttBar.barConfig.pushOnOverlap !== false) {
        let { overlapBar } = ganttBar.getOverlapBarAndType(ganttBar.bar)
        return !!overlapBar
      }
      return false
    },

    snapBackBundleIfNeeded(ganttBar) {
      let barsFromBundle = this.getBarsFromBundle(ganttBar.barConfig.bundle)
      if (this.shouldSnapBackBar(ganttBar) || barsFromBundle.some(gBar => this.shouldSnapBackBar(gBar))) {
        ganttBar.snapBack()
        barsFromBundle.forEach(gBar => gBar.snapBack())
        return true
      }
      return false
    },

    onBarEvent({ event, type, time }, ganttBar) {
      this.$emit(`${type}-bar`, { event, bar: ganttBar.bar, time })
    },

    onDragendBar(e, ganttBar, action) {
      let didSnapBack = this.snapBackBundleIfNeeded(ganttBar)
      let movedBars = didSnapBack ? new Set() : this.movedBarsInDrag
      // Magnetic suction
      if (movedBars.size && this.isMagnetic) {
        let { left, right /*, move*/ } = action

        movedBars.forEach(bar => {
          if (this.precision === 'month') {
            if (left && bar == ganttBar.bar) {
              if (moment(bar[this.barStartKey]).hours() < 12) {
                bar[this.barStartKey] = moment(bar[this.barStartKey]).hours(0).format()
              } else {
                bar[this.barStartKey] = moment(bar[this.barStartKey]).hours(24).format()
              }
            } else if (right && bar == ganttBar.bar) {
              if (moment(bar[this.barEndKey]).hours() < 12) {
                bar[this.barEndKey] = moment(bar[this.barEndKey]).hours(0).format()
              } else {
                bar[this.barEndKey] = moment(bar[this.barEndKey]).hours(24).format()
              }
            } else {
              if (moment(bar[this.barStartKey]).hours() < 12) {
                bar[this.barStartKey] = moment(bar[this.barStartKey]).hours(0).format()
                bar[this.barEndKey] = moment(bar[this.barEndKey]).hours(0).format()
              } else {
                bar[this.barStartKey] = moment(bar[this.barStartKey]).hours(24).format()
                bar[this.barEndKey] = moment(bar[this.barEndKey]).hours(24).format()
              }
            }
          } else {
            if (left && bar == ganttBar.bar) {
              if (moment(bar[this.barStartKey]).minutes() < 30) {
                bar[this.barStartKey] = moment(bar[this.barStartKey]).minutes(0).format()
              } else {
                bar[this.barStartKey] = moment(bar[this.barStartKey]).minutes(60).format()
              }
            } else if (right && bar == ganttBar.bar) {
              if (moment(bar[this.barEndKey]).minutes() < 30) {
                bar[this.barEndKey] = moment(bar[this.barEndKey]).minutes(0).format()
              } else {
                bar[this.barEndKey] = moment(bar[this.barEndKey]).minutes(60).format()
              }
            } else {
              if (moment(bar[this.barStartKey]).minutes() < 30) {
                bar[this.barStartKey] = moment(bar[this.barStartKey]).minutes(0).format()
                bar[this.barEndKey] = moment(bar[this.barEndKey]).minutes(0).format()
              } else {
                bar[this.barStartKey] = moment(bar[this.barStartKey]).minutes(60).format()
                bar[this.barEndKey] = moment(bar[this.barEndKey]).minutes(60).format()
              }
            }
          }
        })
      }
      this.movedBarsInDrag = new Set()
      this.$emit('dragend-bar', { event: e, bar: ganttBar.bar, movedBars })
    },

    // ------------------------------------------------------------------------
    // --------  METHODS FOR SETTING THE DRAG LIMIT OF A BAR   ----------------
    // ------------------------------------------------------------------------

    // how far you can drag a bar depends on the position of the closest immobile bar
    // note that if a bar from the same row belongs to a bundle
    // other rows might need to be taken into consideration, too
    setDragLimitsOfGanttBar(bar) {
      if (!this.pushOnOverlap || bar.barConfig.pushOnOverlap === false) {
        return
      }
      for (let side of ['left', 'right']) {
        let [totalGapDistance, bundleBarsOnPath] = this.countGapDistanceToNextImmobileBar(bar, null, side, false)
        for (let i = 0; i < bundleBarsOnPath.length; i++) {
          let barFromBundle = bundleBarsOnPath[i].bar
          let gapDist = bundleBarsOnPath[i].gapDistance
          let otherBarsFromBundle = this.getBarsFromBundle(barFromBundle.barConfig.bundle).filter(
            otherBar => otherBar !== barFromBundle
          )
          otherBarsFromBundle.forEach(otherBar => {
            let [newGapDistance, newBundleBars] = this.countGapDistanceToNextImmobileBar(otherBar, gapDist, side)
            if (newGapDistance !== null && (newGapDistance < totalGapDistance || !totalGapDistance)) {
              totalGapDistance = newGapDistance
            }
            newBundleBars.forEach(newBundleBar => {
              if (!bundleBarsOnPath.find(barAndGap => barAndGap.bar === newBundleBar.bar)) {
                bundleBarsOnPath.push(newBundleBar)
              }
            })
          })
        }
        if (totalGapDistance != null && side === 'left') {
          bar.dragLimitLeft = bar.$refs['g-bar'].offsetLeft - totalGapDistance
        } else if (totalGapDistance != null && side === 'right') {
          bar.dragLimitRight = bar.$refs['g-bar'].offsetLeft + bar.$refs['g-bar'].offsetWidth + totalGapDistance
        }
      }
      // all bars from the bundle of the clicked bar need to have the same drag limit:
      let barsFromBundleOfClickedBar = this.getBarsFromBundle(bar.barConfig.bundle)
      barsFromBundleOfClickedBar.forEach(barFromBundle => {
        barFromBundle.dragLimitLeft = bar.dragLimitLeft
        barFromBundle.dragLimitRight = bar.dragLimitRight
      })
    },

    // returns the gap distance to the next immobile bar
    // in the row where the given bar (parameter) is (added to gapDistanceSoFar)
    // and a list of all bars on that path that belong to a bundle
    countGapDistanceToNextImmobileBar(bar, gapDistanceSoFar, side = 'left', ignoreShadows = true) {
      let bundleBarsAndGapDist = bar.barConfig.bundle ? [{ bar, gapDistance: gapDistanceSoFar }] : []
      let currentBar = bar
      let nextBar = this.getNextGanttBar(currentBar, side)
      // left side:
      if (side === 'left') {
        while (nextBar) {
          let nextBarOffsetRight = nextBar.$refs['g-bar'].offsetLeft + nextBar.$refs['g-bar'].offsetWidth
          gapDistanceSoFar += currentBar.$refs['g-bar'].offsetLeft - nextBarOffsetRight
          if (nextBar.barConfig.immobile || (nextBar.barConfig.isShadow && !ignoreShadows)) {
            return [gapDistanceSoFar, bundleBarsAndGapDist]
          } else if (nextBar.barConfig.bundle) {
            bundleBarsAndGapDist.push({
              bar: nextBar,
              gapDistance: gapDistanceSoFar
            })
          }
          currentBar = nextBar
          nextBar = this.getNextGanttBar(nextBar, 'left')
        }
      }
      if (side === 'right') {
        while (nextBar) {
          let currentBarOffsetRight = currentBar.$refs['g-bar'].offsetLeft + currentBar.$refs['g-bar'].offsetWidth
          gapDistanceSoFar += nextBar.$refs['g-bar'].offsetLeft - currentBarOffsetRight
          if (nextBar.barConfig.immobile || (nextBar.barConfig.isShadow && !ignoreShadows)) {
            return [gapDistanceSoFar, bundleBarsAndGapDist]
          } else if (nextBar.barConfig.bundle) {
            bundleBarsAndGapDist.push({
              bar: nextBar,
              gapDistance: gapDistanceSoFar
            })
          }
          currentBar = nextBar
          nextBar = this.getNextGanttBar(nextBar, 'right')
        }
      }
      return [null, bundleBarsAndGapDist]
    },

    getNextGanttBar(bar, side = 'left') {
      let allBarsLeftOrRight = []
      if (side === 'left') {
        allBarsLeftOrRight = bar.$parent.$children.filter(gBar => {
          return (
            gBar.$options.name === GGanttBar.name &&
            gBar.$parent === bar.$parent &&
            gBar.$refs['g-bar'] &&
            gBar.$refs['g-bar'].offsetLeft < bar.$refs['g-bar'].offsetLeft &&
            gBar.barConfig.pushOnOverlap !== false
          )
        })
      } else {
        allBarsLeftOrRight = bar.$parent.$children.filter(gBar => {
          return (
            gBar.$options.name === GGanttBar.name &&
            gBar.$parent === bar.$parent &&
            gBar.$refs['g-bar'] &&
            gBar.$refs['g-bar'].offsetLeft > bar.$refs['g-bar'].offsetLeft &&
            gBar.barConfig.pushOnOverlap !== false
          )
        })
      }
      if (allBarsLeftOrRight.length > 0) {
        return allBarsLeftOrRight.reduce((bar1, bar2) => {
          let bar1Dist = Math.abs(bar1.$refs['g-bar'].offsetLeft - bar.$refs['g-bar'].offsetLeft)
          let bar2Dist = Math.abs(bar2.$refs['g-bar'].offsetLeft - bar.$refs['g-bar'].offsetLeft)
          return bar1Dist < bar2Dist ? bar1 : bar2
        }, allBarsLeftOrRight[0])
      } else {
        return null
      }
    }

    // ------------------------------------------------------------------------
    // ------------------------------------------------------------------------
    // ------------------------------------------------------------------------
  },

  // all child components of GGanttChart may have access to
  // the following values by using Vue's "inject" option:
  provide() {
    return {
      getTimeCount: () => this.timeCount,
      getChartProps: () => this.$props,
      initDragOfBarsFromBundle: (bundleId, e) => this.initDragOfBarsFromBundle(bundleId, e),
      moveBarsFromBundleOfPushedBar: (bar, minuteDiff, overlapType) =>
        this.moveBarsFromBundleOfPushedBar(bar, minuteDiff, overlapType),
      setDragLimitsOfGanttBar: ganttBar => this.setDragLimitsOfGanttBar(ganttBar),
      onBarEvent: (e, ganttBar) => this.onBarEvent(e, ganttBar),
      onDragendBar: (e, ganttBar, action) => this.onDragendBar(e, ganttBar, action),
      getTimeUnit: () => this.timeUnit,
      getTimeFormat: () => this.timeFormat
    }
  }
}
</script>
