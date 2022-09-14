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
import { ref, unref, shallowReactive, ShallowReactive, onMounted } from "vue";
import * as knowledge from "./api/knowledge.json";

echarts.use([TitleComponent, TooltipComponent, GraphChart, CanvasRenderer]);

type EChartsOption = echarts.ComposeOption<
    TitleComponentOption | TooltipComponentOption | GraphSeriesOption
>;

interface globalChartType {
    [x: string]: echarts.ECharts
}

const klg = shallowReactive(knowledge);
const val = ref<string>("");
const glb = shallowReactive<globalChartType>({});

const chartDom = ref(null);
const options: EChartsOption = {
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
            width: `100%`,
            height: `100%`,
            edgeSymbol: ["circle", "arrow"],
            edgeSymbolSize: [4, 10],
            edgeLabel: {
                fontSize: 20,
            },
            data: klg.data,
            links: klg.links,
        },
    ],
};

const drawChart = (option?: EChartsOption): void => {
    glb.myChart = echarts.init(chartDom.value as unknown as HTMLElement);
    option && glb.myChart.setOption(option);
};

onMounted(() => {
    drawChart(options as EChartsOption); 
});

const addData = ((dataSet: { name: string; }[], mount: ShallowReactive<globalChartType>, option: EChartsOption = options) => () => {
    const value = unref(val.value);
    dataSet.push({"name": value});
    mount.myChart.setOption(option);
    val.value = "";
})(klg.data, glb, options);
</script>

<template>
    <form action="." method="post" @submit.prevent>
        <label for="addData">添加数据</label><input type="text" id="addData" v-model="val">
        <button type="submit" @click="addData">添加数据</button>
    </form>
    <div ref="chartDom" id="chartDom"></div>
</template>

<style scoped lang="scss">
#chartDom {
    max-width: 100%;
    min-width: 700px;
    min-height: 700px;
    width: 100%;
    height: 100%;
}
</style>
