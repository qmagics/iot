<template>
  <div class="chart">
    <div class="chart-bar" ref="bar" v-loading="loading"></div>
  </div>
</template>

<script>
import echarts from "@/plugins/echarts";

export default {
  props: {
    //加载中标识
    loading: Boolean,
    //数据行
    rows: Array,
    //数据列
    columns: Array,
    //配置项
    settings: {}
  },

  data() {
    return {
      bar: null,

      barOption: {
        tooltip: {
          trigger: "axis",
          padding: this.$root.getPxNumberByRem("0.05rem"),
          textStyle: {
            fontSize: this.$root.getPxNumberByRem("0.14rem")
          },
          appendToBody: true
        },
        grid: {
          top: "25%",
          bottom: "10%",
          left: 0,
          right: 0,
          containLabel: true
        },
        legend: [
          //   {
          //     data: ["仅报警单位数", "仅故障单位数", "报警且故障单位数"],
          //     right: 0,
          //     top: 0,
          //     width: "40%",
          //     height: "15%",
          //     itemWidth: this.$root.getPxNumberByRem("0.16rem"),
          //     itemHeight: this.$root.getPxNumberByRem("0.08rem"),
          //     itemGap: this.$root.getPxNumberByRem("0.2rem"),
          //     textStyle: {
          //       fontSize: this.$root.getPxNumberByRem("0.12rem")
          //     },
          //     orient: "vertical"
          //   },
          //   {
          //     data: ["报警案件数", "故障案件数"],
          //     right: 0,
          //     top: "12%",
          //     width: "40%",
          //     height: "15%",
          //     itemWidth: this.$root.getPxNumberByRem("0.16rem"),
          //     itemHeight: this.$root.getPxNumberByRem("0.08rem"),
          //     itemGap: this.$root.getPxNumberByRem("0.2rem"),
          //     textStyle: {
          //       fontSize: this.$root.getPxNumberByRem("0.12rem")
          //     },
          //     orient: "vertical"
          //   }
        ],
        xAxis: [
          {
            type: "category",
            data: [],
            axisPointer: {
              type: "shadow"
            },
            axisLabel: {
              fontSize: this.$root.getPxNumberByRem("0.12rem")
            }
          }
        ],
        yAxis: [
          {
            type: "value",
            name: "单位数量",
            min: 0,
            // interval: 50,
            splitLine: {
              lineStyle: {
                type: "dashed"
              }
            },
            axisLabel: {
              formatter: "{value}",
              fontSize: this.$root.getPxNumberByRem("0.12rem")
            },
            nameTextStyle: {
              color: "rgb(109,151,191)",
              fontSize: this.$root.getPxNumberByRem("0.12rem")
            }
          }
        ],
        series: []
      }
    };
  },

  computed: {
    FooterBoxVisible() {
      return this.$store.state.FooterBoxVisible;
    }
  },

  watch: {
    FooterBoxVisible: {
      handler() {
        if (this.bar) {
          this.bar.resize();
        }
      },
      immediate: true
    },
    rows: {
      handler() {
        this.refresh();
      },
      deep: true
    },
    columns: {
      handler() {
        this.refresh();
      },
      deep: true
    },
    settings: {
      handler() {
        this.refresh();
      },
      deep: true
    }
  },

  methods: {
    refresh() {
      const series = [];

      this.columns.forEach(c => {
        const { label, prop, type, color } = c;

        const data = this.rows.map(i => i[prop]);

        let serie_item = {
          name: label,
          type,
          color:
            type == "bar"
              ? {
                  type: "linear",
                  x: 0,
                  y: 0,
                  x2: 0,
                  y2: 1,
                  colorStops: [
                    {
                      offset: 0,
                      color: color.toAlpha(1) // 0% 处的颜色
                    },
                    {
                      offset: 0.1,
                      color: color.toAlpha(0.4) // 0% 处的颜色
                    },
                    {
                      offset: 1,
                      color: color.toAlpha(0) // 100% 处的颜色
                    }
                  ],
                  global: false // 缺省为 false
                }
              : color,
          data,
          lineStyle: {
            width: this.$root.getPxNumberByRem("0.01rem")
          }
        };

        series.push(serie_item);
      });

      this.barOption.series = series;

      this.barOption.xAxis[0].data = this.rows.map(
        i => i[this.settings.dimension[0]]
      );

      if (this.bar) {
        this.bar.setOption(this.barOption);
        this.bar.resize();
      }
    },

    onWindowResize() {
      this.bar.resize();
    }
  },

  mounted() {
    this.bar = echarts.init(this.$refs.bar, "xiyu");
    this.refresh();
    window.addEventListener("resize", this.onWindowResize);
  },

  beforeDestroy() {
    window.removeEventListener("resize", this.onWindowResize);
  }
};
</script>

<style lang="scss" scoped>
.chart {
  .chart-bar {
    position: absolute;
    left: 0;
    right: 0;
    bottom: 0;
    top: 0;
  }
}
</style>