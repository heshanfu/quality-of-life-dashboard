<template lang="html">
    <div v-if="sharedState.selected.length > 0 && sharedState.metric.data" id="datatable">
        <div class="tablescroll">
            <table class="mdl-data-table mdl-js-data-table datatable">
                <thead>
                    <tr>
                        <th class="mdl-data-table__cell--non-numeric">
                            <span class="tooltip" v-bind:title="privateState.neighborhoodDefinition">{{ privateState.neighborhoodDescriptor }}</span>
                        </th>
                        <th>{{sharedState.year}} Value</th>
                        <th v-if="sharedState.metric.data.a">Accuracy</th>
                        <th v-if="sharedState.metric.years.length > 1">Trend<br>{{sharedState.metric.years[0]}}-{{sharedState.metric.years[sharedState.metric.years.length - 1]}}</th>
                        <th v-if="sharedState.metric.config.raw_label">Number</th>
                        <th v-if="sharedState.metric.years.length > 1 && sharedState.metric.config.raw_label">Trend<br>{{sharedState.metric.years[0]}}-{{sharedState.metric.years[sharedState.metric.years.length - 1]}}</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="n in sharedState.selected" v-on:mouseover="highlight([n])" v-on:mouseout="highlight([])">
                        <td class="mdl-data-table__cell--non-numeric">{{n}}</td>
                        <td>{{ formatVal(getVal(n)) }}</td>
                        <td v-if="sharedState.metric.config.accuracy"> &#177; {{ formatVal(getAccuracy(n)) }}</td>
                        <td v-if="sharedState.metric.years.length > 1" v-html="trend(n)"></td>
                        <td v-if="sharedState.metric.config.raw_label &&  sharedState.metric.data.w">{{ formatRaw(getRaw(n)) }}<span class="units" v-if="sharedState.metric.config.raw_label" v-html="' ' + sharedState.metric.config.raw_label"></span></td>
                        <td v-if="sharedState.metric.years.length > 1 && sharedState.metric.config.raw_label" v-html="trendRaw(n)"></td>
                    </tr>
                </tbody>
            </table>
        </div>
        <p class="mdl-typography--text-right">
            <a download="data.csv" class="mdl-button mdl-js-button mdl-js-ripple-effect download" v-on:click="downloadTable('#datatable table')">
                Download
            </a>
        </p>
    </div>
</template>

<script>
import table2csv from '../modules/table2csv';
import {prettyNumber, round} from '../modules/number_format';
import isNumeric from '../modules/isnumeric';

export default {
    name: 'sc-datatable',
    methods: {
      highlight: function(n){
        this.sharedState.highlight = n;
      },
        downloadTable: function(theTable) {
            let csvData = table2csv(theTable);
            // i hate you ie
            if (window.navigator.msSaveBlob) {
                let blob = new Blob([csvData],{ type: "application/csv;charset=utf-8;"});
                navigator.msSaveBlob(blob, 'data.csv');
            } else {
                document.querySelector('#datatable .download').href = 'data:text/csv;charset=utf-8;base64,' + btoa(csvData);
            }
        },
        trendIcon: function(num) {
            if (num === 0) {
                return '<svg class="icon"><use xlink:href="#icon-trending_flat"></use></svg>';
            } else if (num > 0) {
                return '<svg class="icon"><use xlink:href="#icon-trending_up"></use></svg>';
            } else {
                return '<svg class="icon"><use xlink:href="#icon-trending_down"></use></svg>';
            }
        },
        trend: function (n) {
            let sharedState = this.sharedState;
            let begin = sharedState.metric.data.map[n][`y_${sharedState.metric.years[sharedState.metric.years.length - 1]}`];
            let end = sharedState.metric.data.map[n][`y_${sharedState.metric.years[0]}`];

            if (isNumeric(begin) && isNumeric(end)) {
                let trendVal = round(Number(begin), sharedState.metric.config.decimals) - round(Number(end), sharedState.metric.config.decimals);
                return `${this.trendIcon(trendVal)} ${prettyNumber(trendVal, this.sharedState.metric.config.decimals, this.sharedState.metric.config.prefix, this.sharedState.metric.config.suffix)}`;
            } else {
                return '--';
            }
        },
        trendRaw(n) {
            let sharedState = this.sharedState;
            let begin = sharedState.metric.data.map[n][`y_${sharedState.metric.years[sharedState.metric.years.length - 1]}`] * sharedState.metric.data.w[n][`y_${sharedState.metric.years[sharedState.metric.years.length - 1]}`];
            let end = sharedState.metric.data.map[n][`y_${sharedState.metric.years[0]}`] * sharedState.metric.data.w[n][`y_${sharedState.metric.years[0]}`];

            if (isNumeric(begin) && isNumeric(end)) {
                let trendVal = begin - end;
                return  `${this.trendIcon(trendVal)} ${prettyNumber(trendVal, 0)}`;
            } else {
                return  '--';
            }
        },
        getVal: function(n) {
            let sharedState = this.sharedState;
            return sharedState.metric.data.map[n][`y_${sharedState.year}`];
        },
        getAccuracy: function(n) {
            let sharedState = this.sharedState;
            return sharedState.metric.data.a[n][`y_${sharedState.year}`];
        },
        getRaw: function(n) {
            let sharedState = this.sharedState;
            return sharedState.metric.data.w[n][`y_${sharedState.year}`] * sharedState.metric.data.map[n][`y_${sharedState.year}`];
        },
        formatVal: function(num) {
            let sharedState = this.sharedState;
            return prettyNumber(num, sharedState.metric.config.decimals, sharedState.metric.config.prefix, sharedState.metric.config.suffix);
        },
        formatRaw: function(num) {
            return prettyNumber(num, 0);
        }
    }
};
</script>

<style lang="css">
#datatable {
    margin: 10px 15px;
}
#datatable table {
    width: 100%;
}
#datatable tbody tr, #datatable tbody td {
    height: 28px;
    padding: 5px 18px 5px 24px;
}
#datatable .tablescroll {
    max-height: 350px;
    overflow: auto;
}
#datatable p {
    margin-top: 10px;
}
#datatable .tooltip {
    border-bottom: 1px dashed rgba(0,0,0,.54);
}
#datatable .icon {
    vertical-align: middle;
    width: 24px;
    height: 24px;
}
</style>
