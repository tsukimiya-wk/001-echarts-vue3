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
    [x: string]: echarts.ECharts;
}

type dataSetType = ShallowReactive<{
    data: {
        name: string;
    }[];
    links: {
        source: string;
        target: string;
    }[];
}>;

const klg = shallowReactive(knowledge);
const val = ref<string>("");
const deps = ref<string>("");
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

const hasDataTarget = (dataSet: dataSetType, target: string = "") => {
    for (const data of dataSet.data) {
        if (data.name === target) {
            return true;
        }
    }

    return false;
}

const hasLinked = (dataSet: dataSetType, source: string, target: string) => {
    return dataSet.links.some((data) => data.source === source && data.target === target);
}

const addData = (
    (
        dataSet: dataSetType,
        mount: ShallowReactive<globalChartType>,
        option: EChartsOption = options,
    ) =>
    () => {
        const target = unref(val.value);
        if (!hasDataTarget(dataSet, target)) {
            dataSet.data.push({ name: target });
        }

        // \uFF0C 是中文的逗号
        const depsArr = unref(deps.value)
            .split(/[,\uFF0C\s+]/)
            .filter((e) => !!e);
        depsArr.forEach(
            ((dataSet: dataSetType, target: string) => (dep) => {
                if (!hasDataTarget(dataSet, dep)) {
                    dataSet.data.push({
                        name: dep,
                    });
                }

                if (!hasLinked(dataSet, target, dep)) {
                    dataSet.links.push({
                        source: target,
                        target: dep,
                    });
                }
            })(dataSet, target),
        );

        mount.myChart.setOption(option);
        val.value = "";
        deps.value = "";
    }
)(klg, glb, options);
</script>

<template>
    <form action="." method="post" @submit.prevent>
        <label for="addData">添加数据</label
        ><input type="text" id="addData" v-model="val" placeholder="添加数据" />
        <label for="addDeps">添加数据依赖</label
        ><input
            type="text"
            id="addDeps"
            v-model="deps"
            placeholder="添加数据的依赖项, 用中/英文逗号, 或是空格隔开各个依赖" />
        <button type="submit" @click="addData">添加数据及依赖</button>
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
