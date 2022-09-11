<script setup lang="ts">
import * as echarts from "echarts/core";
import {
    TitleComponent,
    TitleComponentOption,
    TooltipComponent,
    TooltipComponentOption,
} from "echarts/components";
import { GraphChart, GraphSeriesOption } from "echarts/charts";
import { CanvasRenderer } from "echarts/renderers";
import { ref, onMounted } from "vue";
import * as knowledge from "./api/knowledge.json";

echarts.use([TitleComponent, TooltipComponent, GraphChart, CanvasRenderer]);

type EChartsOption = echarts.ComposeOption<
    TitleComponentOption | TooltipComponentOption | GraphSeriesOption
>;

const chartDom = ref(null);
const options = {
    title: {
        text: "知识关系图谱",
    },
    tooltip: {},
    series: [
        {
            name: "知识关系图谱",
            type: "graph",
            layout: "force",
            roam: true,
            label: {
                show: true,
            },
            edgeSymbol: ["circle", "arrow"],
            edgeSymbolSize: [4, 10],
            edgeLabel: {
                fontSize: 20,
            },
            data: knowledge.data,
            links: knowledge.links,
        },
    ],
};

const drawChart = (option?: EChartsOption): void => {
    const myChart = echarts.init(chartDom.value as unknown as HTMLElement);
    option && myChart.setOption(option);
};

onMounted(() => drawChart(options as EChartsOption));
</script>

<template>
    <div ref="chartDom" id="chartDom"></div>
</template>

<style scoped lang="scss">
#chartDom {
    width: 100%;
    height: 100%;
    min-width: 700px;
    min-height: 700px;
}
</style>
